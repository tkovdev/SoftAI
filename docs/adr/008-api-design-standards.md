# ADR-008: API Design Standards

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer  
**Supersedes**: N/A

---

## Context

We need consistent, well-designed RESTful APIs that are:
- Easy to understand and use
- Follow industry best practices
- Properly versioned for backward compatibility
- Well-documented with OpenAPI/Swagger
- Handle errors consistently
- Support pagination, filtering, and sorting for collections
- Provide clear HTTP status codes

Without standards, APIs become inconsistent, confusing, and difficult to maintain. Frontend developers need predictable API behavior.

---

## Decision

We will implement **RESTful API design standards** with clear conventions for resources, HTTP methods, status codes, versioning, pagination, and error handling.

### URL Structure

**Base URL Pattern**:
```
https://api.example.com/api/v1/{resource}
```

**Resource Naming**:
- Use **plural nouns** for collections: `/api/v1/products`, `/api/v1/orders`
- Use lowercase with hyphens for multi-word resources: `/api/v1/product-categories`
- Use **hierarchical structure** for related resources: `/api/v1/orders/{orderId}/items`

**Examples**:
```
GET    /api/v1/products              - List all products
GET    /api/v1/products/{id}         - Get specific product
POST   /api/v1/products              - Create new product
PUT    /api/v1/products/{id}         - Update entire product
PATCH  /api/v1/products/{id}         - Partial update product
DELETE /api/v1/products/{id}         - Delete product

GET    /api/v1/orders/{orderId}/items         - List order items
GET    /api/v1/orders/{orderId}/items/{itemId} - Get specific item
```

### HTTP Methods & Semantics

| Method | Purpose | Request Body | Response Body | Idempotent |
|--------|---------|--------------|---------------|------------|
| **GET** | Retrieve resource(s) | No | Yes | Yes |
| **POST** | Create new resource | Yes | Yes (created resource) | No |
| **PUT** | Replace entire resource | Yes | Yes (updated resource) | Yes |
| **PATCH** | Partial update | Yes | Yes (updated resource) | No |
| **DELETE** | Remove resource | No | No (204) or Yes (deleted resource) | Yes |

### HTTP Status Codes

**Success Codes**:
- `200 OK` - Successful GET, PUT, PATCH, or DELETE with response body
- `201 Created` - Successful POST, include `Location` header with new resource URL
- `204 No Content` - Successful DELETE or update with no response body

**Client Error Codes**:
- `400 Bad Request` - Invalid request format, validation errors
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Resource conflict (e.g., duplicate email)
- `422 Unprocessable Entity` - Validation errors (alternative to 400)

**Server Error Codes**:
- `500 Internal Server Error` - Unexpected server error
- `503 Service Unavailable` - Temporary unavailability

### Error Response Format

**Standard Error Structure**:
```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/v1/products",
  "traceId": "00-1234567890abcdef-1234567890abcdef-00",
  "errors": {
    "Name": ["The Name field is required.", "Name must be between 3 and 100 characters."],
    "Price": ["Price must be greater than 0."]
  }
}
```

This follows **RFC 7807 Problem Details** specification with ASP.NET Core extensions.

**Implementation**:
```csharp
// Use ASP.NET Core ProblemDetails
builder.Services.AddProblemDetails();

// Custom exception handler (in middleware)
app.UseExceptionHandler(exceptionHandlerApp =>
{
    exceptionHandlerApp.Run(async context =>
    {
        var problemDetailsService = context.RequestServices
            .GetRequiredService<IProblemDetailsService>();
        
        // Create ProblemDetails based on exception
        await problemDetailsService.WriteAsync(new ProblemDetailsContext
        {
            HttpContext = context,
            ProblemDetails = { ... }
        });
    });
});
```

### Versioning Strategy

**URL Path Versioning** (Primary approach):
```
/api/v1/products
/api/v2/products
```

**Versioning Rules**:
- Start at `v1`
- Increment major version for breaking changes
- Support previous version for at least 6 months after new version release
- Document breaking changes in release notes
- Use API versioning middleware in ASP.NET Core

**Implementation**:
```csharp
builder.Services.AddApiVersioning(options =>
{
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
    options.ApiVersionReader = new UrlSegmentApiVersionReader();
});

// Controller attribute
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
public class ProductsController : ControllerBase { }
```

### Pagination

