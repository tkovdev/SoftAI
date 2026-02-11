# ADR-005: Logging Strategy

**Status**: Draft (Pending Review)  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer, Stakeholder  
**Supersedes**: N/A

---

## Context

We need a comprehensive logging strategy to:
- Debug issues in development and production
- Monitor application health and performance
- Track user activities and business events
- Comply with audit and security requirements
- Enable troubleshooting with structured, searchable logs
- Integrate with monitoring and alerting systems

Without clear logging guidelines, agents may:
- Log too much (performance impact, noise)
- Log too little (insufficient debugging information)
- Log inconsistently (hard to search and correlate)
- Log sensitive data (security/privacy violations)

---

## Decision

We will use **Serilog** with structured logging, centralized configuration, and clear guidelines on what and how to log.

### Logging Library: Serilog

**Why Serilog:**
- Structured logging (log events as objects, not strings)
- Rich sink ecosystem (Console, File, Seq, Application Insights, etc.)
- Excellent ASP.NET Core integration
- Async logging for performance
- Powerful filtering and enrichment

### Log Levels (When to Use)

**Trace** (Very Verbose - Usually Disabled):
- Detailed method entry/exit
- Loop iterations
- Variable values during execution
- **Use sparingly**: Only for deep debugging

**Debug** (Verbose - Development Only):
- Method entry/exit for key operations
- Key decision points in logic
- Cache hits/misses
- Query execution plans
- **Enable in Development, disable in Production**

**Information** (Normal - Always On):
- Application startup/shutdown
- HTTP requests (handled by middleware)
- Business operations started/completed
- External API calls
- Background job execution
- User authentication events
- Significant state changes

**Warning** (Attention Needed):
- Handled exceptions that don't stop operation
- Deprecated feature usage
- Approaching resource limits
- Retry attempts
- Fallback to default values
- Performance degradation

**Error** (Failure - Needs Investigation):
- Unhandled exceptions
- Failed operations that impact users
- Database errors
- External service failures
- Expected operations that failed unexpectedly

**Critical** (Disaster - Immediate Action):
- Application cannot start
- Database connection lost
- Critical data corruption
- Security breaches
- System-wide failure

### What to Log

**Application Lifecycle**:
```csharp
// Startup
Log.Information("Application starting. Environment: {Environment}, Version: {Version}",
    environment.EnvironmentName, Assembly.GetExecutingAssembly().GetName().Version);

// Shutdown
Log.Information("Application shutting down gracefully");
```

**HTTP Requests** (via ASP.NET Core middleware):
```csharp
// Automatically logged by Serilog.AspNetCore
// GET /api/products returned 200 in 45ms
```

**Business Operations** (Commands and Queries):
```csharp
public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, Result<ProductDto>>
{
    private readonly ILogger<CreateProductCommandHandler> _logger;
    
    public async Task<Result<ProductDto>> Handle(CreateProductCommand request, CancellationToken ct)
    {
        // Log start with context
        _logger.LogInformation(
            "Creating product. SKU: {Sku}, Name: {Name}, Price: {Price}",
            request.Sku, request.Name, request.Price);
        
        try
        {
            // Business logic
            
            if (skuExists)
            {
                // Log business failure
                _logger.LogWarning("Product creation failed. Duplicate SKU: {Sku}", request.Sku);
                return Result<ProductDto>.Failure("SKU already exists");
            }
            
            var product = CreateProduct(request);
            await _context.SaveChangesAsync(ct);
            
            // Log success
            _logger.LogInformation(
                "Product created successfully. ID: {ProductId}, SKU: {Sku}",
                product.Id, product.Sku);
            
            return Result<ProductDto>.Success(MapToDto(product));
        }
        catch (DbUpdateException ex)
        {
            // Log technical failure with exception
            _logger.LogError(ex,
                "Database error while creating product. SKU: {Sku}",
                request.Sku);
            throw;
        }
    }
}
```

