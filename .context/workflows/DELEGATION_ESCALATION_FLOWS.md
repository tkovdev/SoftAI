# Agent Delegation & Escalation Flows

**Visual reference for agent collaboration and conflict resolution**

---

## 🔀 Delegation Flow (Agent-to-Agent)

```
┌─────────────────────────────────────────────────────────────┐
│  Agent A discovers need for specialized work                │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Agent A documents request in PROJECT_STATE.md              │
│  Section: 🤝 Pending Delegations                            │
│  - What's needed                                            │
│  - Context & files                                          │
│  - Priority & blocking status                               │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Agent B reads PROJECT_STATE.md                             │
│  - Sees delegation request for them                         │
│  - Reviews context and requirements                         │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Agent B completes work                                     │
│  - Implements requested feature                             │
│  - Tests and validates                                      │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Agent B documents response in PROJECT_STATE.md             │
│  - What was delivered                                       │
│  - Files modified                                           │
│  - Notes for Agent A                                        │
│  - Status: ✅ Completed                                      │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│  Agent A continues their work                               │
│  - Uses Agent B's deliverable                               │
│  - Completes original feature                               │
└─────────────────────────────────────────────────────────────┘
```

### Example Scenarios

**Scenario 1**: Backend → Database Engineer
- Backend needs: Index for performance
- Database delivers: Migration with index
- Backend continues: Completes feature with optimized queries

**Scenario 2**: Frontend → API Designer
- Frontend needs: API contract clarification
- API Designer delivers: Updated OpenAPI spec
- Frontend continues: Implements component with correct API calls

**Scenario 3**: Any Agent → Security Engineer
- Agent needs: Security review
- Security delivers: Review report with findings
- Agent continues: Fixes security issues, implements suggestions

---

## ⚖️ Escalation Flow (Conflict Resolution)

```
┌─────────────────────────────────────────────────────────────┐
│                      Problem Encountered                     │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
                  ┌─────────┐
                  │Decision?│
                  └────┬────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│Implementation│ │Cross-cutting │ │Architectural │
│   Detail     │ │   Concern    │ │  Decision    │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │
       ▼                ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ LEVEL 1:     │ │ LEVEL 2:     │ │ LEVEL 3:     │
│ Agent Decides│ │ Peer Consult │ │ Lead Decides │
│ Autonomously │ │ (Delegate)   │ │ (Escalate)   │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │
       │                ▼                │
       │         ┌─────────────┐         │
       │         │Agents Agree?│         │
       │         └──────┬──────┘         │
       │                │                │
       │        ┌───────┼───────┐        │
       │        │               │        │
       │        ▼               ▼        │
       │    ┌───────┐      ┌───────┐    │
       │    │  YES  │      │  NO   │    │
       │    └───┬───┘      └───┬───┘    │
       │        │              │        │
       │        ▼              │        │
       │    Implement      ────┘        │
       │                               │
       └───────────┬───────────────────┘
                   │
                   ▼
            ┌──────────────┐
            │ Implemented  │
            └──────────────┘

                   │
                   ▼ (if architectural or business impact)
            ┌──────────────┐
            │ Lead Reviews │
            └──────┬───────┘
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
    ┌─────────┐         ┌─────────┐
    │Technical│         │Business │
    │Decision │         │Question │
    └────┬────┘         └────┬────┘
         │                   │
         ▼                   ▼
    ┌─────────┐         ┌─────────┐
    │  Lead   │         │ LEVEL 4:│
    │ Decides │         │  Human  │
    │Creates  │         │ Decides │
    │   ADR   │         └────┬────┘
    └────┬────┘              │
         │                   ▼
         │              ┌─────────┐
         │              │  Human  │
         │              │Provides │
         │              │Business │
         │              │Context  │
         │              └────┬────┘
         │                   │
         │                   ▼
         │              ┌─────────┐
         │              │  Lead   │
         │              │Implements│
         │              │Decision  │
         │              │Creates   │
         │              │   ADR    │
         │              └────┬────┘
         │                   │
         └───────────────────┘
                   │
                   ▼
            ┌──────────────┐
            │ Agents       │
            │ Implement    │
            │ Decision     │
            └──────────────┘
```

---

## 🎯 Decision Authority Matrix

| Decision Type | Level | Handler | Example | Time Limit |
|---------------|-------|---------|---------|------------|
| **Implementation Detail** | 1 | Agent | Variable naming, loop choice | Immediate |
| **Domain-Specific Pattern** | 1 | Agent | Component structure, query optimization | Immediate |
| **Cross-Agent Coordination** | 2 | Peer Consult | API contract, data format | 4 hours |
| **Shared Interface** | 2 | Peer Consult | DTO changes, error formats | 4 hours |
| **Disagreement** | 2→3 | Peer→Lead | Two valid approaches, no consensus | 30 min then escalate |
| **Architectural Pattern** | 3 | Lead Engineer | CQRS, layering, design patterns | 24 hours |
| **Technology Selection** | 3 | Lead Engineer | Library choice, framework decision | 24 hours |
| **Performance Trade-off** | 3 | Lead Engineer | Cache strategy, scalability approach | 24 hours |
| **Requirements Ambiguity** | 4 | Human | Feature scope, business rules | 48 hours |
| **Business Priority** | 4 | Human | Cost vs complexity, feature prioritization | 48 hours |
| **Technical with Business Impact** | 3→4 | Lead→Human | Multi-tenancy approach, real-time vs polling | Lead 24h, Human 48h |

