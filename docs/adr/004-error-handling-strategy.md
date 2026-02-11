# ADR-004: Error Handling Strategy

**Status**: Draft (Pending Review)  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer, Stakeholder  
**Supersedes**: N/A

---

## Context

We need a comprehensive error handling strategy that covers:
- Business rule violations (validation errors, domain logic failures)
- Technical failures (database errors, network issues, system exceptions)
- API error responses (HTTP status codes, error format)
- Frontend error handling and user feedback
- Logging and monitoring of errors

Without a clear strategy, agents may handle errors inconsistently, leading to:
- Poor user experience (unclear error messages)
- Difficult debugging (errors not properly logged)
- Security vulnerabilities (exposing internal details)
- Business rule violations treated as exceptions

---

## Decision

We will use a **multi-layered error handling approach** with:
1. **Result<T> pattern** for business failures (commands/domain logic)
2. **Exceptions** for technical failures only
3. **RFC 7807 Problem Details** for HTTP error responses
4. **Global exception handling** middleware in ASP.NET Core
5. **Structured error logging** with Serilog
6. **User-friendly error display** in Angular with PrimeNG Toast

### Layer 1: Result<T> Pattern (Business Logic)

**Never use exceptions for business rule violations**. Use Result<T> to represent success or failure.

**Result<T> Implementation**:
```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public bool IsFailure => !IsSuccess;
    public T? Value { get; }
    public string Error { get; }
    public Dictionary<string, string[]>? ValidationErrors { get; }
    
    private Result(bool isSuccess, T? value, string error, Dictionary<string, string[]>? validationErrors = null)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
        ValidationErrors = validationErrors;
    }
    
    public static Result<T> Success(T value) => new(true, value, string.Empty);
    
    public static Result<T> Failure(string error) => new(false, default, error);
    
    public static Result<T> ValidationFailure(Dictionary<string, string[]> errors) =>
        new(false, default, "Validation failed", errors);
}

// Non-generic version for commands that don't return data
public class Result
{
    public bool IsSuccess { get; }
    public bool IsFailure => !IsSuccess;
    public string Error { get; }
    
    private Result(bool isSuccess, string error)
    {
        IsSuccess = isSuccess;
        Error = error;
    }
    
    public static Result Success() => new(true, string.Empty);
    public static Result Failure(string error) => new(false, error);
}
```

**Usage in Command Handlers**:
```csharp
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Result<ProductDto>>
{
    private readonly AppDbContext _context;
    
    public async Task<Result<ProductDto>> Handle(
        CreateProductCommand request, 
        CancellationToken cancellationToken)
    {
        // Business rule validation
        if (await _context.Products.AnyAsync(p => p.Sku == request.Sku, cancellationToken))
        {
            return Result<ProductDto>.Failure("A product with this SKU already exists");
        }
        
        if (request.Price < 0)
        {
            return Result<ProductDto>.Failure("Price cannot be negative");
        }
        
        // Success path
        var product = new Product
        {
            Name = request.Name,
            Sku = request.Sku,
            Price = request.Price
        };
        
        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);
        
        var dto = MapToDto(product);
        return Result<ProductDto>.Success(dto);
    }
}
```

**FluentValidation Integration**:
```csharp
public class CreateProductCommandValidator : AbstractValidator<CreateProductCommand>
{
    public CreateProductCommandValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required")
            .MaximumLength(200).WithMessage("Product name cannot exceed 200 characters");
            
        RuleFor(x => x.Sku)
            .NotEmpty().WithMessage("SKU is required")
            .Matches("^[A-Z0-9-]+$").WithMessage("SKU must contain only uppercase letters, numbers, and hyphens");
            
        RuleFor(x => x.Price)
            .GreaterThanOrEqualTo(0).WithMessage("Price cannot be negative");
    }
}
```

