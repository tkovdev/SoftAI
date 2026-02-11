# SoftAI Agent Team

This directory contains specifications for specialized AI agents that work together to build robust .NET + Angular web applications.

## Agent Team Structure

### Business Layer

#### Product Manager Agent
**Role**: Requirements gathering and business analysis  
**File**: [product-manager.agent.md](product-manager.agent.md)  
**Responsibilities**:
- Gather requirements from stakeholders
- Ask clarifying questions to uncover hidden requirements
- Document business requirements and user stories
- Work with Lead Engineer to prioritize work
- Manage scope and stakeholder communication
- Hand off comprehensive requirements package to technical team

**Works With**: Stakeholders → Lead Software Engineer

---

### Technical Leadership

#### Lead Software Engineer Agent
**Role**: Technical lead and primary architect  
**File**: [lead-software-engineer.agent.md](lead-software-engineer.agent.md)  
**Responsibilities**:
- Design system architecture following SOLID and clean architecture principles
- Make technical decisions about frameworks and patterns
- Coordinate and delegate work to specialized development agents
- Review code and ensure quality standards
- Create architecture documentation and ADRs
- Provide technical feedback to Product Manager

**Works With**: Product Manager ↔ All Development Agents

---

### Development Specialists

#### Backend Developer Agent
**Role**: .NET/C# API and business logic implementation  
**File**: [backend-developer.agent.md](backend-developer.agent.md)  
**Responsibilities**:
- Implement RESTful APIs following OpenAPI specifications
- Create Commands (write operations) and Queries (read operations) using CQRS with MediatR
- Implement business logic in command/query handlers
- Work with Entity Framework Core for data access
- Implement FluentValidation for input validation
- Write unit tests for all handlers

**Works With**: Lead Engineer → Database Engineer, API Designer, QA/Testing Agent

#### Frontend Developer Agent
**Role**: Angular UI/UX implementation  
**File**: [frontend-developer.agent.md](frontend-developer.agent.md)  
**Responsibilities**:
- Implement Angular standalone components
- Create services and state management using Signals
- Implement routing with functional guards
- Build responsive UIs with Angular Material
- Implement forms with reactive patterns
- Write unit and E2E tests

**Works With**: Lead Engineer → API Designer, QA/Testing Agent

#### Database Engineer Agent
**Role**: Database design and optimization  
**File**: [database-engineer.agent.md](database-engineer.agent.md)  
**Responsibilities**:
- Design database schemas with proper normalization
- Create Entity Framework Core entities and migrations
- Optimize queries and create indexes
- Ensure data integrity with constraints
- Design for scalability and performance

**Works With**: Lead Engineer → Backend Developer Agent

---

### Quality & Operations

#### QA/Testing Agent
**Role**: Comprehensive testing strategy and implementation  
**File**: [qa-testing.agent.md](qa-testing.agent.md)  
**Responsibilities**:
- Create unit tests (xUnit/Jasmine)
- Write integration tests for APIs and database
- Implement E2E tests (Cypress/Playwright)
- Maintain test suites and ensure coverage
- Validate features against acceptance criteria

**Works With**: Backend Developer, Frontend Developer, Lead Engineer

#### DevOps Engineer Agent
**Role**: CI/CD, infrastructure, and deployment  
**File**: [devops-engineer.agent.md](devops-engineer.agent.md)  
**Responsibilities**:
- Configure CI/CD pipelines (GitHub Actions, Azure DevOps)
- Create Docker containers with multi-stage builds
- Set up development, staging, and production environments
- Configure monitoring and logging
- Automate deployment processes

**Works With**: Lead Engineer, All Development Agents

#### Security Engineer Agent
**Role**: Security implementation and auditing  
**File**: [security-engineer.agent.md](security-engineer.agent.md)  
**Responsibilities**:
- Implement authentication (JWT, Identity Server)
- Configure authorization (role-based, claims-based)
- Perform security audits and vulnerability scanning
- Review code for security issues
- Ensure compliance with security standards

**Works With**: Lead Engineer, Backend Developer, Frontend Developer

---

### Support Roles

#### API Designer Agent
**Role**: API contract design and documentation  
**File**: [api-designer.agent.md](api-designer.agent.md)  
**Responsibilities**:
- Create OpenAPI (Swagger) specifications
- Design RESTful endpoints following best practices
- Define request/response models
- Ensure API consistency and versioning
- Document API contracts

**Works With**: Lead Engineer → Backend Developer, Frontend Developer

#### Code Reviewer Agent
**Role**: Detailed code review for quality and patterns  
**File**: [code-reviewer.agent.md](code-reviewer.agent.md)  
**Responsibilities**:
- Review code for adherence to coding standards
- Check for proper design patterns usage
- Identify code smells and anti-patterns
- Provide constructive feedback
- Ensure consistency across codebase

**Works With**: All Development Agents → Lead Engineer

#### Documentation Agent
**Role**: Technical documentation and guides  
**File**: [documentation.agent.md](documentation.agent.md)  
**Responsibilities**:
- Write API documentation
- Create developer onboarding guides
- Document setup and deployment procedures
- Write code comments and XML docs
- Maintain README files

**Works With**: All Agents → Lead Engineer

