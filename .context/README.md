# Context Management System

**Efficient context management for AI agent-driven development**

---

## 🎯 Purpose

This directory contains the context management system that enables:
- **Small context windows**: Agents load only what they need
- **Seamless handoffs**: Any agent can pick up where another left off  
- **Session continuity**: Work flows smoothly across multiple sessions
- **Decision preservation**: Architectural decisions are documented and findable
- **Progress tracking**: Clear visibility into what's been done and what's next

---

## 📁 Files in This Directory

### [PROJECT_STATE.md](PROJECT_STATE.md) ⭐ START HERE
**The single source of truth for current project state**

Read this first, every session. Contains:
- Current focus and active work
- Pending delegations between agents
- Active discussions and collaboration
- Project status summary
- Recent decisions (with ADR links)
- Current blockers and escalations
- Next agent handoff information

**Update this**: Start of session (claim work), during work (delegations/discussions), and end of session (status + handoff)

---

### [CONTEXT_INDEX.md](CONTEXT_INDEX.md)
**Quick reference map to find information fast**

Use this to:
- Find documentation relevant to your task
- Understand where specific information lives
- Learn project structure and conventions
- Identify appropriate search strategies
- Minimize context window usage

**Update this**: When adding new major documentation or establishing new patterns

---

### [WORK_LOG.md](WORK_LOG.md)
**Historical record of all development sessions**

Chronicles:
- What was completed in each session
- Which agents worked on what
- Decisions made (with ADR links)
- Blockers encountered and resolved
- Files created or modified

**Update this**: End of every session with session entry

---

### [AGENT_HANDOFF_PROTOCOL.md](AGENT_HANDOFF_PROTOCOL.md)
**Standard procedures for agent transitions**

Defines:
- How to start a session (what to read, in what order)
- What to do during work (how to track progress)
- How to end a session (what to update, how to handoff)
- Agent-specific guidelines for different roles
- Handling blockers and escalations

**Read this**: When learning the system or if handoff quality is poor

---

### [AGENT_WORKFLOW_GUIDE.md](AGENT_WORKFLOW_GUIDE.md)
**Complete workflow guide for AI agents**

Comprehensive guide covering:
- Phase 1: Session Start (loading context efficiently)
- Phase 2: Work Execution (doing the work)
- Phase 3: Session End (documentation and handoff)
- Agent-to-agent delegation protocols
- Conflict resolution and escalation paths
- Decision-making authority levels (Agent → Peer → Lead → Human)
- Agent coordination patterns
- Best practices and common pitfalls

**Read this**: When onboarding, need delegation guidance, or workflow is unclear

---

### [DELEGATION_ESCALATION_FLOWS.md](DELEGATION_ESCALATION_FLOWS.md) 📊
**Visual diagrams and quick reference for agent collaboration**

Visual reference covering:
- Delegation flow diagram (agent-to-agent requests)
- Escalation flow diagram (conflict resolution)
- Decision authority matrix (what to decide vs escalate)
- Collaboration pattern diagrams
- Time-boxing guidelines
- Quick reference checklists

**Read this**: When you need to delegate work, resolve disagreements, or escalate decisions

---

### [SPRINT_TRACKER.md](SPRINT_TRACKER.md) 🏃
**Agile sprint planning, backlog, and progress tracking**

Manages sprints covering:
- Current sprint goals and progress
- Sprint backlog with user stories and tasks
- Story point estimation and velocity tracking
- Task prioritization (P0 blocking → P3 nice-to-have)
- Sprint ceremonies and definitions (DoR/DoD)
- Integration with PROJECT_STATE for active work

**Read this**: At sprint start, when planning work, or checking what to work on next

**Update this**: When task status changes, at end of sprint, during sprint planning

---

## 🚀 Quick Start

### For Agents Starting Work

**5-Minute Context Load:**

1. **Read [PROJECT_STATE.md](PROJECT_STATE.md)** (5 min)
   - Current focus and active work
   - Your specific handoff instructions
   - Recent decisions and blockers

2. **Skim [CONTEXT_INDEX.md](CONTEXT_INDEX.md)** (2 min)
   - Find relevant documentation for your task
   - Understand how to search for information

3. **Load task-specific context** (10 min)
   - Your agent specification (.github/agents/)
   - Relevant ADRs (docs/adr/)
   - Specific code files you'll modify

**Total: ~15-20 minutes to full context**

---

## 🔄 Workflow Summary

```
┌─────────────────────────────────────────┐
│         SESSION START                   │
│  1. Read PROJECT_STATE.md               │
│  2. Check CONTEXT_INDEX.md              │
│  3. Load task-specific context          │
│  4. Update "Active Work"                │
└─────────────┬───────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│         WORK EXECUTION                  │
│  • Follow agent-specific guidelines     │
│  • Document decisions (ADR if major)    │
│  • Track progress and blockers          │
│  • Maintain clear commits               │
└─────────────┬───────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│         SESSION END                     │
│  1. Update WORK_LOG.md                  │
│  2. Update PROJECT_STATE.md             │
│  3. Create ADRs if needed               │
│  4. Prepare next agent handoff          │
│  5. Commit all changes                  │
└─────────────────────────────────────────┘
```

