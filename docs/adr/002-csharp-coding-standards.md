# ADR-002: C# Coding Standards

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer, Stakeholder  
**Supersedes**: N/A

---

## Context

We need to establish consistent C# coding standards for the .NET 10 backend to ensure:
- Code consistency across all agents and developers
- Readability and maintainability
- Automated enforcement through tooling (StyleCop, analyzers)
- Alignment with Microsoft's official C# conventions
- Fast onboarding for new agents and developers

Without clear standards, agents may produce inconsistent code styles, making the codebase harder to maintain and review.

---

## Decision

We will adopt **Microsoft's C# Coding Conventions** with specific clarifications and tooling configurations.

### Naming Conventions

**Classes, Interfaces, Methods, Properties, Events** (Public members):
```csharp
// PascalCase
public class ProductService { }
public interface IProductRepository { }
public string ProductName { get; set; }
public void CalculateTotal() { }
public event EventHandler OrderPlaced;
```

**Local Variables, Method Parameters**:
```csharp
// camelCase
public void ProcessOrder(int orderId, string customerName)
{
    var orderTotal = 0m;
    var shippingAddress = GetAddress(customerId);
}
```

**Private Fields**:
```csharp
// _camelCase with underscore prefix
private readonly IProductRepository _productRepository;
private int _retryCount;
```

**Constants**:
```csharp
// PascalCase
public const int MaxRetryAttempts = 3;
private const string DefaultCategory = "General";
```

**Interface Prefix**:
- All interfaces MUST start with `I` (e.g., `IProductService`, `IRepository<T>`)

### File Organization

**One Type Per File**:
- One class, interface, or enum per file
- File name matches the type name exactly
- Exception: Nested types can be in same file if tightly coupled

**Namespace Structure**:
```csharp
// Match folder structure
// File: src/ProjectName.Application/Products/Commands/CreateProductCommand.cs

namespace ProjectName.Application.Products.Commands;

public class CreateProductCommand : IRequest<Result<ProductDto>>
{
    // Class members...
}
```

**File-scoped Namespaces** (.NET 10):
```csharp
// ✅ Use file-scoped namespaces (reduces indentation)
namespace ProjectName.Application.Products.Commands;

public class CreateProductCommand
{
    // Members at namespace level
}

// ❌ Don't use traditional block namespaces
namespace ProjectName.Application.Products.Commands
{
    public class CreateProductCommand
    {
        // Extra indentation level
    }
}
```

**Using Directives**:
```csharp
// Order: System namespaces first, then others alphabetically
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

using MediatR;
using Microsoft.EntityFrameworkCore;

using ProjectName.Core.Entities;
using ProjectName.Core.Interfaces;
```

### Type Usage

**Implicit vs Explicit Typing**:
```csharp
// ✅ Use var when type is obvious from right side
var customer = new Customer();
var products = GetProducts();
var count = products.Count();

// ✅ Use explicit type when not obvious
IProductRepository repository = GetRepository();
int timeout = configuration.GetTimeout();

// ❌ Don't use var when it reduces clarity
var result = ProcessData(); // What type is result?
```

### Async/Await Patterns

**Async Method Naming**:
```csharp
// ✅ Suffix with Async
public async Task<Product> GetProductAsync(int id)
public async Task SaveChangesAsync()
public async Task<Result<Order>> CreateOrderAsync(CreateOrderCommand command)

// ❌ Don't omit Async suffix
public async Task<Product> GetProduct(int id) // Missing Async
```

**Always Use Async/Await for I/O**:
```csharp
// ✅ Async all the way
public async Task<Product> GetProductAsync(int productId)
{
    return await _context.Products
        .Where(p => p.Id == productId)
        .FirstOrDefaultAsync();
}

// ❌ Don't block on async code
public Product GetProduct(int productId)
{
    return _context.Products
        .Where(p => p.Id == productId)
        .FirstOrDefaultAsync().Result; // Deadlock risk!
}
```

**ConfigureAwait**:
```csharp
// In library code, use ConfigureAwait(false) for better performance
await SaveToDatabaseAsync().ConfigureAwait(false);

// In ASP.NET Core controllers/handlers, ConfigureAwait(false) is optional
// (but harmless, context is not captured by default)
```

### Error Handling

**Use Specific Exceptions**:
```csharp
// ✅ Throw specific exceptions
throw new ArgumentNullException(nameof(productId));
throw new InvalidOperationException("Product already exists");

// ❌ Don't throw generic Exception
throw new Exception("Something went wrong");
```

**Result Pattern for Business Failures** (not exceptions):
```csharp
// ✅ Commands return Result<T> for business failures
public async Task<Result<ProductDto>> Handle(
    CreateProductCommand request, 
    CancellationToken cancellationToken)
{
    if (await _context.Products.AnyAsync(p => p.Sku == request.Sku))
    {
        return Result<ProductDto>.Failure("Product with this SKU already exists");
    }
    
    // Success path
    var product = new Product { /* ... */ };
    return Result<ProductDto>.Success(productDto);
}

// ❌ Don't throw exceptions for business rule violations
if (skuExists)
{
    throw new InvalidOperationException("SKU exists"); // Wrong!
}
```

### Code Documentation