**MediatR Pipeline Behavior for Validation**:
```csharp
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;
    
    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }
    
    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        if (!_validators.Any())
        {
            return await next();
        }
        
        var context = new ValidationContext<TRequest>(request);
        var validationResults = await Task.WhenAll(
            _validators.Select(v => v.ValidateAsync(context, cancellationToken)));
        
        var failures = validationResults
            .SelectMany(r => r.Errors)
            .Where(f => f != null)
            .ToList();
        
        if (failures.Any())
        {
            // Return Result<T> with validation errors
            var errors = failures
                .GroupBy(x => x.PropertyName)
                .ToDictionary(
                    g => g.Key,
                    g => g.Select(x => x.ErrorMessage).ToArray()
                );
            
            // This assumes TResponse is Result<T>
            var resultType = typeof(TResponse);
            if (resultType.IsGenericType && resultType.GetGenericTypeDefinition() == typeof(Result<>))
            {
                var method = resultType.GetMethod(nameof(Result<object>.ValidationFailure));
                return (TResponse)method!.Invoke(null, new object[] { errors })!;
            }
            
            throw new ValidationException(failures); // Fallback
        }
        
        return await next();
    }
}
```

### Layer 2: Exceptions (Technical Failures Only)

**Use exceptions for**:
- Database connection failures
- Network errors
- File system errors
- Null reference errors (shouldn't happen with nullable reference types)
- Configuration errors
- Third-party API failures

**Don't use exceptions for**:
- Business rule violations
- Validation failures
- Expected error conditions

**Custom Exception Types**:
```csharp
// Base exception for domain
public abstract class DomainException : Exception
{
    protected DomainException(string message) : base(message) { }
}

// Entity not found (404)
public class NotFoundException : DomainException
{
    public NotFoundException(string entityName, object key)
        : base($"{entityName} with key '{key}' was not found")
    {
    }
}

// Concurrency conflict (409)
public class ConcurrencyException : DomainException
{
    public ConcurrencyException(string message) : base(message) { }
}

// Unauthorized access (403)
public class ForbiddenException : DomainException
{
    public ForbiddenException(string message) : base(message) { }
}
```

### Layer 3: RFC 7807 Problem Details (HTTP Responses)

**Implement global exception handler**:
```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;
    
    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }
    
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "An unhandled exception occurred");
        
        var problemDetails = CreateProblemDetails(exception, httpContext);
        
        httpContext.Response.StatusCode = problemDetails.Status ?? 500;
        httpContext.Response.ContentType = "application/problem+json";
        
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);
        
        return true;
    }
    
    private static ProblemDetails CreateProblemDetails(Exception exception, HttpContext context)
    {
        return exception switch
        {
            NotFoundException notFound => new ProblemDetails
            {
                Status = StatusCodes.Status404NotFound,
                Title = "Resource Not Found",
                Detail = notFound.Message,
                Instance = context.Request.Path,
                Type = "https://api.example.com/errors/not-found"
            },
            
            ConcurrencyException concurrency => new ProblemDetails
            {
                Status = StatusCodes.Status409Conflict,
                Title = "Concurrency Conflict",
                Detail = concurrency.Message,
                Instance = context.Request.Path,
                Type = "https://api.example.com/errors/conflict"
            },
            
            ForbiddenException forbidden => new ProblemDetails
            {
                Status = StatusCodes.Status403Forbidden,
                Title = "Forbidden",
                Detail = forbidden.Message,
                Instance = context.Request.Path,
                Type = "https://api.example.com/errors/forbidden"
            },
            
            ValidationException validation => new ProblemDetails
            {
                Status = StatusCodes.Status400BadRequest,
                Title = "Validation Failed",
                Detail = "One or more validation errors occurred",
                Instance = context.Request.Path,
                Type = "https://api.example.com/errors/validation",
                Extensions =
                {
                    ["errors"] = validation.Errors.GroupBy(e => e.PropertyName)
                        .ToDictionary(g => g.Key, g => g.Select(e => e.ErrorMessage).ToArray())
                }
            },
            
            _ => new ProblemDetails
            {
                Status = StatusCodes.Status500InternalServerError,
                Title = "Internal Server Error",
                Detail = "An unexpected error occurred",
                Instance = context.Request.Path,
                Type = "https://api.example.com/errors/internal"
            }
        };
    }
}

// Register in Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
```

**RFC 7807 Response Format**:
```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "One or more validation errors occurred",
  "instance": "/api/products",
  "errors": {
    "Name": ["Product name is required"],
    "Price": ["Price cannot be negative"]
  }
}
```

### Layer 4: Controller Error Handling

**Handle Result<T> in controllers**:
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public ProductsController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> CreateProduct(
        CreateProductCommand command,
        CancellationToken cancellationToken)
    {
        var result = await _mediator.Send(command, cancellationToken);
        
        if (result.IsFailure)
        {
            // Return Problem Details for business failures
            if (result.ValidationErrors != null)
            {
                return ValidationProblem(result.ValidationErrors, detail: result.Error);
            }
            
            return Problem(
                statusCode: StatusCodes.Status400BadRequest,
                title: "Business Rule Violation",
                detail: result.Error
            );
        }
        
        return CreatedAtAction(
            nameof(GetProduct),
            new { id = result.Value!.Id },
            result.Value
        );
    }
    
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var query = new GetProductByIdQuery { Id = id };
        var result = await _mediator.Send(query);
        
        if (result == null)
        {
            throw new NotFoundException(nameof(Product), id); // Global handler catches
        }
        
        return Ok(result);
    }
}
```

### Layer 5: Frontend Error Handling

**Centralized Angular Error Service**:
```typescript
import { Injectable, inject } from '@angular/core';
import { MessageService } from 'primeng/api';
import { HttpErrorResponse } from '@angular/common/http';