---

## 📊 System Benefits

### Before Context Management System
- ❌ Agents read entire codebase (context overload)
- ❌ Unclear what's been done or what's next
- ❌ Decisions lost or undocumented
- ❌ Poor handoffs between agents
- ❌ Duplicate work or conflicts

### After Context Management System
- ✅ Agents read only what's needed (~20% of codebase)
- ✅ Clear visibility into status and next steps
- ✅ All decisions documented in ADRs
- ✅ Smooth transitions between agents
- ✅ Coordinated work, no conflicts

---

## 🎯 Design Principles

1. **Single Source of Truth**: PROJECT_STATE.md is always authoritative
2. **Minimal Context**: Load only what's needed for current task
3. **Progressive Disclosure**: Start broad (state), drill down (specifics)
4. **Link, Don't Duplicate**: Reference detailed docs instead of copying
5. **Timeless Documentation**: Write for future developers without current context

---

## 🛠️ Integration with Agent System

### Agent Hierarchy
```
Instructor Agent (Meta)
    │
    ▼
Lead Software Engineer (Orchestrator)
    │
    ├─> Backend Developer ─────┐
    ├─> Frontend Developer ─────┤
    ├─> Database Engineer ──────┤
    ├─> QA/Testing Agent ───────┤
    ├─> DevOps Engineer ────────┤──> All use context system
    ├─> Security Engineer ──────┤
    ├─> Code Reviewer ──────────┤
    ├─> API Designer ───────────┤
    └─> Documentation Agent ────┘
```

**Every agent**:
- Starts session by reading PROJECT_STATE.md
- Updates state during and after work
- Follows handoff protocol
- Documents decisions
- Prepares clear handoffs

---

## 📚 Related Documentation

- **Agent Specifications**: `.github/agents/` - How each agent works
- **Architecture Decisions**: `docs/adr/` - Why things are the way they are
- **API Documentation**: `docs/api/` - API contracts and specs
- **Developer Guides**: `docs/guides/` - Setup and how-to guides

---

## 🔧 Maintenance

### Keeping the System Healthy

**Daily (during active development)**:
- Update PROJECT_STATE.md every session
- Add WORK_LOG.md entries after work
- Create ADRs for significant decisions

**Weekly**:
- Review PROJECT_STATE.md for stale information
- Verify ADR index is up to date
- Check that handoffs are working smoothly

**Monthly**:
- Archive old WORK_LOG entries if file becomes large
- Review and update CONTEXT_INDEX.md
- Evaluate system effectiveness, improve as needed

---

## ❓ FAQ

**Q: Do I really need to update PROJECT_STATE.md every session?**  
A: Yes! It's the single source of truth. Without updates, the system breaks down.

**Q: When should I create an ADR vs just document in code?**  
A: Create ADR for architectural/design decisions that affect multiple parts of system. Use code comments for implementation details.

**Q: What if PROJECT_STATE.md gets too large?**  
A: It shouldn't - link to details instead of duplicating. If it does, refactor by moving old content to WORK_LOG or archive.

**Q: Can agents work in parallel?**  
A: Yes, if working on non-overlapping areas. Lead Engineer coordinates via PROJECT_STATE.md to prevent conflicts.

**Q: What if handoff is unclear?**  
A: Document the confusion in PROJECT_STATE.md → Blockers, escalate to Lead Engineer.

---

## 🎓 Learning Resources

1. **New to the system?**
   → Read [AGENT_WORKFLOW_GUIDE.md](AGENT_WORKFLOW_GUIDE.md)

2. **Need to handoff work?**
   → Read [AGENT_HANDOFF_PROTOCOL.md](AGENT_HANDOFF_PROTOCOL.md)

3. **Can't find information?**
   → Check [CONTEXT_INDEX.md](CONTEXT_INDEX.md)

4. **Making a decision?**
   → See `docs/adr/README.md` for ADR guidance

5. **Current project state?**
   → Always start with [PROJECT_STATE.md](PROJECT_STATE.md)

---

## 📞 System Issues?

If the context management system isn't working:
1. Review [AGENT_WORKFLOW_GUIDE.md](AGENT_WORKFLOW_GUIDE.md) for best practices
2. Check if all files are being updated properly
3. Verify agents are following the workflow
4. Propose improvements to Lead Software Engineer
5. Update system documentation to reflect improvements

---

**The goal**: Enable efficient, coordinated, high-quality development with AI agents. If something doesn't serve that goal, improve it!

---

Last Updated: 2026-02-10
