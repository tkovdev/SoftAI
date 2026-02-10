# Example: CQRS Pattern with MediatR

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer  
**Supersedes**: N/A

---

## Context

We need to choose an architectural pattern for handling business logic in the backend. The application will have:
- Complex business operations (product management, order processing, inventory)
- Read-heavy workload with some writes
- Need for validation and authorization on commands
- Multiple clients (web UI, mobile app, API consumers)

Traditional service layer patterns can lead to:
- Fat services with mixed concerns (reads and writes together)
- Difficulty in testing and mocking
- Unclear separation between queries and commands
- Challenges in adding cross-cutting concerns (logging, validation, caching)

---

## Decision

We will use **CQRS (Command Query Responsibility Segregation) pattern with MediatR** for all business logic.

### Key Aspects

**Commands** (Write Operations):
- Represent intent to change state (CreateProductCommand, UpdateOrderCommand)
- Return `Result<T>` to indicate success/failure
- Validated using FluentValidation validators
- Handled by dedicated command handlers implementing `IRequestHandler<TCommand, Result<T>>`

**Queries** (Read Operations):
- Represent requests for data (GetProductByIdQuery, GetOrdersQuery)
- Return DTOs optimized for specific use cases
- Use `.AsNoTracking()` for performance
- Handled by dedicated query handlers implementing `IRequestHandler<TQuery, TResponse>`

**Handlers**:
- Single Responsibility: One handler per command/query
- Testable in isolation with mocked dependencies
- Transaction management at handler level
- Can use repository or DbContext directly

**Controllers**:
- Thin, delegate to MediatR
- Handle HTTP concerns only (routing, status codes)
- Inject `IMediator`, send commands/queries
- Map results to HTTP responses

### Technology

- **MediatR**: In-process messaging for command/query dispatching
- **FluentValidation**: Declarative command/query validation
- **AutoMapper**: DTO mapping (optional, can map manually)

---

## Consequences

### Positive Consequences

- **Clear Separation**: Commands and queries are distinctly separated
- **Single Responsibility**: Each handler does one thing
- **Testability**: Handlers easy to unit test in isolation
- **Cross-Cutting Concerns**: Can add pipeline behaviors for logging, validation, transactions
- **Scalability**: Easy to optimize reads vs writes separately
- **Discoverability**: All operations are explicit command/query classes
- **Thin Controllers**: Controllers become very lightweight

### Negative Consequences

- **More Files**: Each operation needs command/query, handler, validator (3 files minimum)
- **Learning Curve**: Team needs to understand CQRS and MediatR concepts
- **Indirection**: Call flow goes through MediatR dispatcher instead of direct service calls
- **Overkill for Simple CRUD**: Very simple operations get the same ceremony as complex ones

### Neutral Considerations

- **Event Sourcing**: CQRS enables event sourcing if we need it later, but we're not using it initially
- **Separate Data Stores**: Could use separate read/write databases later, but starting with single database
- **Async**: All handlers are async by default

---

## Alternatives Considered

### Alternative 1: Traditional Service Layer

- **Description**: Business logic in service classes (ProductService, OrderService) with methods for all operations
- **Pros**: 
  - Familiar pattern for most developers
  - Fewer files (one service class instead of many handlers)
  - Simpler for very basic CRUD
- **Cons**: 
  - Services become bloated over time
  - Mixed responsibilities (reads and writes together)
  - Difficult to add cross-cutting concerns
  - Testing requires mocking large service interfaces
- **Why rejected**: Doesn't scale well as application grows, harder to maintain

### Alternative 2: Repository Pattern Only

- **Description**: Controllers call repositories directly, business logic in domain models or controllers
- **Pros**:
  - Very simple, minimal abstraction
  - Direct data access, no indirection
- **Cons**:
  - Business logic scattered (controllers, models, helpers)
  - No good place for complex operations
  - Difficult to add validation, authorization  
  - Poor testability
- **Why rejected**: Too simplistic for non-trivial application

### Alternative 3: Domain-Driven Design (DDD) with Aggregates

- **Description**: Rich domain models with behavior, aggregate roots enforcing invariants
- **Pros**:
  - Natural fit for complex domain logic
  - Well-defined boundaries and rules
  - Strong modeling capabilities
- **Cons**:
  - Significant complexity overhead
  - Requires deep domain understanding
  - Steeper learning curve
  - Can be over-engineering for straightforward CRUD
- **Why rejected**: CQRS provides good structure without full DDD complexity; can add DDD concepts later if needed

