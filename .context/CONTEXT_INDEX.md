# Context Index - SoftAI Framework

**Quick reference guide for the SoftAI framework and active projects.**

---

## ⚠️ IMPORTANT: Framework vs Project

### This is the Framework Folder (SoftAI)
Contains:
- Agent definitions (`.github/agents/`)
- Architecture standards (`docs/adr/`)
- Workflow templates (`.context/workflows/`)
- Sprint templates (`.context/sprints/`)

### Active Project Locations
**Actual project work** happens in separate workspace folders:

| Project | Location | Status |
|---------|----------|--------|
| **DVD Rental System** | `../SoftAI-Example/` | 🟢 Active |

---

## 🚀 Quick Start (Read This First)

### New Agent Starting Work?

1. **Identify Your Project**: Check [PROJECT_STATE.md](./PROJECT_STATE.md) for active projects
2. **Navigate to Project Folder**: Go to `../SoftAI-Example/` (or relevant project)
3. **Read Project Context** (5 min):
   - Read `PROJECT_STATE.md` in the project folder
   - Read current sprint plan
   - Review relevant requirements
4. **Review Framework Standards** (10-15 min):
   - Read ADRs relevant to your work area below
   - Review your agent definition in `.github/agents/`
5. **Start Working**: Follow the project's sprint plan

