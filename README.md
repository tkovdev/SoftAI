# SoftAI - AI Agent-Driven Development Framework

**A reusable framework for building robust web applications using specialized AI agents**

---

## 🎯 Overview

SoftAI is a **framework** (not an application) that provides:
- **Agent definitions**: Specialized AI agents for different roles (backend, frontend, testing, etc.)
- **Context management**: System to keep AI context small and enable seamless handoffs
- **Architecture standards**: Proven patterns for C# .NET + Angular applications
- **Workflow templates**: Sprint planning, delegation, and collaboration patterns

**Projects using SoftAI** are in separate workspace folders (e.g., `SoftAI-Example/`).

---

## 🏗️ System Architecture

### Framework + Project Structure

```
┌─────────────────────────────────────────────────────────────┐
│                   SOFTAI FRAMEWORK                          │
│  (Reusable across all projects)                            │
│                                                             │
│  • Agent Definitions (.github/agents/)                      │
│  • Architecture Standards (docs/adr/)                       │
│  • Workflow Templates (.context/workflows/)                 │
│  • Context System (.context/)                              │
└─────────────────────────────────────────────────────────────┘
                          ↓ used by
┌─────────────────────────────────────────────────────────────┐
│              PROJECT IMPLEMENTATION                         │
│  (e.g., SoftAI-Example/, SoftAI-YourProject/)              │
│                                                             │
│  • Application Code (src/, frontend/)                       │
│  • Project Tests (tests/)                                  │
│  • Requirements (.context/requirements/)                    │
│  • Sprint Plans (.context/sprints/)                        │
│  • Work Logs (.context/WORK_LOG.md)                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 Repository Structure

### SoftAI/ - Framework Repository
```
SoftAI/                                 # Framework (this repository)
├── .context/                           # Context management templates
│   ├── PROJECT_STATE.md                # Active projects registry
│   ├── CONTEXT_INDEX.md                # Framework navigation guide
│   ├── README.md                       # Context system documentation
│   ├── workflows/                      # Agent collaboration patterns
│   │   ├── AGENT_WORKFLOW_GUIDE.md     # Complete workflow
│   │   ├── AGENT_HANDOFF_PROTOCOL.md   # Handoff procedures
│   │   └── DELEGATION_ESCALATION_FLOWS.md # Visual diagrams
│   ├── sprints/                        # Sprint templates
│   │   └── SPRINT_TEMPLATE.md
│   └── archive/                        # Framework history
│
├── .github/agents/                     # AI agent specifications
│   ├── instructor.agent.md             # Meta-agent
│   ├── lead-software-engineer.agent.md # Orchestrator
│   ├── project-analyst.agent.md        # Project state analysis specialist
│   ├── backend-developer.agent.md      # Backend specialist
│   ├── frontend-developer.agent.md     # Frontend specialist
│   ├── database-engineer.agent.md      # Database specialist
│   ├── qa-testing.agent.md             # Testing specialist
│   ├── devops-engineer.agent.md        # DevOps specialist
│   ├── security-engineer.agent.md      # Security specialist
│   ├── code-reviewer.agent.md          # Code review specialist
│   ├── api-designer.agent.md           # API design specialist
│   ├── documentation.agent.md          # Documentation specialist
│   └── product-manager.agent.md        # Product/requirements specialist
│
└── docs/
    └── adr/                            # Architecture Decision Records
        ├── README.md                   # ADR index
        ├── TEMPLATE.md                 # ADR template
        ├── 001-backend-cqrs-with-mediatr.md
        ├── 002-csharp-coding-standards.md
        ├── 003-typescript-angular-coding-standards.md
        ├── 004-error-handling-strategy.md
        ├── 005-logging-strategy.md
        ├── 006-technology-stack.md
        ├── 007-authentication-authorization-strategy.md
        ├── 008-api-design-standards.md
        ├── 009-database-migration-strategy.md
        ├── 010-testing-strategy.md
        ├── 011-cicd-pipeline-design.md
        └── 012-angular-state-management.md