---

## Implementation Notes

### Project Structure

```
src/[Project].Application/
├── Products/
│   ├── Commands/
│   │   ├── CreateProduct/
│   │   │   ├── CreateProductCommand.cs
│   │   │   ├── CreateProductCommandValidator.cs
│   │   │   └── CreateProductCommandHandler.cs
│   │   └── UpdateProduct/
│   │       ├── UpdateProductCommand.cs
│   │       ├── UpdateProductCommandValidator.cs
│   │       └── UpdateProductCommandHandler.cs
│   ├── Queries/
│   │   ├── GetProductById/
│   │   │   ├── GetProductByIdQuery.cs
│   │   │   └── GetProductByIdQueryHandler.cs
│   │   └── GetProducts/
│   │       ├── GetProductsQuery.cs
│   │       └── GetProductsQueryHandler.cs
│   └── DTOs/
│       ├── ProductDto.cs
│       └── ProductListDto.cs
```

### Command Example

```csharp
// Command
public record CreateProductCommand : IRequest<Result<int>>
{
    public string Name { get; init; }
    public decimal Price { get; init; }
    public string Description { get; init; }
}

// Validator
public class CreateProductCommandValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(200);
        RuleFor(x => x.Price).GreaterThan(0);
    }
}

// Handler
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Result<int>>
{
    private readonly AppDbContext _context;
    
    public async Task<Result<int>> Handle(CreateProductCommand request, CancellationToken ct)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            Description = request.Description
        };
        
        _context.Products.Add(product);
        await _context.SaveChangesAsync(ct);
        
        return Result<int>.Success(product.Id);
    }
}
```

### Query Example

```csharp
// Query
public record GetProductByIdQuery(int Id) : IRequest<ProductDto>;

// Handler
public class GetProductByIdQueryHandler : IRequestHandler<GetProductByIdQuery, ProductDto>
{
    private readonly AppDbContext _context;
    private readonly IMapper _mapper;
    
    public async Task<ProductDto> Handle(GetProductByIdQuery request, CancellationToken ct)
    {
        var product = await _context.Products
            .AsNoTracking()
            .FirstOrDefaultAsync(p => p.Id == request.Id, ct);
            
        return _mapper.Map<ProductDto>(product);
    }
}
```

### Controller Example

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    [HttpPost]
    public async Task<ActionResult<int>> Create(CreateProductCommand command)
    {
        var result = await _mediator.Send(command);
        return result.IsSuccess ? Ok(result.Value) : BadRequest(result.Error);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
    {
        var product = await _mediator.Send(new GetProductByIdQuery(id));
        return product != null ? Ok(product) : NotFound();
    }
}
```

### Testing

```csharp
public class CreateProductCommandHandlerTests
{
    [Fact]
    public async Task Handle_ValidCommand_CreatesProduct()
    {
        // Arrange
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(databaseName: "TestDb")
            .Options;
        var context = new AppDbContext(options);
        var handler = new CreateProductCommandHandler(context);
        var command = new CreateProductCommand 
        { 
            Name = "Test Product", 
            Price = 99.99m,
            Description = "Description"
        };
        
        // Act
        var result = await handler.Handle(command, CancellationToken.None);
        
        // Assert
        Assert.True(result.IsSuccess);
        Assert.True(result.Value > 0);
        Assert.Equal(1, await context.Products.CountAsync());
    }
}
```

### Registration (Program.cs)

```csharp
builder.Services.AddMediatR(cfg => 
    cfg.RegisterServicesFromAssembly(typeof(CreateProductCommand).Assembly));
builder.Services.AddValidatorsFromAssembly(typeof(CreateProductCommand).Assembly);
```

---

## References

- [MediatR Documentation](https://github.com/jbogard/MediatR)
- [FluentValidation Documentation](https://docs.fluentvalidation.net/)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)
- Related: ADR-001 (Clean Architecture) - CQRS fits naturally in Application layer

---

## Notes

- Start simple: Not every command needs complex validation or business logic
- Don't over-separate: If command and query are identical in structure, evaluate if separate handlers are needed
- Consider adding MediatR pipeline behaviors for:
  - Automatic validation (call FluentValidation before handler)
  - Logging (log all commands/queries)
  - Transaction management (wrap handlers in transaction)
  - Performance monitoring

---

**Future Considerations:**
- If read workload becomes dominant, consider read-optimized data store (materialized views, read replicas)
- If event sourcing is needed, commands can also publish events
- Can introduce domain events through handlers for complex workflows