**External API Calls**:
```csharp
public class PaymentService
{
    private readonly ILogger<PaymentService> _logger;
    
    public async Task<PaymentResult> ProcessPayment(PaymentRequest request)
    {
        _logger.LogInformation(
            "Calling payment gateway. OrderId: {OrderId}, Amount: {Amount}",
            request.OrderId, request.Amount);
        
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var response = await _paymentClient.ProcessAsync(request);
            
            _logger.LogInformation(
                "Payment processed successfully. OrderId: {OrderId}, TransactionId: {TransactionId}, Duration: {DurationMs}ms",
                request.OrderId, response.TransactionId, stopwatch.ElapsedMilliseconds);
            
            return PaymentResult.Success(response);
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex,
                "Payment gateway error. OrderId: {OrderId}, Duration: {DurationMs}ms",
                request.OrderId, stopwatch.ElapsedMilliseconds);
            throw;
        }
    }
}
```

**Background Jobs**:
```csharp
public class OrderProcessingJob
{
    private readonly ILogger<OrderProcessingJob> _logger;
    
    public async Task Execute()
    {
        _logger.LogInformation("Order processing job started");
        
        try
        {
            var orders = await GetPendingOrders();
            _logger.LogInformation("Processing {OrderCount} pending orders", orders.Count);
            
            foreach (var order in orders)
            {
                await ProcessOrder(order);
            }
            
            _logger.LogInformation("Order processing job completed successfully. Processed: {Count}", orders.Count);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Order processing job failed");
            throw;
        }
    }
}
```

**Authentication/Authorization**:
```csharp
// Login success
_logger.LogInformation("User logged in successfully. UserId: {UserId}, Username: {Username}",
    user.Id, user.Username);

// Login failure
_logger.LogWarning("Failed login attempt. Username: {Username}, Reason: {Reason}",
    username, "Invalid password");

// Authorization failure
_logger.LogWarning("Unauthorized access attempt. UserId: {UserId}, Resource: {Resource}, Action: {Action}",
    userId, resourceName, actionName);
```

### What NOT to Log

**Never log:**
- ❌ Passwords, API keys, secrets, tokens
- ❌ Credit card numbers, SSN, or other PII
- ❌ Full user records (log ID instead)
- ❌ Large payloads or binary data
- ❌ Inside tight loops (performance impact)

**Sanitize before logging:**
```csharp
// ❌ DON'T
_logger.LogInformation("User created: {@User}", user);

// ✅ DO - Log only ID
_logger.LogInformation("User created. UserId: {UserId}, Email: {Email}",
    user.Id, user.Email);

// ❌ DON'T
_logger.LogDebug("Login request: {@LoginRequest}", request);

// ✅ DO - Exclude password
_logger.LogDebug("Login request. Username: {Username}", request.Username);
```

### Serilog Configuration

