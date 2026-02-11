# ADR-009: Database Migration Strategy

**Status**: Accepted  
**Date**: 2026-02-10  
**Decision Makers**: Lead Software Engineer  
**Supersedes**: N/A

---

## Context

We need a strategy for managing database schema changes throughout the application lifecycle:
- Initial database creation for new environments
- Schema evolution as features are added
- Data migration for existing records when schema changes
- Rollback capability when deployments fail
- Team coordination when multiple developers change schema
- Deployment automation in CI/CD pipelines
- Support for multiple environments (dev, staging, production)

Without a clear migration strategy:
- Schema changes become error-prone manual processes
- Database state diverges between environments
- Rolling back deployments becomes difficult or impossible
- Team members overwrite each other's schema changes
- Production deployments require downtime

---

## Decision

We will use **Entity Framework Core Code-First Migrations** as our primary database migration strategy with clear conventions for creating, naming, versioning, and deploying migrations.

### Migration Approach

**Code-First Development**:
- Define entities in C# code
- Generate migrations from entity changes
- Apply migrations to database

**Migration Files**:
- Store in `Infrastructure` project under `/Migrations` folder
- Version controlled in Git
- Applied automatically on application startup (dev) or via deployment script (production)

### Migration Lifecycle

**1. Local Development**:
```bash
# Create a new migration
dotnet ef migrations add AddProductTable --project Infrastructure --startup-project API

# Review generated migration code
# Edit if needed (e.g., add seed data, custom SQL)

# Apply migration to local database
dotnet ef database update --project Infrastructure --startup-project API
```

**2. Code Review**:
- Migration files included in pull requests
- Reviewed for correctness, naming, and data safety
- Verified that `Up()` and `Down()` methods are implemented correctly

**3. Team Coordination**:
- Pull latest migrations before creating new ones
- Communicate breaking schema changes to team
- Resolve migration conflicts before merging

**4. Deployment**:
- Migrations applied automatically via deployment script or on startup
- Production migrations executed during deployment window
- Support for zero-downtime migrations when possible

### Migration Naming Convention

**Format**: `{Timestamp}_{DescriptiveAction}.cs`

Entity Framework generates timestamp automatically, focus on descriptive names:
```
20260210120000_CreateProductsTable.cs
20260210130000_AddPriceToProducts.cs
20260210140000_CreateOrdersAndOrderItemsTables.cs
20260210150000_AddIndexOnProductName.cs
20260210160000_SeedInitialCategories.cs
```

**Naming Guidelines**:
- Use PascalCase for migration names
- Start with verb: `Create`, `Add`, `Remove`, `Rename`, `Update`, `Alter`, `Seed`
- Be specific about what changed
- Keep names under 50 characters
- Use plurals for table names (following database convention)

### Migration Structure

**Standard Migration Template**:
```csharp
public partial class AddProductsTable : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Products",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 200, nullable: false),
                Description = table.Column<string>(nullable: true),
                Price = table.Column<decimal>(type: "decimal(18,2)", nullable: false),
                CategoryId = table.Column<int>(nullable: false),
                CreatedAt = table.Column<DateTime>(nullable: false),
                UpdatedAt = table.Column<DateTime>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Products", x => x.Id);
                table.ForeignKey(
                    name: "FK_Products_Categories_CategoryId",
                    column: x => x.CategoryId,
                    principalTable: "Categories",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Restrict);
            });
            
        migrationBuilder.CreateIndex(
            name: "IX_Products_CategoryId",
            table: "Products",
            column: "CategoryId");
            
        migrationBuilder.CreateIndex(
            name: "IX_Products_Name",
            table: "Products",
            column: "Name");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Products");
    }
}
```

### Data Seeding

**Two Approaches**:

**1. Initial Seed Data** (via `HasData` in Entity Configuration):
```csharp
public class CategoryConfiguration : IEntityTypeConfiguration<Category>
{
    public void Configure(EntityTypeBuilder<Category> builder)
    {
        builder.HasData(
            new Category { Id = 1, Name = "Electronics", CreatedAt = DateTime.UtcNow },
            new Category { Id = 2, Name = "Books", CreatedAt = DateTime.UtcNow },
            new Category { Id = 3, Name = "Clothing", CreatedAt = DateTime.UtcNow }
        );
    }
}
```

**2. Complex Seed Data** (via custom migration):
```csharp
public partial class SeedInitialData : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(@"
            INSERT INTO Categories (Name, Description, CreatedAt, UpdatedAt)
            VALUES 
                ('Electronics', 'Electronic devices and accessories', GETUTCDATE(), GETUTCDATE()),
                ('Books', 'Physical and digital books', GETUTCDATE(), GETUTCDATE())
        ");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql("DELETE FROM Categories WHERE Name IN ('Electronics', 'Books')");
    }
}
```