---

## 🔄 Collaboration Patterns

### Pattern A: Independent Work
```
Agent A ──────→ Implements Feature A ──────→ Done
                                    
Agent B ──────→ Implements Feature B ──────→ Done
```

### Pattern B: Sequential Pipeline
```
Agent A → Completes Work → Handoff → Agent B → Completes Work → Done
```

### Pattern C: Delegation & Return
```
Agent A ──→ Needs Help ──→ Delegates to Agent B
              │
              └──→ Agent B Delivers ──→ Agent A Continues → Done
```

### Pattern D: Collaborative Refinement
```
Agent A → Initial Implementation
   ↓
Agent B → Review & Feedback
   ↓
Agent A → Incorporates Feedback
   ↓
Agent B → Approves → Done
```

### Pattern E: Multi-Agent Feature
```
                    Lead Engineer
                    (Designs & Coordinates)
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
   Database Eng.    Backend Dev         Frontend Dev
        │                  │                  │
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                      QA/Testing
                           │
                           ▼
                      DevOps Deploy
```

### Pattern F: Conflict Resolution
```
Agent A & Agent B → Disagreement
         │
         ├─→ Try to resolve (30 min)
         │        │
         │        ├─→ Resolved? → Implement
         │        │
         │        └─→ No resolution
         │                │
         └────────────────┘
                  ▼
           Escalate to Lead
                  │
             Lead Analyzes
                  │
          ┌───────┴───────┐
          ▼               ▼
    Can Decide?      Needs Business
          │              Input?
          │                │
     Create ADR     Escalate to Human
          │                │
          │           Human Decides
          │                │
          └────────────────┘
                  │
           Agents Implement
```

---

## 📋 Quick Reference Checklist

### When to Delegate (Agent-to-Agent)
- [ ] Need specialized expertise outside my domain
- [ ] Want to work in parallel to unblock progress
- [ ] Discovered issue needing specialist attention
- [ ] Need review or input from domain expert
- [ ] **Action**: Document in PROJECT_STATE.md → Pending Delegations

### When to Discuss (Agent-to-Agent)
- [ ] Change affects another agent's work
- [ ] Need to align on shared interface or contract
- [ ] Proposing change to existing pattern
- [ ] Want feedback before implementing
- [ ] **Action**: Document in PROJECT_STATE.md → Active Discussions

### When to Escalate to Lead
- [ ] Architectural pattern decision needed
- [ ] Technology/framework selection required
- [ ] Cross-domain design decision
- [ ] Performance vs maintainability trade-off
- [ ] Peer agents cannot reach agreement (after 30 min)
- [ ] **Action**: Document in PROJECT_STATE.md → Blockers (with options)

### When to Escalate to Human
- [ ] **Never directly** - Always through Lead Engineer
- [ ] Lead cannot determine which technical approach is "correct"
- [ ] Business requirements are ambiguous or conflicting
- [ ] Need clarity on business priorities or constraints
- [ ] Decision has significant business/cost implications
- [ ] **Action**: Lead documents in PROJECT_STATE.md → Human Decision Required

---

## ⏱️ Time-Boxing Guidelines

| Activity | Time Limit | What Happens After |
|----------|------------|-------------------|
| **Agent Research** | 30 min | If still stuck, delegate or escalate |
| **Agent Discussion** | 30 min | If no agreement, escalate to Lead |
| **Delegation Wait** | 4 hours | If not blocking, continue other work |
| **Lead Decision** | 24 hours | Lead creates ADR, delegates implementation |
| **Human Decision** | 48 hours | Lead implements human decision |

**Purpose of Time-Boxing**: Prevent analysis paralysis, keep momentum, ensure timely decisions

---

## 🎯 Success Indicators

### Healthy Delegation
- ✅ Requests are clear and actionable
- ✅ Context is complete (files, requirements)
- ✅ Responses include deliverables and notes
- ✅ Original agent can continue immediately

### Healthy Escalation
- ✅ Options are clearly documented
- ✅ Pros/cons analyzed for each option
- ✅ Escalations include specific questions
- ✅ Decisions are documented in ADRs
- ✅ Agents implement decisions faithfully

### Unhealthy Patterns (Avoid)
- ❌ Silent blockers (not documented)
- ❌ Vague delegation requests
- ❌ Endless debates without escalation
- ❌ Decisions without rationale
- ❌ Repeatedly asking same questions
- ❌ Bypassing escalation levels

---

**Remember**: The goal is efficient, collaborative progress with clear decision-making. When in doubt, document and communicate!

---

Last Updated: 2026-02-10