export interface ProblemDetails {
  type: string;
  title: string;
  status: number;
  detail: string;
  instance: string;
  errors?: Record<string, string[]>;
}

@Injectable({ providedIn: 'root' })
export class ErrorHandlerService {
  private messageService = inject(MessageService);
  
  handleError(error: unknown): void {
    if (error instanceof HttpErrorResponse) {
      this.handleHttpError(error);
    } else if (error instanceof Error) {
      this.handleClientError(error);
    } else {
      this.handleUnknownError(error);
    }
  }
  
  private handleHttpError(error: HttpErrorResponse): void {
    const problemDetails = error.error as ProblemDetails;
    
    if (problemDetails && problemDetails.title) {
      // RFC 7807 Problem Details
      if (problemDetails.errors) {
        // Validation errors
        const errorMessages = Object.entries(problemDetails.errors)
          .flatMap(([, messages]) => messages)
          .join('; ');
        
        this.messageService.add({
          severity: 'error',
          summary: problemDetails.title,
          detail: errorMessages,
          life: 5000
        });
      } else {
        this.messageService.add({
          severity: 'error',
          summary: problemDetails.title,
          detail: problemDetails.detail,
          life: 5000
        });
      }
    } else {
      // Fallback for non-Problem Details responses
      this.messageService.add({
        severity: 'error',
        summary: `Error ${error.status}`,
        detail: error.message || 'An unexpected error occurred',
        life: 5000
      });
    }
    
    // Log to console in development
    if (!environment.production) {
      console.error('HTTP Error:', error);
    }
  }
  
  private handleClientError(error: Error): void {
    this.messageService.add({
      severity: 'error',
      summary: 'Application Error',
      detail: error.message,
      life: 5000
    });
    
    console.error('Client Error:', error);
  }
  
  private handleUnknownError(error: unknown): void {
    this.messageService.add({
      severity: 'error',
      summary: 'Unknown Error',
      detail: 'An unexpected error occurred',
      life: 5000
    });
    
    console.error('Unknown Error:', error);
  }
}
```

**HTTP Interceptor for Error Handling**:
```typescript
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';
import { ErrorHandlerService } from './error-handler.service';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const errorHandler = inject(ErrorHandlerService);
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      errorHandler.handleError(error);
      return throwError(() => error);
    })
  );
};
```

**Usage in Components**:
```typescript
@Component({
  selector: 'app-product-form',
  standalone: true,
  // ...
})
export class ProductFormComponent {
  private productService = inject(ProductService);
  private errorHandler = inject(ErrorHandlerService);
  
