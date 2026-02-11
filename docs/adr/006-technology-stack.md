# ADR-006: Technology Stack and Versions

**Status**: Draft (Pending Review)  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer, Stakeholder  
**Supersedes**: N/A

---

## Context

We need to lock down specific versions of all major technologies and frameworks to ensure:
- Consistent development and production environments
- Reproducible builds
- Compatibility between components
- Clear upgrade path
- Stability and support lifecycle

Without version locking, agents may use different versions leading to:
- Incompatibility issues
- Difficult to reproducebugs
- Unclear dependencies
- Deployment failures

---

## Decision

We will use the following technology stack with these specific versions:

### Backend Stack

| Technology | Version | Justification |
|------------|---------|---------------|
| **.NET** | **10.0** | Latest LTS (Long-Term Support), modern features, supported until Nov 2027 |
| **C#** | **13.0** | Included with .NET 10, latest language features |
| **ASP.NET Core** | **10.0** | Web API framework, matches .NET version |
| **Entity Framework Core** | **10.0.x** | ORM for SQL Server, matches .NET version |
| **MediatR** | **12.x** (latest stable) | CQRS pattern implementation |
| **FluentValidation** | **11.x** (latest stable) | Declarative validation library |
| **Serilog** | **4.x** (latest stable) | Structured logging library |
| **Serilog.AspNetCore** | **8.x** (latest stable) | ASP.NET Core integration |
| **AutoMapper** | **13.x** (latest stable) | Object-to-object mapping (optional use) |
| **xUnit** | **2.6.x** (latest stable) | Unit testing framework |
| **Moq** | **4.20.x** (latest stable) | Mocking library for tests |
| **FluentAssertions** | **6.x** (latest stable) | Assertion library for tests |
| **SQL Server** | **2022** (or **2019**) | Relational database, stakeholder choice |

### Frontend Stack

| Technology | Version | Justification |
|------------|---------|---------------|
| **Node.js** | **22.x LTS** | JavaScript runtime for build tools |
| **Angular** | **21.x** (latest major) | Frontend framework with signals and standalone components |
| **TypeScript** | **5.7.x** (bundled with Angular 21) | Type-safe JavaScript |
| **PrimeNG** | **18.x** (latest major) | UI component library, stakeholder choice |
| **PrimeIcons** | **7.x** | Icons for PrimeNG |
| **RxJS** | **7.8.x** (bundled with Angular) | Reactive programming library |
| **Jasmine** | **5.x** (Angular default) | Unit testing framework |
| **Karma** | **6.x** (Angular default) | Test runner |
| **Cypress** or **Playwright** | **13.x** / **1.x** | E2E testing (to be chosen based on needs) |

### Development Tools

| Tool | Version | Purpose |
|------|---------|---------|
| **StyleCop.Analyzers** | **1.2.x** | C# code style enforcement |
| **ESLint** | **9.x** | TypeScript/JavaScript linting |
| **Prettier** | **3.x** | Code formatting for frontend |
| **Swagger/Swashbuckle** | **6.x** | API documentation |
| **Docker** | **27.x** (latest) | Containerization |

### DevOps Stack (Future Implementation)

| Technology | Version | Notes |
|------------|---------|-------|
| **Docker Compose** | **2.x** | Multi-container orchestration (dev) |
| **GitHub Actions** | **Latest** | CI/CD workflows |
| **Azure DevOps** | **Cloud** | Alternative CI/CD option |

---

## Detailed Configurations

### .NET 10 Project Configuration

**Global.json** (SDK Version):
```json
{
  "sdk": {
    "version": "10.0.100",
    "rollForward": "latestMinor",
    "allowPrerelease": false
  }
}
```

**Project Files (.csproj)**:
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <LangVersion>13.0</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <!-- Web API -->
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="10.0.*" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.*" />
    
    <!-- Entity Framework -->
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.*" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="10.0.*" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="10.0.*">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    
    <!-- CQRS and Validation -->
    <PackageReference Include="MediatR" Version="12.*" />
    <PackageReference Include="FluentValidation.DependencyInjectionExtensions" Version="11.*" />
    
    <!-- Logging -->
    <PackageReference Include="Serilog.AspNetCore" Version="8.*" />
    <PackageReference Include="Serilog.Sinks.Console" Version="6.*" />
    <PackageReference Include="Serilog.Sinks.File" Version="6.*" />
    <PackageReference Include="Serilog.Sinks.Async" Version="2.*" />
    
    <!-- Mapping (optional) -->
    <PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="13.*" />
    
    <!-- Code Quality -->
    <PackageReference Include="StyleCop.Analyzers" Version="1.2.*">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

