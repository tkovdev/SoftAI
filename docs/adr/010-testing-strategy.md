# ADR-010: Testing Strategy

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer  
**Supersedes**: N/A

---

## Context

Comprehensive automated testing is essential for:
- Catching bugs early in development
- Enabling confident refactoring
- Documenting expected behavior
- Supporting continuous integration/deployment
- Reducing manual QA effort
- Maintaining code quality over time

We need a clear strategy covering:
- Types of tests (unit, integration, E2E)
- Testing frameworks and tools
- Code coverage requirements
- Test organization and naming conventions
- Mocking strategies
- CI integration
- Performance testing

Without a testing strategy:
- Test coverage is inconsistent
- Critical paths remain untested
- Bugs reach production
- Refactoring becomes risky
- Development velocity decreases

---

## Decision

We will implement a **comprehensive testing pyramid** with emphasis on unit tests, supported by integration tests and critical E2E tests, using MSTest for backend and Jasmine/Karma for frontend.

### Testing Pyramid

```
        /\
       /  \      E2E Tests (5-10%)
      /____\     - Critical user flows
     /      \    - Cypress or Playwright
    /        \   
   /  Integration Tests (20-30%)
  /____________\ - API endpoints
 /              \- Database interactions
/                \
/   Unit Tests    \  (60-75%)
/__________________\ - Business logic
                     - CQRS handlers
                     - Services
                     - Components
```

### Backend Testing (.NET)

**Framework**: **MSTest** (Microsoft Test Framework)

**Why MSTest**:
- First-class support in Visual Studio and VS Code
- Native integration with .NET
- Excellent performance
- Modern features (parallel execution, data-driven tests)
- Great tooling support

**Package Stack**:
```xml
<ItemGroup>
    <PackageReference Include="MSTest.TestFramework" Version="3.x" />
    <PackageReference Include="MSTest.TestAdapter" Version="3.x" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.x" />
    <PackageReference Include="Moq" Version="4.20.x" />
    <PackageReference Include="FluentAssertions" Version="6.x" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="10.x" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="10.x" />
</ItemGroup>
```

### Frontend Testing (Angular)

**Framework**: **Jasmine + Karma** (Angular defaults)

**Package Stack**:
- Jasmine for test structure and assertions
- Karma for test runner
- Cypress or Playwright for E2E tests

**E2E Framework**: **Playwright** (preferred) or Cypress
- Playwright: Faster, better debugging, multi-browser support
- Cypress: Simpler API, real-time reloading

---

## Testing Types & Standards

### 1. Unit Tests (60-75% of tests)

**Purpose**: Test individual components in isolation

**Backend Unit Tests** (.NET):

**Target**: CQRS handlers, validators, services, domain logic

**Example**:
```csharp
[TestClass]
public class CreateProductCommandHandlerTests
{
    private Mock<IApplicationDbContext> _mockContext;
    private Mock<ICurrentUserService> _mockCurrentUser;
    private CreateProductCommandHandler _handler;
    
    [TestInitialize]
    public void Setup()
    {
        _mockContext = new Mock<IApplicationDbContext>();
        _mockCurrentUser = new Mock<ICurrentUserService>();
        _handler = new CreateProductCommandHandler(_mockContext.Object, _mockCurrentUser.Object);
    }
    
    [TestMethod]
    public async Task Handle_ValidProduct_ShouldCreateProduct()
    {
        // Arrange
        var command = new CreateProductCommand("Test Product", "Description", 19.99m, 1);
        var mockDbSet = new Mock<DbSet<Product>>();
        _mockContext.Setup(c => c.Products).Returns(mockDbSet.Object);
        _mockCurrentUser.Setup(u => u.UserId).Returns("user-123");
        
        // Act
        var result = await _handler.Handle(command, CancellationToken.None);
        
        // Assert
        result.Should().NotBeNull();
        result.Name.Should().Be("Test Product");
        mockDbSet.Verify(m => m.Add(It.IsAny<Product>()), Times.Once);
        _mockContext.Verify(m => m.SaveChangesAsync(It.IsAny<CancellationToken>()), Times.Once);
    }
    
    [TestMethod]
    public async Task Handle_NegativePrice_ShouldThrowValidationException()
    {
        // Arrange
        var command = new CreateProductCommand("Test Product", "Description", -10m, 1);
        
        // Act & Assert
        await Assert.ThrowsExceptionAsync<ValidationException>(() => 
            _handler.Handle(command, CancellationToken.None));
    }
}
```

**Frontend Unit Tests** (Angular):

**Target**: Components, services, pipes, directives

