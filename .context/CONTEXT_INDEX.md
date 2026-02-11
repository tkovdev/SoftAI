# Context Index

**Quick reference guide to find information fast and minimize context window usage.**

---

## 🚀 Quick Start (Read This First)

### New Agent Starting Work?

1. Read [PROJECT_STATE.md](.context/PROJECT_STATE.md) (5 min) - Current state
2. Skim this file for relevant documentation (2 min)
3. Read only the sections below that relate to your current task
4. Use semantic search if you need to find specific code

### Estimated Time to Context
- **Current State**: 5 minutes (PROJECT_STATE.md)
- **Your Domain**: 10-15 minutes (read relevant sections below)
- **Full Context**: Not needed! That's the point of this system.

---

## 📁 Documentation Map

### Context Management (.context/)
- **[PROJECT_STATE.md](.context/PROJECT_STATE.md)** - ⭐ START HERE - Current project state, sprint, delegations
- **[SPRINT_TRACKER.md](.context/SPRINT_TRACKER.md)** - 🏃 Sprint backlog, progress, and metrics
- **[WORK_LOG.md](.context/WORK_LOG.md)** - Session history, what's been done
- **[AGENT_HANDOFF_PROTOCOL.md](.context/AGENT_HANDOFF_PROTOCOL.md)** - How to hand off work between agents
- **[AGENT_WORKFLOW_GUIDE.md](.context/AGENT_WORKFLOW_GUIDE.md)** - Complete workflow including delegation and escalation
- **[DELEGATION_ESCALATION_FLOWS.md](.context/DELEGATION_ESCALATION_FLOWS.md)** - 📊 Visual diagrams of collaboration patterns
- **[CONTEXT_INDEX.md](.context/CONTEXT_INDEX.md)** - This file

### Architecture Decisions (docs/adr/)
- **[README.md](docs/adr/README.md)** - How to create and use ADRs
- **[TEMPLATE.md](docs/adr/TEMPLATE.md)** - ADR template
- **Active ADRs**: See docs/adr/README.md for index

### Agent Instructions (.github/agents/)
- **[instructor.agent.md](.github/agents/instructor.agent.md)** - Meta-agent for creating new agents
- **[lead-software-engineer.agent.md](.github/agents/lead-software-engineer.agent.md)** - Lead engineer orchestrating work
- **Team Agents**: See Agent Reference section below

---

## 🎯 Find Information By Task

### "I need to understand current status"
→ Read: [PROJECT_STATE.md](.context/PROJECT_STATE.md)

### "I need to see current sprint and backlog"
→ Read: [SPRINT_TRACKER.md](.context/SPRINT_TRACKER.md)
→ Quick view: PROJECT_STATE.md → Current Sprint section

### "I need to pick up a task from the sprint"
→ Read: [SPRINT_TRACKER.md - Sprint 0 Backlog](.context/SPRINT_TRACKER.md#-sprint-0-backlog)
→ Update task status: ⬜ → 🟡
→ Update: PROJECT_STATE.md → Active Work

### "I need to delegate to another agent"
→ Read: [AGENT_WORKFLOW_GUIDE.md - Agent-to-Agent Delegation](.context/AGENT_WORKFLOW_GUIDE.md#-agent-to-agent-delegation)  
→ Visual: [DELEGATION_ESCALATION_FLOWS.md](.context/DELEGATION_ESCALATION_FLOWS.md)  
→ Update: PROJECT_STATE.md → Pending Delegations section

### "I disagree with another agent"
→ Read: [AGENT_WORKFLOW_GUIDE.md - Conflict Resolution](.context/AGENT_WORKFLOW_GUIDE.md#️-conflict-resolution--escalation)  
→ Visual: [Escalation Flow Diagram](.context/DELEGATION_ESCALATION_FLOWS.md#️-escalation-flow-conflict-resolution)  
→ Update: PROJECT_STATE.md → Active Discussions section

### "I need to escalate a decision"
→ Read: [Decision-Making Authority Levels](.context/AGENT_WORKFLOW_GUIDE.md#decision-making-authority-levels)  
→ Visual: [Decision Authority Matrix](.context/DELEGATION_ESCALATION_FLOWS.md#-decision-authority-matrix)  
→ Update: PROJECT_STATE.md → Blockers section with escalation format

### "I need to see what's been completed"
→ Read: [WORK_LOG.md](.context/WORK_LOG.md)

### "I need to understand an architectural decision"
→ Read: Relevant ADR in [`docs/adr/`](docs/adr/)

### "I need to implement a backend feature"
→ Read: 
1. [backend-developer.agent.md](.github/agents/backend-developer.agent.md) - Your guidelines
2. Relevant ADRs (architecture, backend, database)
3. Existing code in `src/[Project].API/` and `src/[Project].Application/`

### "I need to implement a frontend feature"
→ Read:
1. [frontend-developer.agent.md](.github/agents/frontend-developer.agent.md) - Your guidelines
2. Relevant ADRs (frontend, API design)
3. Existing code in frontend workspace

### "I need to design database schema"
→ Read:
1. [database-engineer.agent.md](.github/agents/database-engineer.agent.md) - Your guidelines
2. Database ADRs
3. Existing migrations and entity models

### "I need to write tests"
→ Read:
1. [qa-testing.agent.md](.github/agents/qa-testing.agent.md) - Your guidelines
2. Testing ADRs
3. Existing test files as examples

### "I need to set up DevOps/CI/CD"
→ Read:
1. [devops-engineer.agent.md](.github/agents/devops-engineer.agent.md) - Your guidelines
2. DevOps ADRs
3. Existing CI/CD configs

### "I need to implement security features"
→ Read:
1. [security-engineer.agent.md](.github/agents/security-engineer.agent.md) - Your guidelines
2. Security ADRs
3. Existing authentication/authorization code

### "I need to review code"
→ Read:
1. [code-reviewer.agent.md](.github/agents/code-reviewer.agent.md) - Your guidelines
2. Coding standards ADRs
3. The specific files under review

### "I need to design an API"
→ Read:
1. [api-designer.agent.md](.github/agents/api-designer.agent.md) - Your guidelines
2. API design ADRs
3. Existing OpenAPI specs

### "I need to write documentation"
→ Read:
1. [documentation.agent.md](.github/agents/documentation.agent.md) - Your guidelines
2. Documentation standards ADRs
3. Existing documentation for style consistency

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

## 🗂️ Project Structure Reference

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
src/
├── app/
│   ├── features/           # Feature modules (lazy loaded)
│   ├── core/               # Singletons (services, guards, interceptors)
│   ├── shared/             # Reusable components, pipes, directives
│   └── models/             # TypeScript interfaces and types
```

### Documentation Structure
```
docs/
├── adr/                    # Architecture Decision Records
├── api/                    # API documentation (OpenAPI specs)
└── guides/                 # Developer guides, setup instructions
```

### Context Structure
```
.context/
├── PROJECT_STATE.md        # Current project state
├── WORK_LOG.md             # Session history
├── AGENT_HANDOFF_PROTOCOL.md # Handoff procedures
└── CONTEXT_INDEX.md        # This file
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