```

### Project Implementations (Separate Workspaces)

Example: **SoftAI-Example/** (DVD Rental System)
```
SoftAI-Example/                         # Project implementation
├── .context/                           # Project-specific tracking
│   ├── PROJECT_STATE.md                # Project status
│   ├── WORK_LOG.md                     # Development history
│   ├── requirements/                   # Project requirements
│   │   ├── BUSINESS_REQUIREMENTS.md
│   │   ├── user-stories/
│   │   └── domain-model.md
│   └── sprints/                        # Sprint plans
│       └── sprint-0/
│           └── SPRINT_0_PLAN.md
│
├── src/                                # Application source code
│   ├── [Project].API/                  # ASP.NET Core Web API
│   ├── [Project].Core/                 # Domain entities
│   ├── [Project].Application/          # CQRS handlers
│   └── [Project].Infrastructure/       # EF Core, data access
│
├── tests/                              # Test projects
│   ├── [Project].UnitTests/
│   └── [Project].IntegrationTests/
│
└── frontend/                           # Angular applications
    ├── customer-portal/                # External portal
    └── back-office/                    # Internal portal
```

---

## 🚀 Getting Started

### For Teams Using SoftAI

**Starting a New Project?**

1. **Create Project Workspace**
   - Create new folder: `SoftAI-YourProject/`
   - Copy `.context/` structure from `SoftAI-Example/`
   - Update project-specific details

2. **Register Your Project**
   - Add to `.context/PROJECT_STATE.md` in SoftAI framework
   - Link to your project's context folder

3. **Begin Sprint 0**
   - Define requirements
   - Plan architecture using ADRs as guide
   - Set up project structure
   - Configure development environment

**See**: [SoftAI-Example](../SoftAI-Example/) for a complete working example (DVD Rental System)

### For AI Agents

**Starting Work on a Project?**

1. **Get Quick Briefing** (2 min) - *Recommended for fast onboarding*
   - Request briefing from **Project Analyst Agent**
   - Get current state, progress, blockers, and next steps
   - Receive role-specific context for your work

2. **Read Framework Registry** (2 min) - *Alternative: Manual onboarding*
   - Open [.context/PROJECT_STATE.md](.context/PROJECT_STATE.md)
   - Find your project in Active Projects list
   - Note project folder location

3. **Navigate to Project** (3 min)
   - Go to project workspace folder
   - Read project's `PROJECT_STATE.md`
   - Review current sprint plan

4. **Review Standards** (10-15 min)
   - Read ADRs relevant to your work area (see [docs/adr/README.md](docs/adr/README.md))
   - Review your agent definition in [.github/agents/](.github/agents/)
   - Check workflow templates if needed

5. **Start Working**
   - Follow project's sprint backlog
   - Update project context (not framework)
   - Use framework workflows for collaboration

**See**: [.context/README.md](.context/README.md) for complete workflow guide
   - Logging infrastructure requirements
   - Technology version constraints

3. **Review & Approve ADRs** as they're drafted
   - Each ADR explains options and trade-offs
   - Provide feedback or approval
   - Your input shapes the foundation

4. **Monitor Progress**:
   - Check [PROJECT_STATE.md](.context/PROJECT_STATE.md) for current status
   - Review [SPRINT_TRACKER.md](.context/SPRINT_TRACKER.md) for backlog and velocity
   - Participate in sprint reviews

---

### For AI Agents

**Starting a new work session:**

1. **Read** [.context/PROJECT_STATE.md](.context/PROJECT_STATE.md) (5 min)
   - Understand current project state
   - Check if there's work assigned to you
   - See recent decisions and blockers

2. **Check** [.context/CONTEXT_INDEX.md](.context/CONTEXT_INDEX.md) (2 min)
   - Find relevant documentation
   - Understand where information lives

3. **Load task-specific context** (10 min)
   - Your agent specification (`.github/agents/`)
   - Relevant ADRs (`docs/adr/`)
   - Code files you'll work with

4. **Follow the workflow** in [.context/AGENT_WORKFLOW_GUIDE.md](.context/AGENT_WORKFLOW_GUIDE.md)

**Total time to context: ~15-20 minutes**

---

### For Human Developers

**Understanding the system:**

1. Read this README (you're here!)
2. Review [.context/README.md](.context/README.md) - Context management system
3. Check [PROJECT_STATE.md](.context/PROJECT_STATE.md) - Current status
4. Review agent specifications in `.github/agents/` to understand capabilities

**Working with agents:**

1. Update [PROJECT_STATE.md](.context/PROJECT_STATE.md) with requirements
2. Specify which agent should handle the work
3. Agents will follow their workflows and update state
4. Review progress in [WORK_LOG.md](.context/WORK_LOG.md)
5. Review architectural decisions in `docs/adr/`

---

## 🤖 Agent System

### Agent Roles

**Instructor Agent** (Meta-agent)
- Creates specifications for new agents
- Ensures agent quality and consistency

**Lead Software Engineer** (Orchestrator)
- Technical lead and architect
- Delegates work to specialized agents
- Makes architectural decisions
- Coordinates team

**Project Analyst** (Analysis & Reporting)
- Analyzes current project state and progress
- Provides quick briefings for agents after context switches
- Identifies gaps between planned and actual implementation
- Generates status reports for stakeholders
- Monitors sprint progress and blockers

**Specialized Development Agents:**
- **Backend Developer**: .NET/C# with CQRS pattern
- **Frontend Developer**: Angular 21 with Signals
- **Database Engineer**: Schema design, migrations, EF Core
- **QA/Testing**: Unit, integration, E2E testing
- **DevOps Engineer**: CI/CD, containers, deployment
- **Security Engineer**: Auth, authorization, security audits
- **Code Reviewer**: Code quality and standards
- **API Designer**: OpenAPI specifications
- **Documentation**: Technical documentation

### Agent Workflow

```
1. Agent reads PROJECT_STATE.md
2. Agent loads relevant context
3. Agent performs work
4. Agent documents decisions (ADRs)
5. Agent updates PROJECT_STATE.md and WORK_LOG.md
6. Agent prepares handoff for next agent
```

See [.context/AGENT_WORKFLOW_GUIDE.md](.context/AGENT_WORKFLOW_GUIDE.md) for complete details.

---

## 📚 Context Management System

### Why Context Management?

Large projects with AI agents face challenges:
- **Context window limits**: Can't load entire codebase
- **Session discontinuity**: Hard to pick up where previous agent left off
- **Decision tracking**: Why were things done this way?
- **Progress visibility**: What's done? What's next?

### Our Solution

**Minimal Context Loading**: Agents read only what's needed (~20% of codebase)

**State Tracking**: Single source of truth in PROJECT_STATE.md

**Decision Preservation**: Architecture Decision Records (ADRs) for all significant decisions

**Smooth Handoffs**: Standardized protocol for agent transitions

**Session History**: Complete log of all work sessions

See [.context/README.md](.context/README.md) for complete documentation.

---

## 🏛️ Technical Architecture

### Backend Stack

- **Framework**: ASP.NET Core 8.0 (LTS)
- **Pattern**: CQRS with MediatR
- **ORM**: Entity Framework Core
- **Validation**: FluentValidation
- **Testing**: xUnit, Moq, FluentAssertions
- **API Docs**: Swagger/Swashbuckle

**Architecture**: Clean Architecture
```
API Layer (Controllers, Middleware)
    ↓