### Data Migration Pattern

**When schema changes affect existing data**:
```csharp
public partial class SplitFullNameIntoFirstAndLastName : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // 1. Add new columns
        migrationBuilder.AddColumn<string>(
            name: "FirstName",
            table: "Users",
            maxLength: 100,
            nullable: true);
            
        migrationBuilder.AddColumn<string>(
            name: "LastName",
            table: "Users",
            maxLength: 100,
            nullable: true);
        
        // 2. Migrate existing data
        migrationBuilder.Sql(@"
            UPDATE Users
            SET FirstName = LEFT(FullName, CHARINDEX(' ', FullName) - 1),
                LastName = SUBSTRING(FullName, CHARINDEX(' ', FullName) + 1, LEN(FullName))
            WHERE FullName IS NOT NULL AND CHARINDEX(' ', FullName) > 0
        ");
        
        // 3. Make columns non-nullable after data migration
        migrationBuilder.AlterColumn<string>(
            name: "FirstName",
            table: "Users",
            maxLength: 100,
            nullable: false);
            
        migrationBuilder.AlterColumn<string>(
            name: "LastName",
            table: "Users",
            maxLength: 100,
            nullable: false);
        
        // 4. Drop old column
        migrationBuilder.DropColumn(name: "FullName", table: "Users");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Reverse process
        migrationBuilder.AddColumn<string>(
            name: "FullName",
            table: "Users",
            maxLength: 200,
            nullable: true);
            
        migrationBuilder.Sql(@"
            UPDATE Users
            SET FullName = FirstName + ' ' + LastName
        ");
        
        migrationBuilder.AlterColumn<string>(
            name: "FullName",
            table: "Users",
            nullable: false);
            
        migrationBuilder.DropColumn(name: "FirstName", table: "Users");
        migrationBuilder.DropColumn(name: "LastName", table: "Users");
    }
}
```

### Migration Deployment

**Development Environment**:
```csharp
// In Program.cs - auto-apply migrations
if (app.Environment.IsDevelopment())
{
    using var scope = app.Services.CreateScope();
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    context.Database.Migrate(); // Apply pending migrations
}
```

**Production Environment**:

**Option 1: Application Startup** (for zero-downtime):
```csharp
// Apply migrations on startup
using var scope = app.Services.CreateScope();
var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
await context.Database.MigrateAsync();
```

**Option 2: Deployment Script** (recommended for production):
```bash
#!/bin/bash
# deploy.sh

echo "Applying database migrations..."
dotnet ef database update --project Infrastructure --startup-project API --connection "$PRODUCTION_CONNECTION_STRING"

if [ $? -eq 0 ]; then
    echo "Migrations applied successfully"
    # Deploy application
else
    echo "Migration failed, aborting deployment"
    exit 1
fi
```

**Option 3: SQL Script Generation** (for DBA approval):
```bash
# Generate SQL script from migrations
dotnet ef migrations script --project Infrastructure --startup-project API --idempotent --output migration.sql

# DBA reviews and executes SQL script manually
```

### Zero-Downtime Migration Strategies

**Backwards-Compatible Changes**:
- Add new columns as nullable initially
- Populate data in separate migration
- Make non-nullable in future migration
- Drop old columns in future migration after code deployment

**Expanded & Contracted Pattern**:
1. **Expand**: Add new schema elements (tables, columns)
2. **Deploy Code**: Application uses both old and new schema
3. **Migrate Data**: Background process migrates data
4. **Remove Old Code**: Update application to use only new schema
5. **Contract**: Remove old schema elements

**Example Timeline**:
```
Version 1.0: FullName column exists
Version 1.1: Add FirstName/LastName (nullable), write to both
Version 1.2: Migrate data, make FirstName/LastName non-nullable
Version 1.3: Remove FullName column
```

### DbContext Configuration

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    public DbSet<Product> Products => Set<Product>();
    public DbSet<Order> Orders => Set<Order>();
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Apply all entity configurations
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
    }
}
```

### Connection String Management

**Development** (appsettings.Development.json):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDev;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

**Production** (Environment variable or Azure Key Vault):
```bash
export ConnectionStrings__DefaultConnection="Server=prod-server;Database=MyAppProd;User Id=sa;Password=***"
```

### Migration Rollback Strategy

**Development**:
```bash
# Rollback last migration
dotnet ef database update PreviousMigrationName --project Infrastructure --startup-project API