**Query Parameters**:
```
GET /api/v1/products?page=2&pageSize=20
```

**Response Format**:
```json
{
  "items": [ /* array of products */ ],
  "pagination": {
    "currentPage": 2,
    "pageSize": 20,
    "totalPages": 10,
    "totalCount": 200,
    "hasPreviousPage": true,
    "hasNextPage": true
  }
}
```

**Default Values**:
- `page`: 1 (first page)
- `pageSize`: 20 (configurable, max 100)

**Implementation**:
```csharp
public class PaginatedList<T>
{
    public List<T> Items { get; set; }
    public PaginationMetadata Pagination { get; set; }
}

public class PaginationMetadata
{
    public int CurrentPage { get; set; }
    public int PageSize { get; set; }
    public int TotalPages { get; set; }
    public int TotalCount { get; set; }
    public bool HasPreviousPage => CurrentPage > 1;
    public bool HasNextPage => CurrentPage < TotalPages;
}
```

### Filtering & Sorting

**Filtering** (Query parameters):
```
GET /api/v1/products?category=electronics&minPrice=100&maxPrice=500
GET /api/v1/products?search=laptop
```

**Sorting**:
```
GET /api/v1/products?sortBy=price&sortOrder=desc
GET /api/v1/products?sortBy=name,price&sortOrder=asc,desc
```

**Implementation in CQRS Query**:
```csharp
public record GetProductsQuery(
    int Page = 1,
    int PageSize = 20,
    string? Category = null,
    decimal? MinPrice = null,
    decimal? MaxPrice = null,
    string? Search = null,
    string? SortBy = "name",
    string? SortOrder = "asc"
) : IRequest<PaginatedList<ProductDto>>;
```

### Request/Response DTOs

**Request DTOs** (for POST/PUT/PATCH):
```csharp
public record CreateProductRequest(
    string Name,
    string Description,
    decimal Price,
    int CategoryId
);
```

**Response DTOs**:
```csharp
public record ProductResponse(
    int Id,
    string Name,
    string Description,
    decimal Price,
    CategoryResponse Category,
    DateTime CreatedAt,
    DateTime UpdatedAt
);
```

**Naming Convention**:
- Request DTOs: `Create{Resource}Request`, `Update{Resource}Request`
- Response DTOs: `{Resource}Response` or `{Resource}Dto`
- Use `record` types for immutability

### Content Negotiation

**Default**: JSON (`application/json`)

**Accept Header Support**:
```
Accept: application/json  (default)
Accept: application/xml   (optional, if needed)
```

**Implementation**:
```csharp
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
    });
```

### OpenAPI/Swagger Documentation

**Swagger Configuration**:
```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "Product management API",
        Contact = new OpenApiContact { Name = "API Support", Email = "support@example.com" }
    });
    
    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
    
    // JWT Authentication
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using the Bearer scheme",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });
});

// Enable in development
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

**Controller XML Documentation**:
```csharp
/// <summary>
/// Gets a product by ID
/// </summary>
/// <param name="id">The product identifier</param>
/// <returns>The product details</returns>
/// <response code="200">Returns the product</response>
/// <response code="404">Product not found</response>
[HttpGet("{id}")]
[ProducesResponseType(typeof(ProductResponse), StatusCodes.Status200OK)]
[ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
public async Task<ActionResult<ProductResponse>> GetProduct(int id)
{
    // Implementation
}
```

### CORS Configuration

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAngularApp", policy =>
    {
        policy.WithOrigins("http://localhost:4200", "https://app.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials(); // For cookies/auth
    });
});

app.UseCors("AllowAngularApp");
```

### Rate Limiting

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
        RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: context.User.Identity?.Name ?? context.Request.Headers.Host.ToString(),
            factory: partition => new FixedWindowRateLimiterOptions
            {
                AutoReplenishment = true,
                PermitLimit = 100,
                QueueLimit = 0,
                Window = TimeSpan.FromMinutes(1)
            }));
});