  async onSubmit(): Promise<void> {
    try {
      const result = await this.productService.create(this.form.value);
      this.router.navigate(['/products', result.id]);
    } catch (error) {
      // Interceptor already handled user notification
      // Additional component-specific error handling here if needed
      console.error('Failed to create product:', error);
    }
  }
}
```

### Layer 6: Logging

**Log errors with context**:
```csharp
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Result<ProductDto>>
{
    private readonly ILogger<CreateProductCommandHandler> _logger;
    
    public async Task<Result<ProductDto>> Handle(CreateProductCommand request, CancellationToken ct)
    {
        _logger.LogInformation("Creating product with SKU: {Sku}", request.Sku);
        
        try
        {
            // Business logic
            
            if (skuExists)
            {
                _logger.LogWarning("Product creation failed: SKU {Sku} already exists", request.Sku);
                return Result<ProductDto>.Failure("SKU already exists");
            }
            
            // Success
            _logger.LogInformation("Product created successfully with ID: {ProductId}", product.Id);
            return Result<ProductDto>.Success(dto);
        }
        catch (DbUpdateException ex)
        {
            _logger.LogError(ex, "Database error while creating product with SKU: {Sku}", request.Sku);
            throw; // Let global handler deal with it
        }
    }
}
```

---

## Consequences

### Positive Consequences

- **Clear Separation**: Business failures vs technical failures handled differently
- **Standard Format**: RFC 7807 provides consistent API errors
- **Better UX**: Validation errors clearly communicated to users
- **No Exception Abuse**: Exceptions only for exceptional cases
- **Centralized Handling**: Global handler catches all unhandled exceptions
- **Structured Logging**: All errors logged with context
- **Type Safety**: Result<T> is type-safe and explicit

### Negative Consequences

- **Complexity**: More error handling code to write
- **Learning Curve**: Agents must understand Result<T> pattern
- **Verbosity**: Every command handler must handle Result<T>
- **Testing**: More test cases needed to cover error paths

---

## Alternatives Considered

### Alternative 1: Exceptions for Everything
- **Pros**: Simple, .NET standard pattern
- **Cons**: Performance overhead, control flow via exceptions, unclear business failures
- **Rejected**: Mixing business and technical failures is error-prone

### Alternative 2: Custom Error Format (Not RFC 7807)
- **Pros**: Simpler, less verbose
- **Cons**: Non-standard, harder to integrate with tools, less information
- **Rejected**: RFC 7807 is industry standard

### Alternative 3: Error Codes Instead of Messages
- **Pros**: Easier localization, more structured
- **Cons**: Need error code registry, less human-readable
- **Deferred**: Can add error codes to Problem Details later if needed

---

## Implementation Notes

### Setup Required

1. Implement `Result<T>` class in `Core` project
2. Add `GlobalExceptionHandler` in `API` project
3. Register exception handler in `Program.cs`
4. Add validation pipeline behavior to MediatR
5. Create custom exception types
6. Implement Angular `ErrorHandlerService`
7. Add HTTP error interceptor

### Testing Strategy

- Unit test command handlers for both success and failure paths
- Test that Result<T> is returned correctly  
- Test global exception handler with different exception types
- Integration test API endpoints for error responses
- Test Angular error service displays correct messages

---

## Related Decisions

- [ADR-002: C# Coding Standards](002-csharp-coding-standards.md) - Error handling conventions
- [ADR-005: Logging Strategy](005-logging-strategy.md) - How errors are logged
- [ADR-003: TypeScript Standards](003-typescript-angular-coding-standards.md) - Frontend error handling

---

**Status**: Ready for Review  
**Action Required**: Stakeholder approval to proceed with implementation