**Example**:
```typescript
describe('ProductListComponent', () => {
  let component: ProductListComponent;
  let fixture: ComponentFixture<ProductListComponent>;
  let mockProductService: jasmine.SpyObj<ProductService>;
  
  beforeEach(() => {
    mockProductService = jasmine.createSpyObj('ProductService', ['getProducts']);
    
    TestBed.configureTestingModule({
      imports: [ProductListComponent],
      providers: [
        { provide: ProductService, useValue: mockProductService }
      ]
    });
    
    fixture = TestBed.createComponent(ProductListComponent);
    component = fixture.componentInstance;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should load products on init', () => {
    const mockProducts: Product[] = [
      { id: 1, name: 'Product 1', price: 10 },
      { id: 2, name: 'Product 2', price: 20 }
    ];
    mockProductService.getProducts.and.returnValue(of(mockProducts));
    
    component.ngOnInit();
    
    expect(component.products()).toEqual(mockProducts);
    expect(mockProductService.getProducts).toHaveBeenCalled();
  });
  
  it('should handle loading error', () => {
    const mockError = new Error('Failed to load');
    mockProductService.getProducts.and.returnValue(throwError(() => mockError));
    
    component.ngOnInit();
    
    expect(component.error()).toBe('Failed to load products');
  });
});
```

**Coverage Target**: 80% minimum for business logic

### 2. Integration Tests (20-30% of tests)

**Purpose**: Test component interactions, API endpoints, database operations

**Backend Integration Tests** (.NET):

**Target**: API controllers, database context, external services

**Example**:
```csharp
[TestClass]
public class ProductsControllerIntegrationTests : IDisposable
{
    private WebApplicationFactory<Program> _factory;
    private HttpClient _client;
    private ApplicationDbContext _dbContext;
    
    [TestInitialize]
    public void Setup()
    {
        _factory = new WebApplicationFactory<Program>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureServices(services =>
                {
                    // Replace with in-memory database
                    var descriptor = services.SingleOrDefault(
                        d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));
                    if (descriptor != null) services.Remove(descriptor);
                    
                    services.AddDbContext<ApplicationDbContext>(options =>
                        options.UseInMemoryDatabase("TestDb"));
                });
            });
            
        _client = _factory.CreateClient();
        _dbContext = _factory.Services.CreateScope().ServiceProvider
            .GetRequiredService<ApplicationDbContext>();
    }
    
    [TestMethod]
    public async Task GetProducts_ReturnsSuccessStatusCode()
    {
        // Arrange
        await SeedDatabase();
        
        // Act
        var response = await _client.GetAsync("/api/v1/products");
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        var content = await response.Content.ReadAsStringAsync();
        var products = JsonSerializer.Deserialize<PaginatedList<ProductResponse>>(
            content, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        products.Items.Should().HaveCount(2);
    }
    
    [TestMethod]
    public async Task CreateProduct_ValidProduct_ReturnsCreatedProduct()
    {
        // Arrange
        var request = new CreateProductRequest("New Product", "Description", 29.99m, 1);
        var content = new StringContent(
            JsonSerializer.Serialize(request), 
            Encoding.UTF8, 
            "application/json");
        
        // Act
        var response = await _client.PostAsync("/api/v1/products", content);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        response.Headers.Location.Should().NotBeNull();
        var product = await response.Content.ReadFromJsonAsync<ProductResponse>();
        product.Name.Should().Be("New Product");
    }
    
    [TestMethod]
    public async Task CreateProduct_InvalidProduct_ReturnsBadRequest()
    {
        // Arrange
        var request = new CreateProductRequest("", "Description", -10m, 1); // Invalid
        var content = new StringContent(
            JsonSerializer.Serialize(request), 
            Encoding.UTF8, 
            "application/json");
        
        // Act
        var response = await _client.PostAsync("/api/v1/products", content);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
        var problemDetails = await response.Content.ReadFromJsonAsync<ProblemDetails>();
        problemDetails.Should().NotBeNull();
    }
    
    private async Task SeedDatabase()
    {
        _dbContext.Products.AddRange(
            new Product { Id = 1, Name = "Product 1", Price = 10m, CategoryId = 1 },
            new Product { Id = 2, Name = "Product 2", Price = 20m, CategoryId = 1 }
        );
        await _dbContext.SaveChangesAsync();
    }
    
    public void Dispose()
    {
        _dbContext.Database.EnsureDeleted();
        _dbContext.Dispose();
        _client.Dispose();
        _factory.Dispose();
    }
}
```

**Frontend Integration Tests** (Angular):

**Target**: Component + service + HTTP interactions

