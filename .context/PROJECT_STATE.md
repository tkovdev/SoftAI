# SoftAI Framework - Active Projects Registry

**Purpose**: This file tracks active projects using the SoftAI framework  
**Last Updated**: 2026-02-10  
**Framework Version**: 1.0

---

## ⚠️ IMPORTANT: This is the Framework, Not a Project

**SoftAI** is a reusable framework containing:
- Agent definitions (`.github/agents/`)
- Architecture Decision Records (ADRs) (`docs/adr/`)
- Workflow templates (`.context/workflows/`)
- Sprint templates (`.context/sprints/`)

**Actual project implementation** happens in separate workspace folders.

---

## 🎯 Active Projects

### DVD Rental System
**Location**: `../SoftAI-Example/`  
**Status**: 🟢 Active Development  
**Phase**: Sprint 0 - Requirements & Foundation  
**Started**: 2026-02-10

**Quick Links**:
- **Project Status**: [../SoftAI-Example/.context/PROJECT_STATE.md](../SoftAI-Example/.context/PROJECT_STATE.md)
- **Requirements**: [../SoftAI-Example/.context/requirements/BUSINESS_REQUIREMENTS.md](../SoftAI-Example/.context/requirements/BUSINESS_REQUIREMENTS.md)
- **Current Sprint**: [../SoftAI-Example/.context/sprints/sprint-0/SPRINT_0_PLAN.md](../SoftAI-Example/.context/sprints/sprint-0/SPRINT_0_PLAN.md)
- **Work Log**: [../SoftAI-Example/.context/WORK_LOG.md](../SoftAI-Example/.context/WORK_LOG.md)

**Description**: Dual-system DVD rental platform with internal back-office management and external customer portal.

**Technology Stack**:
- Backend: .NET 10 with CQRS (MediatR)
- Frontend: Angular 21 with Signals & PrimeNG
- Database: SQL Server 2022

---

## 📚 Framework Resources

### For All Agents
- **Agent Definitions**: `./.github/agents/` - Your role and responsibilities
- **Architecture Standards**: `./docs/adr/` - All architectural decisions
- **Workflow Guides**: `./.context/workflows/` - Collaboration patterns

### Key Framework Documents
- **[CONTEXT_INDEX.md](./.context/CONTEXT_INDEX.md)** - Quick reference guide
- **[AGENT_WORKFLOW_GUIDE.md](./.context/workflows/AGENT_WORKFLOW_GUIDE.md)** - Complete workflow
- **[AGENT_HANDOFF_PROTOCOL.md](./.context/workflows/AGENT_HANDOFF_PROTOCOL.md)** - Handoff procedures
- **[DELEGATION_ESCALATION_FLOWS.md](./.context/workflows/DELEGATION_ESCALATION_FLOWS.md)** - Visual diagrams

---

## 🚀 Starting Work?

### 1. Identify Your Project
Check the **Active Projects** section above for your project location.

### 2. Read Project Context
Navigate to the project folder and read:
1. `PROJECT_STATE.md` - Current project status
2. Current sprint plan in `sprints/sprint-X/`
3. Relevant requirements documents

### 3. Review Framework Standards
Read ADRs relevant to your work area:
- Backend: ADR-001, 002, 004, 005, 006, 009
- Frontend: ADR-003, 006, 012
- Database: ADR-006, 009
- API: ADR-008
- Security: ADR-007
- Testing: ADR-010
- DevOps: ADR-011

### 4. Follow Agent Workflow
Use `.context/workflows/AGENT_WORKFLOW_GUIDE.md` for collaboration patterns.

---

## 📝 Framework Maintenance

### Framework Updates
- **ADRs**: Created by Lead Software Engineer, apply to all projects
- **Agent Definitions**: Updated by Instructor Agent when needed
- **Workflows**: Updated by Lead Software Engineer based on lessons learned

### Project-Specific Content
- **Requirements**: Stored in project folder
- **Sprint Plans**: Stored in project folder
- **Work Logs**: Stored in project folder
- **Code**: Stored in project folder

---

## 🔗 Adding New Projects

To start a new project using SoftAI:

1. Create new workspace folder (e.g., `SoftAI-ProjectName/`)
2. Copy `.context/` structure from existing project
3. Update this registry with new project details
4. Follow Sprint 0 process for foundation setup

---

**Last Framework Update**: 2026-02-10 (All 12 ADRs established)

---

## 🤝 Pending Delegations

_Agents use this section to request work from other specialized agents_