---

### Meta Roles

#### Instructor Agent
**Role**: Create new specialized agent specifications  
**File**: [instructor.agent.md](instructor.agent.md)  
**Responsibilities**:
- Design new agent specifications when needed capabilities aren't covered
- Define agent purpose, responsibilities, and workflows
- Ensure new agents integrate with existing team structure
- Document agent interaction patterns

**Works With**: Lead Engineer

---

## Workflow Overview

### Typical Project Flow

```
1. Product Manager
   ↓ (requirements package)
2. Lead Software Engineer
   ↓ (architecture + task delegation)
3. Specialized Agents (parallel work)
   - API Designer → Backend Developer
   - Database Engineer → Backend Developer
   - Frontend Developer
   - DevOps Engineer
   ↓ (implementations)
4. QA/Testing Agent (testing)
   ↓
5. Code Reviewer Agent (review)
   ↓
6. Lead Engineer (final review & integration)
   ↓
7. DevOps Engineer (deployment)
   ↓
8. Documentation Agent (final docs)
```

### Agent Collaboration Patterns

#### New Feature Development
1. **Product Manager** documents user story with acceptance criteria
2. **Lead Engineer** designs technical approach
3. **API Designer** creates API specification
4. **Database Engineer** designs schema changes
5. **Backend Developer** implements API using CQRS
6. **Frontend Developer** implements UI components
7. **QA/Testing Agent** creates tests
8. **Security Engineer** reviews for vulnerabilities
9. **Code Reviewer** provides detailed feedback
10. **Lead Engineer** integrates and validates
11. **Documentation Agent** updates documentation

#### Bug Fix
1. **QA/Testing Agent** reproduces and documents issue
2. **Lead Engineer** diagnoses and assigns to appropriate agent
3. **Specialist Agent** (Backend/Frontend/Database) fixes issue
4. **QA/Testing Agent** validates fix
5. **Code Reviewer** reviews changes
6. **Lead Engineer** approves

#### Performance Optimization
1. **Lead Engineer** identifies performance issue
2. **Database Engineer** optimizes queries and indexes
3. **Backend Developer** improves API efficiency
4. **Frontend Developer** implements caching/lazy loading
5. **QA/Testing Agent** validates performance improvements
6. **DevOps Engineer** monitors production metrics

---

## How to Use These Agents

### For New Projects
1. Start with **Product Manager Agent** to gather and document requirements
2. Hand off to **Lead Software Engineer Agent** for technical planning
3. Lead Engineer delegates to specialized agents based on tasks
4. Agents collaborate following the workflow patterns above

### For Existing Projects
1. **Lead Engineer** assesses current state and needs
2. Delegates specific tasks to appropriate specialist agents
3. **Code Reviewer** ensures changes align with existing codebase
4. **QA/Testing Agent** validates changes don't break existing functionality

### For Specific Tasks
Choose the appropriate agent directly:
- Need API design? → **API Designer Agent**
- Need database schema? → **Database Engineer Agent**
- Need CI/CD setup? → **DevOps Engineer Agent**
- Need security review? → **Security Engineer Agent**
- Need documentation? → **Documentation Agent**

### Creating New Agents
When you need a capability not covered by existing agents:
1. Work with **Instructor Agent** to create new agent specification
2. **Lead Engineer** reviews and approves new agent
3. New agent is added to the team structure

---

## Technology Stack

### Backend
- **Framework**: ASP.NET Core (latest LTS)
- **Pattern**: CQRS with MediatR
- **ORM**: Entity Framework Core
- **Testing**: xUnit, Moq, FluentAssertions
- **Validation**: FluentValidation

### Frontend
- **Framework**: Angular 21+ (standalone components)
- **State**: Signals, computed, effect, NgRx Signal Store
- **UI**: Angular Material
- **Testing**: Jasmine/Karma, Cypress/Playwright

### DevOps
- **Containers**: Docker
- **CI/CD**: GitHub Actions, Azure DevOps
- **Databases**: SQL Server, PostgreSQL

---

## Documentation

### Additional Resources
- [Architecture Decision Records (ADRs)](../../docs/adr/)
- [Technology Stack Details](../../docs/adr/006-technology-stack.md)
- [Coding Standards - C#](../../docs/adr/002-csharp-coding-standards.md)
- [Coding Standards - TypeScript/Angular](../../docs/adr/003-typescript-angular-coding-standards.md)
- [Error Handling Strategy](../../docs/adr/004-error-handling-strategy.md)
- [Logging Strategy](../../docs/adr/005-logging-strategy.md)

---

## Agent Principles

### Clear Separation of Concerns
Each agent has a specific focus area and doesn't overlap with others.

### Collaboration Over Silos
Agents work together, coordinated by the Lead Engineer.

### Documentation First
Requirements, designs, and decisions are documented before implementation.

### Quality Built-In
Testing, security, and code review are integral to the development process.

### Business-Technology Alignment
Product Manager ensures technical solutions serve business needs.

### Continuous Improvement
Agents can be created, modified, or retired as needs evolve.

---

**Version**: 1.0  
**Last Updated**: 2026-02-10  
**Maintained By**: Lead Software Engineer