**Example**:
```typescript
describe('ProductListComponent Integration', () => {
  let component: ProductListComponent;
  let fixture: ComponentFixture<ProductListComponent>;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [
        ProductListComponent,
        HttpClientTestingModule
      ],
      providers: [ProductService]
    });
    
    fixture = TestBed.createComponent(ProductListComponent);
    component = fixture.componentInstance;
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify();
  });
  
  it('should fetch and display products', fakeAsync(() => {
    const mockProducts: Product[] = [
      { id: 1, name: 'Product 1', price: 10 },
      { id: 2, name: 'Product 2', price: 20 }
    ];
    
    fixture.detectChanges(); // Triggers ngOnInit
    
    const req = httpMock.expectOne('/api/v1/products');
    expect(req.request.method).toBe('GET');
    req.flush({ items: mockProducts, pagination: { totalCount: 2 } });
    
    tick();
    fixture.detectChanges();
    
    const compiled = fixture.nativeElement;
    const productElements = compiled.querySelectorAll('.product-item');
    expect(productElements.length).toBe(2);
    expect(productElements[0].textContent).toContain('Product 1');
  }));
});
```

**Coverage Target**: All API endpoints, all critical service interactions

### 3. End-to-End Tests (5-10% of tests)

**Purpose**: Test complete user workflows from UI to database

**Target**: Critical business flows only

**Framework**: Playwright (preferred)

**Example**:
```typescript
import { test, expect } from '@playwright/test';

test.describe('Product Management', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:4200');
    await page.click('text=Login');
    await page.fill('input[name="username"]', 'testuser');
    await page.fill('input[name="password"]', 'Test123!');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL(/.*dashboard/);
  });
  
  test('should create a new product', async ({ page }) => {
    // Navigate to products
    await page.click('text=Products');
    await expect(page).toHaveURL(/.*products/);
    
    // Click create button
    await page.click('button:has-text("Create Product")');
    
    // Fill form
    await page.fill('input[name="name"]', 'Test Product');
    await page.fill('textarea[name="description"]', 'Test Description');
    await page.fill('input[name="price"]', '29.99');
    await page.selectOption('select[name="category"]', '1');
    
    // Submit
    await page.click('button:has-text("Save")');
    
    // Verify success
    await expect(page.locator('.success-message')).toContainText('Product created');
    await expect(page.locator('.product-list')).toContainText('Test Product');
  });
  
  test('should validate product form', async ({ page }) => {
    await page.click('text=Products');
    await page.click('button:has-text("Create Product")');
    
    // Submit without filling required fields
    await page.click('button:has-text("Save")');
    
    // Verify validation errors
    await expect(page.locator('.error-message')).toContainText('Name is required');
    await expect(page.locator('.error-message')).toContainText('Price is required');
  });
  
  test('should search and filter products', async ({ page }) => {
    await page.click('text=Products');
    
    // Search
    await page.fill('input[placeholder="Search products"]', 'laptop');
    await page.waitForTimeout(500); // Debounce
    
    // Verify filtered results
    const products = page.locator('.product-item');
    await expect(products).toHaveCount(3);
    await expect(products.first()).toContainText('laptop');
  });
});
```

**Coverage Target**: All critical user flows (checkout, registration, key workflows)

---

## Testing Conventions

### Naming Conventions

**Backend** (MSTest):
- Test class: `{ClassUnderTest}Tests`
- Test method: `{MethodName}_{Scenario}_{ExpectedBehavior}`

**Examples**:
```csharp
[TestClass]
public class CreateProductCommandHandlerTests
{
    [TestMethod]
    public async Task Handle_ValidProduct_ShouldCreateProduct() { }
    
    [TestMethod]
    public async Task Handle_DuplicateName_ShouldThrowException() { }
    
    [TestMethod]
    public async Task Handle_InvalidCategory_ShouldReturnNotFound() { }
}
```

**Frontend** (Jasmine):
- Test suite: `describe('{ComponentName}', ...)`
- Test case: `it('should {expected behavior}', ...)`

**Examples**:
```typescript
describe('ProductListComponent', () => {
  it('should create', () => { });
  it('should load products on init', () => { });
  it('should handle loading error', () => { });
  it('should display empty message when no products', () => { });
});
```

### Test Organization

**Backend Project Structure**:
```
tests/
├── MyApp.UnitTests/
│   ├── Application/
│   │   ├── Products/
│   │   │   ├── Commands/
│   │   │   │   └── CreateProductCommandHandlerTests.cs
│   │   │   └── Queries/
│   │   │       └── GetProductsQueryHandlerTests.cs
│   │   └── Validators/
│   │       └── CreateProductValidatorTests.cs
│   └── Domain/
│       └── Entities/
│           └── ProductTests.cs
│
└── MyApp.IntegrationTests/
    ├── Controllers/
    │   └── ProductsControllerTests.cs
    ├── Infrastructure/
    │   └── Repositories/
    │       └── ProductRepositoryTests.cs
    └── TestWebApplicationFactory.cs
```

