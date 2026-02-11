# SoftAI Framework - Context Management System

**Context management templates and active project registry for AI agent-driven development**

---

## ⚠️ IMPORTANT: Framework vs Project

### This Folder (SoftAI/.context/)
Contains **reusable framework resources**:
- Active project registry ([PROJECT_STATE.md](PROJECT_STATE.md))
- Agent workflow templates ([workflows/](workflows/))
- Sprint planning templates ([sprints/](sprints/))
- Context management guidelines (this file)

### Actual Project Folders
**Real project work** happens in separate workspace folders:
- **DVD Rental System**: `../SoftAI-Example/` - Active development
- **Future Projects**: `../SoftAI-YourProject/`

Each project has its own `.context/` folder with:
- Project-specific `PROJECT_STATE.md`
- Project-specific `WORK_LOG.md`
- Requirements and sprint plans
- Project tracking and progress

---

## 🎯 Purpose of This System

This framework enables:
- **Small context windows**: Agents load only what they need
- **Seamless handoffs**: Any agent can pick up where another left off  
- **Clear separation**: Framework stays reusable, projects stay separate
- **Decision preservation**: ADRs documented once, used by all projects
- **Progress tracking**: Each project tracks its own progress

---

## 📁 Directory Structure

```
SoftAI/.context/              # Framework folder (this location)
├── PROJECT_STATE.md          # ⭐ Active projects registry
├── CONTEXT_INDEX.md          # Framework navigation guide
├── README.md                 # This file
│
├── workflows/                # Reusable agent patterns
│   ├── AGENT_WORKFLOW_GUIDE.md
│   ├── AGENT_HANDOFF_PROTOCOL.md
│   └── DELEGATION_ESCALATION_FLOWS.md
│
├── sprints/                  # Sprint templates
│   ├── SPRINT_TEMPLATE.md
│   └── sprint-0/             # Framework's own Sprint 0 (complete)
│
└── archive/                  # Framework development history
    └── WORK_LOG.md           # Framework evolution log

SoftAI-Example/.context/      # Project folder (separate workspace)
├── PROJECT_STATE.md          # Project's current state
├── WORK_LOG.md               # Project's session history
├── requirements/             # Project requirements
└── sprints/                  # Project sprint plans
```

---

## 📚 Framework Files

### [PROJECT_STATE.md](PROJECT_STATE.md) ⭐ START HERE
**Registry of active projects using SoftAI**

Contains:
- List of all active projects
- Links to each project's context folder
- Framework version and status
- Quick links to framework resources

**When to read**: When starting work to identify your project location

---

### [CONTEXT_INDEX.md](CONTEXT_INDEX.md)
**Framework navigation guide**

Use this to:
- Find ADRs relevant to your work
- Locate agent definitions
- Understand framework patterns
- Learn how to navigate project context

**When to read**: After identifying your project, to find framework standards

---

## 🚀 Getting Started with a Project

### For Agents Starting Work:

1. **Read Framework Registry** (1 min)
   - Open [PROJECT_STATE.md](PROJECT_STATE.md)
   - Find your project in the Active Projects list
   - Note the project folder location

2. **Navigate to Project Folder**
   - Go to the project workspace (e.g., `../SoftAI-Example/`)
   - Read the project's `PROJECT_STATE.md`
   - Review current sprint plan

3. **Review Framework Standards** (10-15 min)
   - Use [CONTEXT_INDEX.md](CONTEXT_INDEX.md) to find relevant ADRs
   - Read your agent definition in `../.github/agents/`
   - Review workflow patterns if needed

4. **Start Working**
   - Follow the project's sprint plan
   - Update the project's context (not framework context)
   - Use framework workflows for collaboration

---

## 🔄 Agent Workflow Templates

This folder contains **reusable workflow patterns** that apply to all projects:
- What to do during work (how to track progress)
- How to end a session (what to update, how to handoff)
- Agent-specific guidelines for different roles
- Handling blockers and escalations

**Read this**: When learning the system or if handoff quality is poor

---

### [workflows/AGENT_WORKFLOW_GUIDE.md](workflows/AGENT_WORKFLOW_GUIDE.md)
**Complete workflow guide for AI agents**

Comprehensive guide covering:
- Session start, work execution, session end
- Agent-to-agent delegation protocols
- Conflict resolution and escalation paths
- Decision-making authority levels
- Best practices and common pitfalls

**Use this**: For delegation guidance, conflict resolution, workflow clarification

---

### [workflows/AGENT_HANDOFF_PROTOCOL.md](workflows/AGENT_HANDOFF_PROTOCOL.md)
**Standard procedures for agent transitions**

Defines:
- How to start a session (what to read, in what order)
- What to do during work
- How to end a session and document handoffs

**Use this**: When learning the system or improving handoff quality

---

### [workflows/DELEGATION_ESCALATION_FLOWS.md](workflows/DELEGATION_ESCALATION_FLOWS.md) 📊
**Visual diagrams for agent collaboration**

Visual reference including:
- Delegation flow diagrams
- Escalation flow diagrams
- Decision authority matrix
- Quick reference checklists

**Use this**: When delegating work, resolving disagreements, or escalating decisions

---

### [sprints/SPRINT_TEMPLATE.md](sprints/SPRINT_TEMPLATE.md)
**Template for sprint planning**

Reusable template for creating sprint plans in projects. Each project uses this to create their own sprint documentation.

**Use this**: When planning a new sprint in your project

---

## 📝 Project Context vs Framework Context

### What Goes in Framework Context (SoftAI/.context/)
- Active project registry
- Workflow templates
- Sprint planning templates
- Framework evolution history

### What Goes in Project Context (e.g., SoftAI-Example/.context/)
- Project status and progress
- Project requirements
- Sprint plans and backlogs
- Project work logs
- Project-specific decisions

**Key Rule**: Framework = reusable, Project = specific implementation

---

## 🆕 Creating a New Project

To start a new project using SoftAI:

1. **Create Project Workspace**
   - Create folder: `SoftAI-YourProject/`
   - Create `.context/` subfolder

2. **Copy Context Structure**
   - Use `SoftAI-Example/.context/` as template
   - Copy folder structure and templates
   - Update with your project details

3. **Register Project**
   - Add to [PROJECT_STATE.md](PROJECT_STATE.md) Active Projects section
   - Include project name, location, status, description

4. **Follow Sprint 0 Process**
   - Requirements gathering
   - Domain modeling
   - Architecture design
   - Project scaffolding

---

## 📚 Additional Framework Resources

### Architecture Decision Records
Location: `../docs/adr/`
- All architectural standards and decisions
- Apply to all projects using SoftAI
- Created by Lead Software Engineer

### Agent Definitions
Location: `../.github/agents/`
- Specifications for all agent roles
- Responsibilities and guidelines
- Valid across all projects

### SoftAI README
Location: `../README.md`
- Framework overview
- System architecture
- Getting started guide

---

## 💡 Best Practices

### For All Agents
1. **Start with registry**: Read framework [PROJECT_STATE.md](PROJECT_STATE.md) to find your project
2. **Work in project context**: Update project's context, not framework's
3. **Reference framework**: Use ADRs and workflows, don't duplicate them
4. **Minimize context**: Load only what you need for current task

### For Lead Software Engineer
- Keep project registry current
- Update workflow templates based on lessons learned
- Ensure agents understand framework vs project separation
- Create new ADRs when architectural patterns emerge

---

**Last Updated**: 2026-02-10  
**Framework Version**: 1.0  
**Active Projects**: 1 (DVD Rental System)