### Example Template (delete when adding real delegations):
```markdown
### Request from [Agent A] to [Agent B]
- **Requested by**: [Your Agent Name]
- **Target Agent**: [Specialist Agent Name]
- **Request**: [Specific task needed]
- **Context**: [Why needed, relevant files/docs]
- **Priority**: High/Medium/Low
- **Blocking**: Yes/No
- **Expected Deliverable**: [What you need back]
```

_No pending delegations currently_

---

## 🤝 Active Discussions

_Agents use this section when they need to collaborate or resolve disagreements_

### Example Template (delete when adding real discussions):
```markdown
### Topic: [Discussion Topic]
**Participants**: [Agent A], [Agent B]
**Issue**: [What needs to be resolved]
**Agent A Position**: [Their view and reasoning]
**Agent B Position**: [Their view and reasoning]
**Status**: Discussing / Escalated to Lead / Resolved
**Resolution**: [If resolved, what was decided]
```

_No active discussions currently_

---

## 📊 Project Overview

### Project Information
- **Name**: [To be defined]
- **Type**: Web Application
- **Stack**: C# .NET (Backend) + Angular 21 (Frontend)
- **Architecture**: Clean Architecture with CQRS pattern
- **Code Location**: `/SoftAI-Example/` workspace folder (separate from agent system)

### Status Summary
- **Backend**: Not started (awaiting requirements)
- **Frontend**: Not started (awaiting requirements)
- **Database**: Not started (awaiting requirements)
- **Tests**: Not started
- **DevOps**: Not started
- **Documentation**: ✅ Context management system complete

---

## 🔄 Active Sessions

### Session Tracking
No active development sessions yet. This file will track:
- Which agent is currently working
- What specific task they're executing
- Expected completion time
- Blockers or dependencies

---

## 📝 Recent Decisions

Link to recent ADRs (most recent first):
- [ADR-001: CQRS with MediatR](../docs/adr/001-backend-cqrs-with-mediatr.md) - Backend architecture pattern (example)

---

## 🚧 Blockers & Issues

### Current Blockers
_Document any issues preventing progress. Use escalation format for Lead/Human decisions._

None

### Example Escalation Template (delete when adding real blockers):
```markdown
### Escalation: [Decision Needed]
- **Issue**: [What decision is needed]
- **Impact**: [Who/what is blocked]
- **Context**: [Background, what you were doing]
- **Options**:
  - Option A: [Description]
    - Pros: [Benefits]
    - Cons: [Drawbacks]
  - Option B: [Description]
    - Pros: [Benefits]
    - Cons: [Drawbacks]
- **Recommendation**: [Optional agent suggestion]
- **Decision Needed By**: Lead Software Engineer / Human
- **Urgency**: High/Medium/Low
```

### Open Questions
- What is the specific application being built?
- What are the core features required?
- What is the deployment target?

### Sprint 0 Decisions (Approved by Stakeholder)
- ✅ **Workflow**: Option B - Batch Review of all ADRs
- ✅ **Database**: SQL Server
- ✅ **Error Format**: RFC 7807 Problem Details
- ✅ **Logging**: Serilog with structured logging
- ✅ **Tech Stack**: .NET 10, Angular 21, PrimeNG for UI components

---

## 📋 Next Agent Handoff

### For Next Agent
- **Recommended Agent**: Lead Software Engineer
- **Context Needed**: User requirements, project scope
- **Expected Task**: Initial project planning and architecture design
- **Preparation Required**: None

---

## 📚 Context for Quick Start

### Essential Files to Read (in order)
1. This file (PROJECT_STATE.md) - Current state
2. CONTEXT_INDEX.md - Find relevant documentation
3. archive/WORK_LOG.md - What's been completed
4. Recent ADRs in /docs/adr/ - Key decisions

### Key Patterns & Conventions
- **Backend**: CQRS with MediatR (Commands for writes, Queries for reads)
- **Frontend**: Angular 21 with Signals for state management
- **Testing**: Test-driven approach with handlers tested in isolation
- **Architecture**: Clean Architecture (Core → Infrastructure → API)

---

## 🎓 Learning from Previous Sessions

### What Went Well
- [Track successes to repeat]

### What Could Improve
- [Track learnings for future sessions]

### Technical Debt
- [Track debt items with priority]

---

## 📞 Communication Notes

### Stakeholder Feedback
- [Track user/stakeholder input]

### Agent Coordination Notes
- [Track inter-agent dependencies or coordination needs]

---

**How to use this file:**
1. **Start of session**: Read this file first for current state
2. **During work**: Update "Active Work" section
3. **End of session**: Update status, add to archive/WORK_LOG.md, prepare handoff
4. **Making decisions**: Create ADR, link here
5. **Blocked**: Document in Blockers section with context