**Frontend Project Structure**:
```
src/app/
├── products/
│   ├── product-list/
│   │   ├── product-list.component.ts
│   │   ├── product-list.component.spec.ts  (unit tests)
│   │   └── product-list.component.html
│   └── services/
│       ├── product.service.ts
│       └── product.service.spec.ts
└── e2e/
    └── products.spec.ts  (E2E tests)
```

### AAA Pattern (Arrange-Act-Assert)

All tests should follow AAA structure:

```csharp
[TestMethod]
public async Task Example_Test()
{
    // Arrange - Set up test data and dependencies
    var command = new CreateProductCommand(...);
    var mockService = new Mock<IService>();
    mockService.Setup(m => m.DoSomething()).Returns(...);
    
    // Act - Execute the method under test
    var result = await _handler.Handle(command, CancellationToken.None);
    
    // Assert - Verify the expected outcome
    result.Should().NotBeNull();
    result.Id.Should().BeGreaterThan(0);
    mockService.Verify(m => m.DoSomething(), Times.Once);
}
```

### Mocking Strategy

**Backend (.NET)**:
- Use **Moq** for mocking dependencies
- Mock interfaces, not concrete classes
- Use `Mock<T>` for creating mocks
- Use `Verify()` to assert method calls

**Frontend (Angular)**:
- Use **Jasmine spies** for mocking
- Mock HTTP calls with `HttpClientTestingModule`
- Use `jasmine.createSpyObj()` for service mocks

---

## Code Coverage Requirements

**Minimum Coverage Targets**:
- **Overall**: 70%
- **Business Logic** (CQRS handlers, services): 85%
- **Controllers/Components**: 75%
- **Infrastructure**: 60%

**Measure Coverage**:

**Backend**:
```bash
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura
```

**Frontend**:
```bash
ng test --code-coverage
```

**CI Integration**: Fail build if coverage drops below threshold

---

## CI/CD Integration

**Run Tests in CI** (GitHub Actions):
```yaml
- name: Run Backend Tests
  run: dotnet test --configuration Release --logger trx --results-directory ./test-results
  
- name: Run Frontend Tests
  run: npm run test:ci
  
- name: Run E2E Tests
  run: npx playwright test
```

**Test Reports**:
- Publish test results to CI dashboard
- Track coverage trends over time
- Block PR merge if tests fail

---

## Consequences

### Positive Consequences
- **Confidence**: Refactor safely with comprehensive test coverage
- **Early Bug Detection**: Catch issues before they reach production
- **Documentation**: Tests serve as executable documentation
- **Faster Development**: Less time debugging, more time building
- **CI/CD Enablement**: Automated testing enables continuous deployment
- **Maintainability**: Tests ensure code remains correct as it evolves

### Negative Consequences
- **Initial Time Investment**: Writing tests takes time upfront
- **Maintenance Overhead**: Tests need updates when code changes
- **Test Flakiness**: E2E tests can be flaky and require maintenance
- **Learning Curve**: Team needs to learn testing best practices

### Neutral Considerations
- Need to educate team on testing practices
- Require code review for test quality
- May slow down initial feature development
- Need to maintain test infrastructure (CI, E2E environment)

---

## Alternatives Considered

### Alternative 1: xUnit Instead of MSTest
- **Description**: Use xUnit as primary backend testing framework
- **Pros**: Modern, popular in .NET community, excellent design
- **Cons**: MSTest has better Visual Studio integration, stakeholder preference
- **Why rejected**: MSTest selected per stakeholder requirement (ADR-006)

### Alternative 2: Cypress Instead of Playwright
- **Description**: Use Cypress for E2E testing
- **Pros**: Simpler API, excellent developer experience, real-time reloading
- **Cons**: Slower, single browser at a time, some limitations
- **Why rejected**: Playwright offers better performance and multi-browser support, though Cypress is acceptable alternative

### Alternative 3: No Integration Tests
- **Description**: Only unit and E2E tests
- **Pros**: Simpler, fewer tests to maintain
- **Cons**: Gap between unit and E2E, slower E2E tests, misses API contract issues
- **Why rejected**: Integration tests provide essential API coverage

---

## Implementation Notes

Run tests:
```bash
# Backend
dotnet test

# Frontend
ng test
ng e2e

# With coverage
dotnet test /p:CollectCoverage=true
ng test --code-coverage
```

---

## References

- [MSTest Documentation](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest)
- [ASP.NET Core Integration Testing](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests)
- [Angular Testing Guide](https://angular.dev/guide/testing)
- [Playwright Documentation](https://playwright.dev/)
- [Moq Documentation](https://github.com/moq/moq4)
- [Testing Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