Application Layer (Commands, Queries, Handlers, DTOs)
    ↓
Core Layer (Domain Entities, Interfaces)
    ↑
Infrastructure Layer (EF Core, Repositories, External Services)
```

### Frontend Stack

- **Framework**: Angular 21+ (standalone components)
- **State**: Signals (`signal()`, `computed()`, `effect()`)
- **Data**: `rxResource()` for reactive data fetching
- **UI**: Angular Material (or custom components)
- **Forms**: Reactive Forms with signal integration
- **Testing**: Jasmine/Karma (or Jest), Cypress/Playwright

**Key Patterns**:
- Standalone components with OnPush change detection
- Signal-based state management (no BehaviorSubject)
- Modern control flow (`@if`, `@for`, `@switch`)
- Lazy-loaded feature modules

### DevOps

- **Containerization**: Docker with multi-stage builds
- **CI/CD**: GitHub Actions / Azure DevOps
- **Database**: SQL Server / PostgreSQL
- **Monitoring**: Serilog, Application Insights

---

## 📖 Documentation

### Architecture Decision Records (ADRs)

All significant architectural and design decisions are documented as ADRs in `docs/adr/`.

**Current ADRs:**
- [001-backend-cqrs-with-mediatr.md](docs/adr/001-backend-cqrs-with-mediatr.md) - CQRS pattern decision (example)

**Creating ADRs:**
See [docs/adr/README.md](docs/adr/README.md) for guidelines.

### API Documentation

OpenAPI specifications will be maintained in `docs/api/` (to be created as APIs are designed).

### Developer Guides

Setup and how-to guides will be in `docs/guides/` (to be created as needed).

---

## 🔄 Development Workflow

### Typical Feature Development Flow

```
1. User/Stakeholder defines requirement
   ↓