**XML Documentation for Public APIs**:
```csharp
/// <summary>
/// Retrieves a product by its unique identifier.
/// </summary>
/// <param name="productId">The unique identifier of the product.</param>
/// <param name="cancellationToken">Cancellation token.</param>
/// <returns>The product if found; otherwise, null.</returns>
/// <exception cref="ArgumentException">Thrown when productId is less than 1.</exception>
public async Task<Product?> GetProductByIdAsync(
    int productId, 
    CancellationToken cancellationToken = default)
{
    if (productId < 1)
        throw new ArgumentException("Product ID must be positive", nameof(productId));
        
    return await _context.Products.FindAsync(productId, cancellationToken);
}
```

**Internal Implementation Comments** (when needed):
```csharp
// ✅ Explain WHY, not WHAT
// Use cached value to avoid database roundtrip for unchanged data
if (_cachedProduct?.Version == currentVersion)
    return _cachedProduct;

// ❌ Don't state the obvious
// Get the product
var product = await GetProductAsync(id);
```

### StyleCop Configuration

Enable these StyleCop rules via `.editorconfig`:

```ini
[*.cs]

# SA1200: Using directives should be placed correctly (inside namespace = false)
dotnet_diagnostic.SA1200.severity = none

# SA1633: File should have header (we don't require copyright headers)
dotnet_diagnostic.SA1633.severity = none

# SA1101: Prefix local calls with this (we don't require 'this.')
dotnet_diagnostic.SA1101.severity = none

# SA1309: Field names should not begin with underscore (we WANT underscore)
dotnet_diagnostic.SA1309.severity = none

# SA1413: Use trailing comma in multi-line initializers
dotnet_diagnostic.SA1413.severity = warning

# CA1062: Validate public method arguments (handled by nullable reference types)
dotnet_diagnostic.CA1062.severity = suggestion

# Enforce nullable reference types
nullable = enable
```

### Code Formatting

**Indentation**: 4 spaces (no tabs)

**Braces**:
```csharp
// ✅ Braces on new line (Allman style)
if (condition)
{
    DoSomething();
}

public class Product
{
    public int Id { get; set; }
}

// ❌ Don't use K&R style
if (condition) {
    DoSomething();
}
```

**Line Length**: Aim for 120 characters max, break at 140

**Method Length**: Aim for < 50 lines, extract methods if longer

---

## Consequences

### Positive Consequences

- **Consistency**: All agents produce uniform code style
- **Automated Enforcement**: StyleCop catches violations at compile time
- **Readability**: Code is easy to read and understand
- **Microsoft Alignment**: Follows official C# conventions
- **Tooling Support**: Visual Studio, Rider, VS Code all support these standards
- **Fast Reviews**: Code reviews focus on logic, not style

### Negative Consequences

- **Initial Setup**: Requires `.editorconfig` configuration in all projects
- **Learning Curve**: Agents must learn Result<T> pattern and async conventions
- **Strict Enforcement**: StyleCop warnings can be noisy initially
- **XML Comments**: Requires discipline to document public APIs

### Neutral Considerations

- **File-scoped Namespaces**: .NET 10 feature, requires understanding
- **Nullable Reference Types**: Requires `nullable = enable`, adds ? annotations

---

## Alternatives Considered

### Alternative 1: No Formal Standards
- **Pros**: Faster initial setup, less enforcement overhead
- **Cons**: Inconsistent code, harder to maintain, code review conflicts
- **Rejected**: Consistency is critical for AI agent teams

### Alternative 2: Stricter Standards (All StyleCop Rules)
- **Pros**: Maximum consistency, catches more potential issues
- **Cons**: Too restrictive, slows development, many false positives
- **Rejected**: Balance needed between strictness and productivity

### Alternative 3: Use Only .editorconfig (No StyleCop)
- **Pros**: Less tooling, simpler setup
- **Cons**: Weaker enforcement, requires manual code review for style
- **Rejected**: StyleCop provides valuable automated enforcement

---

## Implementation Notes

### Setup Required

1. **Create `.editorconfig`** at solution root:
```ini
root = true

[*.cs]
# Indentation
indent_style = space
indent_size = 4

# Naming conventions
dotnet_naming_rule.private_fields_rule.severity = warning
dotnet_naming_rule.private_fields_rule.symbols = private_fields
dotnet_naming_rule.private_fields_rule.style = underscore_camel_case

dotnet_naming_symbols.private_fields.applicable_kinds = field
dotnet_naming_symbols.private_fields.applicable_accessibilities = private

dotnet_naming_style.underscore_camel_case.capitalization = camel_case
dotnet_naming_style.underscore_camel_case.required_prefix = _

# Nullability
nullable = enable

# StyleCop overrides (see above)
```

2. **Add StyleCop.Analyzers** NuGet package to all projects

3. **Configure CI/CD** to fail builds on StyleCop errors

4. **Create Code Snippet Templates** for common patterns:
   - Command handler template
   - Query handler template
   - Entity template
   - Controller template

### Reference Examples

See `docs/examples/backend/` for reference implementations following these standards.

### Training

- All agents will reference this ADR before writing C# code
- Lead Engineer reviews all code for standard compliance
- First PR includes standards enforcement setup

---

## Related Decisions

- [ADR-001: CQRS with MediatR](001-backend-cqrs-with-mediatr.md) - Command/Query patterns referenced here
- [ADR-004: Error Handling Strategy](004-error-handling-strategy.md) - Result<T> pattern details
- [ADR-005: Logging Strategy](005-logging-strategy.md) - Logging standards complement coding standards

---

**Status**: Ready for Review  
**Action Required**: Stakeholder approval to proceed with implementation