# Remove last migration (if not applied)
dotnet ef migrations remove --project Infrastructure --startup-project API
```

**Production**:
- Have rollback scripts ready before deployment
- Test rollback in staging environment
- Consider data loss implications
- May require manual intervention for data migrations

---

## Consequences

### Positive Consequences
- **Version Control**: Database schema tracked in Git alongside code
- **Reproducibility**: Consistent database state across environments
- **Automation**: Migrations applied automatically in CI/CD
- **Team Coordination**: Clear process for schema changes
- **Type Safety**: Schema changes reflected in C# entities
- **Rollback Support**: Down() methods enable rollback (with limitations)
- **Developer Experience**: Simple CLI commands for common operations

### Negative Consequences
- **Learning Curve**: Developers must learn EF migrations
- **Complex Data Migrations**: May require custom SQL for complex scenarios
- **Rollback Limitations**: Some operations difficult to reverse (e.g., dropped columns)
- **Merge Conflicts**: Multiple developers changing schema can conflict
- **Production Risk**: Automatic migrations on startup can cause issues if migration fails

### Neutral Considerations
- Need training on EF Core migrations best practices
- Require code review for all migration files
- Must test migrations in staging before production
- May need DBA review for production migrations
- Zero-downtime migrations require careful planning

---

## Alternatives Considered

### Alternative 1: Manual SQL Scripts
- **Description**: Write and manage SQL scripts manually for all schema changes
- **Pros**: Full control, works with any database, no ORM dependency
- **Cons**: Error-prone, not version controlled easily, divergence between code and schema, manual tracking of applied scripts
- **Why rejected**: EF migrations integrate better with code-first approach and provide better developer experience

### Alternative 2: Database-First with Reverse Engineering
- **Description**: Design database first, reverse engineer to C# entities
- **Pros**: DBAs control schema, better for database-centric teams
- **Cons**: Loses type-safety benefits, schema-first doesn't fit with agile development, harder to prototype
- **Why rejected**: Code-first approach better suits our development workflow

### Alternative 3: Third-Party Tools (e.g., DbUp, Roundhouse)
- **Description**: Use dedicated database migration tools
- **Pros**: More features, better suited for complex scenarios, works with stored procedures
- **Cons**: Additional dependency, learning curve, less integration with EF Core
- **Why rejected**: EF Core migrations sufficient for our needs, one less tool to learn

### Alternative 4: No Migrations (Manual Database Management)
- **Description**: Manually create and modify database as needed
- **Pros**: Maximum flexibility, no migration framework overhead
- **Cons**: Not reproducible, error-prone, can't automate deployment, team chaos
- **Why rejected**: Unacceptable for professional software development

---

## Implementation Notes

### Project Setup

**1. Install EF Core Tools**:
```bash
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
```

**2. Configure DbContext in Program.cs**:
```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        b => b.MigrationsAssembly("Infrastructure")));
```

**3. Initial Migration**:
```bash
dotnet ef migrations add InitialCreate --project Infrastructure --startup-project API
dotnet ef database update --project Infrastructure --startup-project API
```

### Best Practices

1. **Review Generated Migrations**: Always review migration code before committing
2. **Test Rollback**: Test Down() method works before merging
3. **Idempotent Scripts**: Use `--idempotent` flag for production SQL scripts
4. **Backup Before Migration**: Always backup production database before applying migrations
5. **Staging First**: Test all migrations in staging environment
6. **Communicate Breaking Changes**: Notify team of schema changes that affect existing code
7. **Small Migrations**: Keep migrations focused on single logical change
8. **Document Complex Migrations**: Add XML comments to explain complex data migrations

### Common Commands

```bash
# List all migrations
dotnet ef migrations list --project Infrastructure --startup-project API

# Generate SQL script
dotnet ef migrations script --project Infrastructure --startup-project API --idempotent

# Check pending migrations
dotnet ef migrations has-pending-model-changes --project Infrastructure --startup-project API

# Remove last migration (if not applied to database)
dotnet ef migrations remove --project Infrastructure --startup-project API

# Update to specific migration
dotnet ef database update MigrationName --project Infrastructure --startup-project API

# Revert all migrations
dotnet ef database update 0 --project Infrastructure --startup-project API
```

### Testing Requirements

- **Unit Tests**: Test entity configurations using in-memory database
- **Integration Tests**: Test that migrations create expected schema
- **Migration Tests**: Test Up() and Down() methods work correctly
- **Data Migration Tests**: Verify data transformations work correctly

---

## References

- [Entity Framework Core Migrations](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/)
- [EF Core Migration Best Practices](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/managing)
- [Automated Deployment with EF Core](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying)
- [Zero-Downtime Deployment Strategies](https://www.infoq.com/articles/zero-downtime-database-deployments/)
- [Expand Contract Pattern](https://martinfowler.com/bliki/ParallelChange.html)