### Estimated Time to Context
- **Project State**: 5 minutes (project's PROJECT_STATE.md)
- **Your Domain**: 10-15 minutes (relevant ADRs + agent definition)
- **Full Context**: Not needed! This system keeps context minimal.

---

## 📁 Framework Documentation Map

### Context Management (.context/)
- **[PROJECT_STATE.md](./PROJECT_STATE.md)** - ⭐ START HERE - Active projects registry
- **[CONTEXT_INDEX.md](./CONTEXT_INDEX.md)** - This file
- **[workflows/AGENT_WORKFLOW_GUIDE.md](./workflows/AGENT_WORKFLOW_GUIDE.md)** - Complete agent workflow
- **[workflows/AGENT_HANDOFF_PROTOCOL.md](./workflows/AGENT_HANDOFF_PROTOCOL.md)** - How to hand off work
- **[workflows/DELEGATION_ESCALATION_FLOWS.md](./workflows/DELEGATION_ESCALATION_FLOWS.md)** - Visual collaboration diagrams
- **[sprints/SPRINT_TEMPLATE.md](./sprints/SPRINT_TEMPLATE.md)** - Template for sprint planning
- **[archive/](./archive/)** - Historical framework development records

### Architecture Decisions (docs/adr/)
- **[README.md](../docs/adr/README.md)** - ADR index and how to use
- **[TEMPLATE.md](../docs/adr/TEMPLATE.md)** - ADR template
- **Active ADRs**: 12 architectural decisions covering all aspects

### Agent Instructions (.github/agents/)
- **[instructor.agent.md](../.github/agents/instructor.agent.md)** - Meta-agent for creating new agents
- **[lead-software-engineer.agent.md](../.github/agents/lead-software-engineer.agent.md)** - Lead engineer
- **Team Agents**: Backend, Frontend, Database, QA, DevOps, Security, API Designer, Documentation, Product Manager, Code Reviewer

---

## 🎯 Find Information By Task

### "I need to see current project status"
→ Read: [PROJECT_STATE.md](./PROJECT_STATE.md) for active projects list
→ Then: Navigate to project folder and read its `PROJECT_STATE.md`

**For DVD Rental System**: See [../SoftAI-Example/.context/PROJECT_STATE.md](../SoftAI-Example/.context/PROJECT_STATE.md)

### "I need to see current sprint and backlog"
→ Navigate to project folder: `../SoftAI-Example/`
→ Read: `.context/sprints/sprint-X/SPRINT_X_PLAN.md`

**For DVD Rental System**: [../SoftAI-Example/.context/sprints/sprint-0/SPRINT_0_PLAN.md](../SoftAI-Example/.context/sprints/sprint-0/SPRINT_0_PLAN.md)

### "I need to pick up a task from the sprint"
→ Navigate to project: `../SoftAI-Example/`
→ Read: `.context/sprints/sprint-X/SPRINT_X_PLAN.md`
→ Update task status in project's sprint plan
→ Log work in project's `WORK_LOG.md`

### "I need to delegate to another agent"
→ Read: [workflows/AGENT_WORKFLOW_GUIDE.md - Agent-to-Agent Delegation](./workflows/AGENT_WORKFLOW_GUIDE.md)
→ Visual: [workflows/DELEGATION_ESCALATION_FLOWS.md](./workflows/DELEGATION_ESCALATION_FLOWS.md)
→ Document delegation in project's sprint plan or work log

### "I disagree with another agent"
→ Read: [workflows/AGENT_WORKFLOW_GUIDE.md - Conflict Resolution](./workflows/AGENT_WORKFLOW_GUIDE.md)
→ Visual: [Escalation Flow Diagram](./workflows/DELEGATION_ESCALATION_FLOWS.md)
→ Follow escalation process, document in project context

### "I need to escalate a decision"
→ Read: [Decision-Making Authority Levels](./workflows/AGENT_WORKFLOW_GUIDE.md)
→ Visual: [Decision Authority Matrix](./workflows/DELEGATION_ESCALATION_FLOWS.md)
→ Escalate to Lead Software Engineer

### "I need to see what's been completed"
→ Navigate to project: `../SoftAI-Example/`
→ Read: `.context/WORK_LOG.md`

### "I need to understand an architectural decision"
### "I need to implement a backend feature"
→ Read:
1. [../.github/agents/backend-developer.agent.md](../.github/agents/backend-developer.agent.md) - Your role
2. Relevant ADRs: 001 (CQRS), 002 (C# Standards), 004 (Error Handling), 008 (API Design)
3. Navigate to project code in `../SoftAI-Example/src/`

### "I need to implement a frontend feature"
→ Read:
1. [../.github/agents/frontend-developer.agent.md](../.github/agents/frontend-developer.agent.md) - Your role
2. Relevant ADRs: 003 (TypeScript Standards), 012 (State Management), 008 (API Design)
3. Navigate to project code in `../SoftAI-Example/frontend/`

### "I need to design database schema"
→ Read:
1. [../.github/agents/database-engineer.agent.md](../.github/agents/database-engineer.agent.md) - Your role
2. Relevant ADRs: 006 (Technology Stack), 009 (Database Migrations)
3. Review project's domain model in requirements

### "I need to write tests"
→ Read:
1. [../.github/agents/qa-testing.agent.md](../.github/agents/qa-testing.agent.md) - Your role
2. Relevant ADR: 010 (Testing Strategy)
3. Navigate to project tests in `../SoftAI-Example/tests/`

### "I need to set up DevOps/CI/CD"
→ Read:
1. [../.github/agents/devops-engineer.agent.md](../.github/agents/devops-engineer.agent.md) - Your role
2. Relevant ADR: 011 (CI/CD Pipeline Design)
3. Navigate to project CI/CD configs in `../SoftAI-Example/.github/workflows/`

### "I need to implement security features"
→ Read:
1. [../.github/agents/security-engineer.agent.md](../.github/agents/security-engineer.agent.md) - Your role
2. Relevant ADR: 007 (Authentication & Authorization Strategy)
3. Navigate to project security code

### "I need to review code"
→ Read:
1. [../.github/agents/code-reviewer.agent.md](../.github/agents/code-reviewer.agent.md) - Your role
2. Relevant ADRs: 002 (C# Standards), 003 (TypeScript Standards)
3. The specific files under review in project folder

### "I need to design an API"
→ Read:
1. [../.github/agents/api-designer.agent.md](../.github/agents/api-designer.agent.md) - Your role
2. Relevant ADR: 008 (API Design Standards)
3. Review project requirements and domain model

### "I need to write documentation"
→ Read:
1. [../.github/agents/documentation.agent.md](../.github/agents/documentation.agent.md) - Your role
2. Relevant standards from ADRs
3. Review existing project documentation for consistency

---

## 👥 Agent Reference

### Orchestration
- **Instructor Agent**: Creates new agent specifications
- **Lead Software Engineer**: Technical lead, delegates to specialized agents

### Development
- **Backend Developer**: Implements .NET/C# with CQRS pattern
- **Frontend Developer**: Implements Angular 21 with Signals
- **Database Engineer**: Designs schemas, writes migrations

### Quality & Operations
- **QA/Testing Agent**: Writes tests at all levels
- **DevOps Engineer**: CI/CD, containerization, deployment
- **Security Engineer**: Authentication, authorization, security audits

### Support
- **Code Reviewer**: Reviews code quality and standards
- **API Designer**: Creates OpenAPI specifications
- **Documentation Agent**: Writes technical documentation

---

## 🗂️ Standard Project Structure

Projects using SoftAI follow this structure:

### Backend Structure (C# .NET)
```
src/
├── [Project].API/           # Web API, controllers, startup
├── [Project].Core/          # Domain entities, interfaces
├── [Project].Application/   # CQRS handlers, DTOs, validators
├── [Project].Infrastructure/# EF Core, repositories, external services
tests/
├── [Project].UnitTests/    # Unit tests for handlers
└── [Project].IntegrationTests/ # API integration tests
```

### Frontend Structure (Angular)
```
frontend/
├── src/
│   ├── app/
│   │   ├── features/       # Feature modules (lazy loaded)
│   │   ├── core/           # Singletons (services, guards, interceptors)
│   │   ├── shared/         # Reusable components, pipes, directives
│   │   └── models/         # TypeScript interfaces and types
```

### Project Context Structure
```
.context/
├── PROJECT_STATE.md        # Current project state
├── WORK_LOG.md             # Development session history
├── README.md               # Context folder guide
├── requirements/           # All requirements
│   ├── BUSINESS_REQUIREMENTS.md
│   ├── user-stories/
│   └── domain-model.md
└── sprints/                # Sprint planning
    └── sprint-X/
        └── SPRINT_X_PLAN.md
```

---

## 🔍 Search Strategies

### When to Use Different Search Methods

**Use semantic_search when:**
- Looking for concepts or functionality: "authentication logic", "product validation"
- Finding examples of patterns: "CQRS command handler", "Angular signal service"
- Exploring unfamiliar codebase

**Use grep_search when:**
- Looking for specific strings: class names, method names, error messages
- Finding all usages of something: "IProductRepository", "AuthService"
- Searching within specific file patterns: "*.cs", "*.component.ts"

**Use file_search when:**
- You know the filename pattern: "*Product*.cs", "*.spec.ts"
- Finding files of specific types

**Use list_code_usages when:**
- Understanding how a class/method is used throughout codebase
- Refactoring and need to update all usages

---

## 📊 Key Patterns & Conventions

### Backend (CQRS with MediatR)
- **Commands**: Write operations, return `Result<T>`, validated with FluentValidation
- **Queries**: Read operations, return DTOs, use `.AsNoTracking()`
- **Handlers**: Implement `IRequestHandler<TRequest, TResponse>`
- **Controllers**: Thin, delegate to MediatR

### Frontend (Angular 21 with Signals)
- **State**: Use `signal()`, `computed()`, `effect()` (not BehaviorSubject)
- **Data Fetching**: Use `rxResource()` for reactive data loading
- **Components**: Standalone components with OnPush change detection
- **Forms**: Reactive forms with signal integration
- **Control Flow**: `@if`, `@for`, `@switch` (not *ngIf, *ngFor)

### Testing
- **Unit Tests**: Test handlers in isolation with mocked dependencies
- **Integration Tests**: Test full API endpoints with in-memory database
- **E2E Tests**: Test critical user flows

### Architecture
- **Clean Architecture**: Core (domain/application) → Infrastructure → API
- **Dependency Rule**: Dependencies point inward only
- **Separation**: Business logic separate from infrastructure

---

## 💡 Context Window Tips

### Minimize Context
1. **Don't read everything**: Use this index to find only what you need
2. **Link, don't copy**: Reference files instead of duplicating content
3. **Summarize**: Provide brief summaries with links to details
4. **Stay focused**: Only load context for your current task

### Agent-Specific Context Needs

**Lead Engineer**: Broad overview, delegate details to specialists
**Backend Developer**: Backend code + database schema high-level
**Frontend Developer**: Frontend code + API contracts
**Database Engineer**: Schema files + related migrations
**QA/Testing**: Code being tested + test examples
**DevOps**: Infrastructure configs + deployment requirements
**Security**: Auth code + security-related ADRs
**Code Reviewer**: Specific files under review + standards
**API Designer**: OpenAPI specs + business requirements
**Documentation**: Code to document + existing docs

---

## 🔄 Keeping This File Updated

Update CONTEXT_INDEX.md when:
- Adding new major documentation files
- Creating new directory structures
- Establishing new patterns or conventions
- Adding new types of searchable information

**Don't document**:
- Every individual code file (too much noise)
- Temporary files or notes
- Implementation details (those go in ADRs)

---

## 📞 Getting Unstuck

Can't find what you need?

1. **Check PROJECT_STATE.md**: Is the feature even implemented yet?
2. **Check WORK_LOG.md**: Maybe it was worked on recently?
3. **Search semantically**: Use semantic_search with concept description
4. **Ask Lead Engineer**: If truly stuck, escalate to Lead Software Engineer
5. **Document the gap**: If information should exist but doesn't, note it

---

Last Updated: 2026-02-10