**Test Project (.csproj)**:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.*" />
    <PackageReference Include="xUnit" Version="2.6.*" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.*">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="Moq" Version="4.20.*" />
    <PackageReference Include="FluentAssertions" Version="6.*" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="10.0.*" />
  </ItemGroup>
</Project>
```

### Angular 21 Configuration

**package.json**:
```json
{
  "name": "product-app",
  "version": "1.0.0",
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test",
    "lint": "ng lint"
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "^21.0.0",
    "@angular/common": "^21.0.0",
    "@angular/compiler": "^21.0.0",
    "@angular/core": "^21.0.0",
    "@angular/forms": "^21.0.0",
    "@angular/platform-browser": "^21.0.0",
    "@angular/platform-browser-dynamic": "^21.0.0",
    "@angular/router": "^21.0.0",
    "primeng": "^18.0.0",
    "primeicons": "^7.0.0",
    "rxjs": "^7.8.0",
    "tslib": "^2.6.0",
    "zone.js": "^0.15.0"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^21.0.0",
    "@angular/cli": "^21.0.0",
    "@angular/compiler-cli": "^21.0.0",
    "@angular-eslint/builder": "^19.0.0",
    "@angular-eslint/eslint-plugin": "^19.0.0",
    "@angular-eslint/eslint-plugin-template": "^19.0.0",
    "@angular-eslint/schematics": "^19.0.0",
    "@angular-eslint/template-parser": "^19.0.0",
    "@types/jasmine": "^5.1.0",
    "@typescript-eslint/eslint-plugin": "^8.0.0",
    "@typescript-eslint/parser": "^8.0.0",
    "eslint": "^9.0.0",
    "jasmine-core": "^5.1.0",
    "karma": "^6.4.0",
    "karma-chrome-launcher": "^3.2.0",
    "karma-coverage": "^2.2.0",
    "karma-jasmine": "^5.1.0",
    "karma-jasmine-html-reporter": "^2.1.0",
    "prettier": "^3.0.0",
    "typescript": "~5.7.0"
  }
}
```

**angular.json** (Key Settings):
```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "product-app": {
      "projectType": "application",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:application",
          "options": {
            "outputPath": "dist/product-app",
            "index": "src/index.html",
            "browser": "src/main.ts",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": [
              "node_modules/primeng/resources/themes/lara-light-blue/theme.css",
              "node_modules/primeng/resources/primeng.css",
              "node_modules/primeicons/primeicons.css",
              "src/styles.scss"
            ],
            "scripts": []
          }
        }
      }
    }
  }
}
```

**tsconfig.json**:
```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "./dist/out-tsc",
    "strict": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "declaration": false,
    "moduleResolution": "bundler",
    "importHelpers": true,
    "target": "ES2022",
    "module": "ES2022",
    "lib": ["ES2022", "dom"],
    "useDefineForClassFields": false
  },
  "angularCompilerOptions": {
    "enableI18nLegacyMessageIdFormat": false,
    "strictInjectionParameters": true,
    "strictInputAccessModifiers": true,
    "strictTemplates": true
  }
}
```

### SQL Server Configuration

**Connection String Format**:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=ProductDb;User Id=sa;Password=YourStrong!Password;TrustServerCertificate=True"
  }
}
```

**Supported Versions**:
- **SQL Server 2022** (Recommended - latest features)
- **SQL Server 2019** (Supported - stable, widely deployed)
- **Azure SQL Database** (Fully compatible)

**Entity Framework Core Setup**:
```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        sqlOptions =>
        {
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 3,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: null);
            sqlOptions.CommandTimeout(30);
        }));
```

---

## Version Upgrade Policy

### Major Version Upgrades (Breaking Changes)

**When**: Annually or as needed for critical features  
**Process**:
1. Evaluate new features and breaking changes
2. Create spike/proof-of-concept in branch
3. Document migration strategy in new ADR
4. Update all agent specifications
5. Implement and test thoroughly
6. Deploy to production after validation

**Examples**: .NET 10 → .NET 11, Angular 21 → Angular 22

### Minor/Patch Version Upgrades (Non-Breaking)

**When**: Quarterly or as needed for security fixes  
**Process**:
1. Review release notes for changes
2. Update version numbers in config files
3. Test in development environment
4. Deploy to production if stable

**Examples**: .NET 10.0.1 → 10.0.2, Angular 21.0 → 21.1

### Security Patches (Critical)

**When**: Immediately upon release  
**Process**:
1. Assess severity and impact
2. Apply patch in development
3. Run smoke tests
4. Deploy to production ASAP