2. Lead Engineer designs architecture (creates ADR if significant)
   ↓
3. API Designer creates OpenAPI spec
   ↓
4. Backend Developer & Frontend Developer work in parallel:
   - Backend: CQRS commands/queries/handlers
   - Frontend: Components, services, UI
   ↓
5. Database Engineer creates/updates schema (if needed)
   ↓
6. QA/Testing Agent writes tests:
   - Unit tests for handlers
   - Integration tests for APIs
   - E2E tests for user flows
   ↓
7. Code Reviewer reviews all changes
   ↓
8. Security Engineer audits security aspects
   ↓
9. DevOps Engineer deploys to environment
   ↓
10. Documentation Agent updates documentation
```

All steps coordinated via PROJECT_STATE.md with clear handoffs between agents.

---

## 🎯 Key Principles

### Code Quality
- **SOLID principles**: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
- **DRY**: Don't Repeat Yourself
- **Clean Code**: Readable, maintainable, well-tested
- **Type Safety**: Leverage strong typing (C#, TypeScript)

### Architecture
- **Clean Architecture**: Separation of concerns, dependency rule
- **CQRS**: Commands for writes, Queries for reads
- **Testability**: All business logic easily testable
- **Security**: Built-in from the start, not bolted on

### Process
- **Documentation**: All decisions documented (ADRs)
- **Testing**: Test-driven or test-alongside development
- **Code Review**: All code reviewed before merge
- **Continuous Integration**: Automated testing and deployment

---

## 📊 Project Status

See [.context/PROJECT_STATE.md](.context/PROJECT_STATE.md) for current status.

**Current Phase**: Planning  
**Last Updated**: 2026-02-10

---

## 🤝 Contributing

### For AI Agents
Follow the workflow in [.context/AGENT_WORKFLOW_GUIDE.md](.context/AGENT_WORKFLOW_GUIDE.md).

### For Humans
1. Review current state in PROJECT_STATE.md
2. Update requirements/specifications  
3. Coordinate with Lead Software Engineer agent
4. Review agent outputs and provide feedback

---

## 📞 Support

### Agent System Issues
- Review [.context/AGENT_WORKFLOW_GUIDE.md](.context/AGENT_WORKFLOW_GUIDE.md)
- Check [.context/AGENT_HANDOFF_PROTOCOL.md](.context/AGENT_HANDOFF_PROTOCOL.md)
- Escalate to Lead Software Engineer agent

### Context Management Issues
- Review [.context/README.md](.context/README.md)
- Check if PROJECT_STATE.md is up to date
- Verify all agents following handoff protocol

### Technical Issues
- Check relevant ADRs in `docs/adr/`
- Review agent specifications in `.github/agents/`
- Search WORK_LOG.md for similar issues

---

## 🎓 Learning Resources

1. **Context Management**: [.context/README.md](.context/README.md)
2. **Agent Workflow**: [.context/AGENT_WORKFLOW_GUIDE.md](.context/AGENT_WORKFLOW_GUIDE.md)
3. **Handoff Protocol**: [.context/AGENT_HANDOFF_PROTOCOL.md](.context/AGENT_HANDOFF_PROTOCOL.md)
4. **Architecture Decisions**: [docs/adr/README.md](docs/adr/README.md)
5. **Agent Capabilities**: Individual files in `.github/agents/`

---

## 📈 Success Metrics

### Context Efficiency
- Agents load < 20% of codebase context per session
- Time to context < 20 minutes
- Context relevance > 80%

### Development Velocity
- Features delivered with consistent quality
- Minimal rework due to miscommunication
- Smooth agent transitions

### Quality
- Test coverage > 80% on business logic
- All architectural decisions documented
- No critical security vulnerabilities

---

## 🔮 Future Enhancements

As the project grows, consider:
- Event sourcing (CQRS enables this)
- Separate read/write data stores
- Microservices architecture
- Domain-Driven Design patterns
- Real-time features with SignalR

All significant changes should have ADRs documenting the decision.

---

## 📄 License

[To be determined]

---

## 🙏 Acknowledgments

This system combines:
- Clean Architecture principles (Robert C. Martin)
- CQRS pattern (Greg Young, Udi Dahan)  
- Context management for AI agents
- Modern web development best practices

---

**Remember**: The goal is efficient, coordinated, high-quality development. If something doesn't serve that goal, improve it!

---

Last Updated: 2026-02-10
