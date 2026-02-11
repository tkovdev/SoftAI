# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records documenting significant architectural and design decisions made throughout the project.

---

## What is an ADR?

An Architecture Decision Record captures an important architectural decision made along with its context and consequences.

### When to Create an ADR

Create an ADR when making decisions about:
- **Architecture**: Layering, patterns (e.g., CQRS, Repository), separation of concerns
- **Technology Selection**: Frameworks, libraries, databases, tools
- **Integration Patterns**: How systems communicate, API design, messaging
- **Security**: Authentication/authorization approaches, encryption, data protection
- **Performance**: Caching strategies, optimization approaches
- **Scalability**: Horizontal vs vertical scaling, partitioning strategies
- **Data Management**: Schema design, migration approaches, data sovereignty

### When NOT to Create an ADR

Skip ADRs for:
- Minor implementation details
- Obvious or conventional choices
- Temporary workarounds
- Personal coding preferences (covered by style guides)

---

## ADR Naming Convention

Use sequential numbering with descriptive titles:

```
001-architecture-clean-architecture.md
002-backend-cqrs-with-mediatr.md
003-frontend-angular-signals.md
004-database-postgresql-selection.md
005-authentication-jwt-tokens.md
```

Format: `NNN-topic-brief-description.md`
- **NNN**: Zero-padded sequential number (001, 002, 003, ...)
- **topic**: Category (architecture, backend, frontend, database, security, devops, etc.)
- **brief-description**: Hyphenated lowercase description

---

## ADR Statuses

- **Proposed**: Decision is under consideration
- **Accepted**: Decision is approved and should be implemented
- **Deprecated**: Decision is no longer valid but kept for historical context
- **Superseded**: Decision has been replaced (link to replacement ADR)

---

## Creating a New ADR

1. Copy `TEMPLATE.md` to a new file following naming convention
2. Fill in all sections thoughtfully
3. Have it reviewed by Lead Software Engineer
4. Update status to "Accepted" when approved
5. Link from `.context/PROJECT_STATE.md` if currently relevant
6. Reference in code comments where decision is implemented

---

## ADR Index

### Active Decisions

- **[001-backend-cqrs-with-mediatr.md](001-backend-cqrs-with-mediatr.md)** - Use CQRS pattern with MediatR for backend business logic (Status: Accepted, Example)

### Accepted (Sprint 0)

- **[002-csharp-coding-standards.md](002-csharp-coding-standards.md)** - C# naming conventions, StyleCop rules, async/await patterns
- **[003-typescript-angular-coding-standards.md](003-typescript-angular-coding-standards.md)** - TypeScript, Angular 21 standalone components, signals, PrimeNG
- **[004-error-handling-strategy.md](004-error-handling-strategy.md)** - Result<T> pattern, RFC 7807 Problem Details, global exception handling
- **[005-logging-strategy.md](005-logging-strategy.md)** - Serilog structured logging, log levels, what to log
- **[006-technology-stack.md](006-technology-stack.md)** - .NET 10, Angular 21, SQL Server 2022, MSTest, package versions

### Superseded Decisions

(None yet)

---

## Best Practices

### Writing Good ADRs

**Be specific**: Provide enough detail for someone to understand the decision
**Capture context**: Explain _why_ this decision was necessary
**Document trade-offs**: Be honest about negative consequences
**Consider alternatives**: Show you evaluated multiple options
**Keep it timeless**: Write for future developers who lack current context

### Example Quality Indicators

✅ **Good**: "We chose PostgreSQL over MongoDB because our data model has complex relationships between entities (products, orders, customers) that require ACID transactions and foreign key constraints. MongoDB's eventual consistency model would require significant application-level logic to maintain referential integrity."

❌ **Poor**: "We're using PostgreSQL because it's better."

### Maintaining ADRs

- **Don't modify accepted ADRs**: If a decision changes, create a new ADR and mark old one as superseded
- **Keep historical record**: Even superseded ADRs are valuable for understanding project history
- **Link relationships**: Reference related ADRs to show evolution of decisions

---

## ADR Workflow

```
1. Identify Decision Need
   ↓
2. Research & Evaluate Options
   ↓
3. Create ADR (Status: Proposed)
   ↓
4. Review with Lead Engineer
   ↓
5. Update Status to Accepted
   ↓
6. Implement Decision
   ↓
7. Reference ADR in Code
```

---

## Questions?

If uncertain whether a decision needs an ADR:
1. Ask: "Will someone 6 months from now wonder why we did this?"
2. Ask: "Does this affect multiple parts of the system?"
3. Ask: "Could this decision be controversial or have significant trade-offs?"

If yes to any question → Create an ADR.