app.UseRateLimiter();
```

---

## Consequences

### Positive Consequences
- **Consistency**: Predictable API behavior across all endpoints
- **Developer Experience**: Easy to understand and use
- **Documentation**: Swagger provides interactive API documentation
- **Versioning**: Breaking changes don't affect existing clients
- **Standards Compliance**: Follows REST principles and RFC 7807
- **Error Handling**: Clear, structured error responses
- **Scalability**: Pagination prevents large data transfers

### Negative Consequences
- **Initial Setup Time**: Requires configuration and infrastructure setup
- **Verbose Responses**: ProblemDetails can be large for simple errors
- **Versioning Overhead**: Must maintain multiple API versions
- **Learning Curve**: Team must learn and follow conventions

### Neutral Considerations
- Need to document API in OpenAPI spec before implementation
- Frontend developers must handle pagination, filtering, sorting
- Need to educate team on proper HTTP status code usage
- API versioning strategy must be communicated to clients

---

## Alternatives Considered

### Alternative 1: GraphQL
- **Description**: Use GraphQL instead of REST for flexible querying
- **Pros**: Flexible queries, single endpoint, strong typing, no over-fetching
- **Cons**: Steeper learning curve, more complex caching, harder to leverage HTTP semantics, overhead for simple CRUD
- **Why rejected**: REST is simpler for CRUD operations, better tooling support in .NET, team familiarity

### Alternative 2: Header-based API Versioning
- **Description**: Version via `Accept` header (e.g., `Accept: application/vnd.api.v1+json`)
- **Pros**: Clean URLs, follows REST purity
- **Cons**: Less visible, harder to test in browser, more complex routing
- **Why rejected**: URL-based versioning is more explicit and easier to work with

### Alternative 3: gRPC
- **Description**: Use gRPC for high-performance binary protocol
- **Pros**: Very fast, strong typing with Protocol Buffers, bi-directional streaming
- **Cons**: Not browser-friendly, requires code generation, overkill for typical CRUD
- **Why rejected**: REST/JSON is more web-friendly and sufficient for our needs

---

## Implementation Notes

### Controller Structure

**Base Controller** (optional):
```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status500InternalServerError)]
public abstract class BaseApiController : ControllerBase
{
    protected readonly IMediator Mediator;
    
    protected BaseApiController(IMediator mediator)
    {
        Mediator = mediator;
    }
    
    protected ActionResult<T> HandleResult<T>(T result)
    {
        if (result == null)
            return NotFound();
        return Ok(result);
    }
}
```

**Example Controller**:
```csharp
[ApiVersion("1.0")]
public class ProductsController : BaseApiController
{
    public ProductsController(IMediator mediator) : base(mediator) { }
    
    /// <summary>
    /// Retrieves a paginated list of products
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(PaginatedList<ProductResponse>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PaginatedList<ProductResponse>>> GetProducts(
        [FromQuery] GetProductsQuery query)
    {
        var result = await Mediator.Send(query);
        return Ok(result);
    }
    
    /// <summary>
    /// Gets a specific product by ID
    /// </summary>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(ProductResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductResponse>> GetProduct(int id)
    {
        var result = await Mediator.Send(new GetProductQuery(id));
        return HandleResult(result);
    }
    
    /// <summary>
    /// Creates a new product
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(ProductResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductResponse>> CreateProduct(CreateProductCommand command)
    {
        var result = await Mediator.Send(command);
        return CreatedAtAction(nameof(GetProduct), new { id = result.Id }, result);
    }
    
    /// <summary>
    /// Updates an existing product
    /// </summary>
    [HttpPut("{id}")]
    [ProducesResponseType(typeof(ProductResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductResponse>> UpdateProduct(int id, UpdateProductCommand command)
    {
        if (id != command.Id)
            return BadRequest();
            
        var result = await Mediator.Send(command);
        return HandleResult(result);
    }
    
    /// <summary>
    /// Deletes a product
    /// </summary>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        await Mediator.Send(new DeleteProductCommand(id));
        return NoContent();
    }
}
```

### FluentValidation Integration

Validation errors automatically converted to ProblemDetails:
```csharp
builder.Services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());

// Add validation behavior to MediatR pipeline
builder.Services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```

### Testing Requirements

- **Unit Tests**: Test command/query handlers return correct DTOs
- **Integration Tests**: Test API endpoints, status codes, error responses
- **Contract Tests**: Validate OpenAPI spec matches implementation
- **E2E Tests**: Test complete user flows through API

---

## References

- [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)
- [RFC 7807: Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807)
- [ASP.NET Core API Versioning](https://github.com/dotnet/aspnet-api-versioning)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [REST API Best Practices](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [ASP.NET Core ProblemDetails](https://docs.microsoft.com/en-us/aspnet/core/web-api/handle-errors)