### Dependency Updates

**Automated**: Use Dependabot or Renovate Bot for automated PRs  
**Review**: Lead Engineer reviews and approves dependency updates  
**Cadence**: Weekly for patches, monthly for minor updates

---

## Consequences

### Positive Consequences

- **Reproducible Builds**: Same versions everywhere (dev, test, prod)
- **Stability**: Tested and stable versions
- **Support**: Enterprise support for .NET 10 LTS (until 2027)
- **Modern Features**: Latest C# 13, Angular 21 signals, .NET 10 performance
- **Clear Upgrade Path**: Documented process for version upgrades
- **Compatibility**: All components tested together

### Negative Consequences

- **Version Lock-in**: Can't use features from newer versions until upgrade
- **Maintenance**: Must periodically review and update versions
- **Security**: Old versions may have vulnerabilities (mitigated by patch policy)
- **Legacy**: Will become legacy as new versions release (managed by upgrade policy)

### Neutral Considerations

- **.NET 10 LTS**: Supported until November 2027 (plenty of time)
- **Angular 21**: Major version, expect frequent minor updates
- **SQL Server 2022**: Latest version, long support lifecycle
- **PrimeNG**: Active development, frequent updates

---

## Alternatives Considered

### Alternative 1: .NET 9 (Non-LTS)
- **Pros**: Slightly more modern than .NET 10 initially
- **Cons**: Not LTS, support ends May 2025 (too soon)
- **Rejected**: .NET 10 LTS is better choice for stability

### Alternative 2: Older Versions (.NET 8, Angular 17)
- **Pros**: More mature, fewer breaking changes
- **Cons**: Missing modern features (signals, .NET 10 performance), shorter support
- **Rejected**: Want to use latest stable LTS features

### Alternative 3: PostgreSQL Instead of SQL Server
- **Pros**: Open source, free, good performance
- **Cons**: Stakeholder chose SQL Server
- **Rejected**: SQL Server is enterprise-standard and stakeholder preference

### Alternative 4: Angular Material Instead of PrimeNG
- **Pros**: Official Angular UI library
- **Cons**: Stakeholder chose PrimeNG for richer components
- **Rejected**: PrimeNG chosen for better business components

---

## Implementation Notes

### Setup Checklist

**Backend**:
- [ ] Install .NET 10 SDK
- [ ] Create `global.json` with SDK version
- [ ] Set up solution with project structure
- [ ] Add NuGet packages with versions
- [ ] Configure SQL Server connection
- [ ] Run EF Core migrations

**Frontend**:
- [ ] Install Node.js 22 LTS
- [ ] Create Angular 21 workspace
- [ ] Install PrimeNG and dependencies
- [ ] Configure `angular.json` for PrimeNG styles
- [ ] Set up ESLint and Prettier
- [ ] Configure TypeScript strict mode

**Development Environment**:
- [ ] Install Visual Studio Code or Visual Studio 2022 or Rider
- [ ] Install SQL Server 2022 (or Docker container)
- [ ] Install Node.js 22 LTS
- [ ] Install Angular CLI: `npm install -g @angular/cli@21`
- [ ] Install .NET EF Core tools: `dotnet tool install --global dotnet-ef`

### Docker Configuration (Development)

**SQL Server Container**:
```bash
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong!Password" \
  -p 1433:1433 --name sqlserver \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

**docker-compose.yml** (Future):
```yaml
version: '3.8'
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong!Password
    ports:
      - "1433:1433"
    volumes:
      - sqlserver-data:/var/opt/mssql

  api:
    build: ./src/ProductApi
    ports:
      - "5000:8080"
    depends_on:
      - sqlserver
    environment:
      - ConnectionStrings__DefaultConnection=Server=sqlserver,1433;Database=ProductDb;User Id=sa;Password=YourStrong!Password;TrustServerCertificate=True

  webapp:
    build: ./src/ProductApp
    ports:
      - "4200:80"
    depends_on:
      - api

volumes:
  sqlserver-data:
```

---

## Related Decisions

- [ADR-002: C# Coding Standards](002-csharp-coding-standards.md) - C# 13 language features
- [ADR-003: TypeScript/Angular Standards](003-typescript-angular-coding-standards.md) - Angular 21 and PrimeNG usage
- [ADR-004: Error Handling](004-error-handling-strategy.md) - Framework error handling features
- [ADR-005: Logging Strategy](005-logging-strategy.md) - Serilog version and configuration

---

**Status**: Ready for Review  
**Action Required**: Stakeholder approval to proceed with implementation