**Program.cs Setup**:
```csharp
using Serilog;
using Serilog.Events;

// Configure Serilog early
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft.Hosting.Lifetime", LogEventLevel.Information)
    .MinimumLevel.Override("System", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithProperty("Application", "ProductAPI")
    .WriteTo.Console(
        outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext}: {Message:lj}{NewLine}{Exception}")
    .WriteTo.File(
        path: "logs/log-.txt",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 30,
        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {SourceContext} {Message:lj}{NewLine}{Exception}")
    .CreateLogger();

try
{
    Log.Information("Starting application");
    
    var builder = WebApplication.CreateBuilder(args);
    
    // Use Serilog for ASP.NET Core logging
    builder.Host.UseSerilog((context, services, configuration) => configuration
        .ReadFrom.Configuration(context.Configuration)
        .ReadFrom.Services(services)
        .Enrich.FromLogContext()
        .WriteTo.Console()
        .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day));
    
    // Build and run app
    var app = builder.Build();
    
    // Add Serilog request logging
    app.UseSerilogRequestLogging(options =>
    {
        options.MessageTemplate = "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000}ms";
        options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
        {
            diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
            diagnosticContext.Set("UserAgent", httpContext.Request.Headers["User-Agent"].ToString());
            diagnosticContext.Set("UserId", httpContext.User?.FindFirst("sub")?.Value);
        };
    });
    
    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

**appsettings.json Configuration**:
```json
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information",
        "System": "Warning",
        "Microsoft.EntityFrameworkCore": "Warning"
      }
    },
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"],
    "Properties": {
      "Application": "ProductAPI",
      "Environment": "Production"
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext}: {Message:lj}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/log-.txt",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30,
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {SourceContext} {Message:lj}{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

**appsettings.Development.json** (More Verbose):
```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Debug",
      "Override": {
        "Microsoft": "Information",
        "Microsoft.EntityFrameworkCore": "Information"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext}: {Message:lj}{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

### Structured Logging Best Practices

**Use Message Templates** (not string interpolation):
```csharp
// ✅ DO - Structured (searchable by values)
_logger.LogInformation("Product created. ProductId: {ProductId}, SKU: {Sku}",
    product.Id, product.Sku);

// ❌ DON'T - String interpolation (not searchable)
_logger.LogInformation($"Product created. ProductId: {product.Id}, SKU: {product.Sku}");
```

**Use Semantic Property Names**:
```csharp
// ✅ DO - Clear and consistent
_logger.LogInformation("Order placed. OrderId: {OrderId}, CustomerId: {CustomerId}, Total: {OrderTotal}",
    order.Id, order.CustomerId, order.Total);

// ❌ DON'T - Unclear or inconsistent
_logger.LogInformation("Order placed. Id: {Id}, Customer: {Cust}, Amount: {Amt}",
    order.Id, order.CustomerId, order.Total);
```

**Destructure Complex Objects** (when needed):
```csharp
// ✅ DO - Destructure with @ (log as object)
_logger.LogDebug("Processing order. OrderSummary: {@OrderSummary}", orderSummary);

// Output: OrderSummary: { OrderId: 123, Items: 3, Total: 99.99 }

// ❌ DON'T - Without @ (calls ToString())
_logger.LogDebug("Processing order. OrderSummary: {OrderSummary}", orderSummary);

// Output: OrderSummary: "MyApp.Models.OrderSummary"
```

### Log Context Enrichment

**Use LogContext for Scoped Properties**:
```csharp
public async Task<IActionResult> ProcessOrder(int orderId)
{
    // Push order ID into log context for all logs in this scope
    using (LogContext.PushProperty("OrderId", orderId))
    {
        _logger.LogInformation("Starting order processing");
        
        await _orderService.Process(orderId);
        // All logs in this scope will include OrderId
        
        _logger.LogInformation("Order processing completed");
    }
}

// Output:
// [12:34:56 INF] Starting order processing (OrderId: 123)
// [12:34:57 INF] Order processing completed (OrderId: 123)
```

### Performance Considerations

**Use Log Level Checks for Expensive Operations**:
```csharp
// ❌ DON'T - Always builds expensive string
_logger.LogDebug("Cache contents: {CacheContents}", 
    string.Join(", ", _cache.Select(x => x.ToString())));

// ✅ DO - Only builds if Debug is enabled
if (_logger.IsEnabled(LogLevel.Debug))
{
    var contents = string.Join(", ", _cache.Select(x => x.ToString()));
    _logger.LogDebug("Cache contents: {CacheContents}", contents);
}
```

**Async Sinks for High-Volume Logging**:
```csharp
// Use async wrapper for file logging
.WriteTo.Async(a => a.File("logs/log-.txt", rollingInterval: RollingInterval.Day))
```

### Frontend Logging (Angular)

**Console Logging in Development**:
```typescript
// environment.ts
export const environment = {
  production: false,
  logLevel: 'debug'
};

// logger.service.ts
@Injectable({ providedIn: 'root' })
export class LoggerService {
  private logLevel = environment.production ? 'warn' : 'debug';
  
  debug(message: string, ...args: unknown[]): void {
    if (this.shouldLog('debug')) {
      console.log(`[DEBUG] ${message}`, ...args);
    }
  }
  
  info(message: string, ...args: unknown[]): void {
    if (this.shouldLog('info')) {
      console.info(`[INFO] ${message}`, ...args);
    }
  }
  
  warn(message: string, ...args: unknown[]): void {
    if (this.shouldLog('warn')) {
      console.warn(`[WARN] ${message}`, ...args);
    }
  }
  
  error(message: string, error?: unknown): void {
    console.error(`[ERROR] ${message}`, error);
    // Optional: Send to backend logging endpoint
    this.sendToBackend('error', message, error);
  }
  
  private shouldLog(level: string): boolean {
    const levels = ['debug', 'info', 'warn', 'error'];
    return levels.indexOf(level) >= levels.indexOf(this.logLevel);
  }
  
  private sendToBackend(level: string, message: string, error?: unknown): void {
    // Optional: POST to /api/logs endpoint
  }
}
```

---

## Consequences

### Positive Consequences

- **Structured Logs**: Searchable by properties, not just text
- **Consistent Format**: All agents log the same way
- **Performance**: Async sinks, minimal overhead
- **Flexibility**: Easy to add new sinks (Seq, App Insights, etc.)
- **Troubleshooting**: Rich context for debugging issues
- **Compliance**: Audit trail for business operations

### Negative Consequences

- **Disk Space**: Log files can grow large (mitigated by retention policy)
- **Learning Curve**: Agents must learn structured logging patterns
- **Verbosity**: Can be noisy if too much logging enabled
- **Configuration**: Requires setup and tuning per environment

### Neutral Considerations

- **Log Aggregation**: Consider Seq, Elastic, or Application Insights for production
- **Retention**: 30 days for file logs (configurable)
- **Performance Monitoring**: Serilog has minimal impact (<1% overhead)

---

## Alternatives Considered

### Alternative 1: Built-in Microsoft.Extensions.Logging Only
- **Pros**: No external dependencies, simpler setup
- **Cons**: Less powerful, fewer sinks, not structured by default
- **Rejected**: Serilog provides much better structured logging

### Alternative 2: NLog
- **Pros**: Mature, flexible, good ASP.NET Core integration
- **Cons**: Less modern API than Serilog, lower adoption
- **Rejected**: Serilog has better structured logging and more active community

### Alternative 3: Log Everything to Database
- **Pros**: Easy to query logs with SQL
- **Cons**: Performance impact, database bloat, not log aggregation best practice
- **Rejected**: File-based or log aggregation services are better for logs

---

## Implementation Notes

### Setup Required

1. **Install Serilog NuGet Packages**:
```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Async
```

2. **Configure Program.cs** (see above)

3. **Add appsettings.json Configuration** (see above)

4. **Create LoggerService** in Angular (see above)

5. **Document Logging Guidelines** for agents

### Future Enhancements

- **Add Seq Sink** for development (searchable structured logs UI)
- **Add Application Insights** for production monitoring
- **Implement Log Correlation** across services (CorrelationId)
- **Add OpenTelemetry** for distributed tracing

### Testing

- Verify logs are written to console and files
- Test log rotation and retention
- Verify no sensitive data is logged
- Performance test with high log volume

---

## Related Decisions

- [ADR-004: Error Handling Strategy](004-error-handling-strategy.md) - Errors are logged comprehensively
- [ADR-002: C# Coding Standards](002-csharp-coding-standards.md) - Logging conventions
- [ADR-006: Technology Stack](006-technology-stack.md) - Serilog version

---

**Status**: Ready for Review  
**Action Required**: Stakeholder approval to proceed with implementation
