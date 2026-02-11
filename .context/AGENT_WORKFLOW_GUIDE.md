# Agent Workflow Guide

**Complete guide for AI agents working on this project**

---

## 🎯 Quick Reference: Key Capabilities

### ✅ What Agents CAN Do (New in v2)

1. **Delegate to Other Agents**: Directly request help from specialists (see [🤝 Agent-to-Agent Delegation](#-agent-to-agent-delegation))
2. **Resolve Disagreements**: Follow escalation path → Agent discussions → Lead Engineer → Human (see [⚖️ Conflict Resolution](#️-conflict-resolution--escalation))
3. **Make Autonomous Decisions**: Implementation details within your domain
4. **Consult Peers**: Coordinate with other agents on shared concerns
5. **Escalate Strategically**: Know when to ask Lead vs Human

### 📖 Critical Sections

- **[🤝 Agent-to-Agent Delegation](#-agent-to-agent-delegation)**: How to request work from other agents
- **[⚖️ Conflict Resolution & Escalation](#️-conflict-resolution--escalation)**: 4-level decision authority, when to escalate
- **[🔄 Agent Coordination Patterns](#-agent-coordination-patterns)**: 6 collaboration patterns with examples
- **[Delegation Protocol](#delegation-protocol)**: Step-by-step process for requesting help
- **[Decision-Making Authority](#decision-making-authority-levels)**: What you can decide vs escalate

---

## 🎯 Overview

This project uses a structured approach to manage context, enable agent handoffs, and maintain continuity across sessions. This guide explains the complete workflow.

---

## 🏗️ System Architecture

### Context Management System
```
.context/
├── PROJECT_STATE.md         # Single source of truth for current state
├── WORK_LOG.md              # Historical record of sessions
├── AGENT_HANDOFF_PROTOCOL.md # Handoff procedures
└── CONTEXT_INDEX.md         # Quick reference map

docs/adr/                    # Architecture decisions
.github/agents/              # Agent specifications
```

### Design Principles
1. **Minimal Context Windows**: Agents read only what's needed
2. **Progressive Disclosure**: Start broad, drill down as needed
3. **Single Source of Truth**: PROJECT_STATE.md is authoritative
4. **Link, Don't Duplicate**: Reference instead of copying
5. **Continuity**: Any agent can pick up where another left off

---

## 🚀 Complete Agent Workflow

### Phase 1: Session Start (5-10 minutes)

#### Step 1.1: Read Current State
```
→ Read: .context/PROJECT_STATE.md
```

Look for:
- **Current Focus**: What's actively being worked on
- **Active Agent**: Who was working last (is it still you?)
- **Immediate Next Steps**: What needs to be done
- **Blockers**: Any obstacles to be aware of
- **Recent Decisions**: Links to recent ADRs

#### Step 1.2: Orient Yourself
```
→ Read: .context/CONTEXT_INDEX.md
```

Use CONTEXT_INDEX.md to:
- Find documentation relevant to your task
- Understand where to find specific information
- Learn project structure and conventions
- Identify search strategies

#### Step 1.3: Load Task-Specific Context

**For New Feature Implementation:**
- Read relevant ADRs (architecture, your domain)
- Review your agent specification (.github/agents/)
- Check existing code examples in your domain
- Review API contracts if frontend/backend integration

**For Bug Fixes:**
- Read recent WORK_LOG.md entries (when was this last working?)
- Search for related code semantically
- Check if there's an ADR about the area

**For Code Review:**
- Read files under review
- Check coding standards ADRs
- Review related tests

#### Step 1.4: Verify Understanding
Ask yourself:
- ✅ Do I understand what I'm supposed to do?
- ✅ Do I have the context I need?
- ✅ Are there any blockers I should know about?
- ✅ Do I know what patterns/conventions to follow?

If no to any → Check CONTEXT_INDEX.md or escalate to Lead Engineer

---

### Phase 2: Work Execution

#### Step 2.1: Update PROJECT_STATE.md
```markdown
## 🎯 Current Focus

### Active Work
- [YOUR TASK]: Implementing [specific feature]
  - Status: In progress
  - Agent: [Your agent name]
  - Started: [timestamp]
```

This announces you're working and prevents conflicts.

#### Step 2.2: Execute Your Task

Follow your agent-specific guidelines:

**Backend Developer:**
1. Implement command/query with handler
2. Add FluentValidation validator
3. Add thin controller endpoint
4. Add unit tests for handler
5. Add integration tests for endpoint

**Frontend Developer:**
1. Create standalone component
2. Implement service with signals
3. Add routing if needed
4. Style with Angular Material or custom
5. Add component tests

**Database Engineer:**
1. Define entities/relationships
2. Create migration
3. Update DbContext
4. Test migration up/down
5. Document schema changes

**QA/Testing:**
1. Review code to test
2. Write unit tests (handlers, services)
3. Write integration tests (APIs)
4. Write E2E tests (user flows)
5. Verify coverage meets standards

**DevOps:**
1. Configure infrastructure
2. Set up CI/CD pipeline
3. Configure environments
4. Test deployment
5. Document deployment process

**Security:**
1. Review security requirements
2. Implement auth/authz
3. Add input validation
4. Test security measures
5. Document security implementation

**Code Reviewer:**
1. Read code changes
2. Check against standards
3. Verify tests exist
4. Check for security issues
5. Provide constructive feedback

**API Designer:**
1. Understand requirements
2. Design endpoints (REST principles)
3. Create OpenAPI spec
4. Share with backend/frontend
5. Document API decisions

**Documentation:**
1. Understand what to document
2. Read existing docs for style
3. Write clear, concise documentation
4. Include code examples
5. Review for completeness

#### Step 2.3: Document Decisions

**For Significant Decisions (Architectural, Technical):**

1. Create new ADR:
```bash
# Next sequential number, descriptive name
docs/adr/00X-topic-decision-brief.md
```

2. Copy from `docs/adr/TEMPLATE.md`
3. Fill in all sections:
   - Context (why this decision?)
   - Decision (what are we doing?)
   - Consequences (trade-offs)
   - Alternatives Considered
   - Implementation Notes

4. Get reviewed by Lead Engineer
5. Update status to "Accepted"
6. Reference ADR in PROJECT_STATE.md

**For Minor Decisions:**
- Document in code comments
- Note in WORK_LOG.md if worth mentioning
- Don't create ADR for everything

#### Step 2.4: Maintain Context

As you work:
- **Keep notes** of what you're doing (for WORK_LOG.md later)
- **Track blockers** encountered
- **Note decisions** made
- **Document surprises** or unexpected complexity

---

### Phase 3: Session End (10-15 minutes)

#### Step 3.1: Complete Your Work
- ✅ Tests passing
- ✅ Code committed with clear messages
- ✅ Documentation updated
- ✅ No obvious blockers left for next agent

#### Step 3.2: Update WORK_LOG.md

Add session entry:
```markdown
## Session N - 2026-02-10

**Agent(s)**: [Your agent name]
**Duration**: [time]
**Phase**: [Development/Testing/etc.]

### Work Completed
- ✅ [Completed task 1]
- ✅ [Completed task 2]

### Decisions Made
- [Decision with brief rationale, link to ADR if exists]

### Files Created/Modified
- [path/to/file.cs] - [what changed]

### Blockers Encountered
- [Blocker and how resolved, or status if unresolved]

### Next Session Preparation
- [What next agent needs to know]
```

#### Step 3.3: Update PROJECT_STATE.md

Update multiple sections:

**Status Summary:**
```markdown
### Status Summary
- **Backend**: [Updated status]
- **Frontend**: [Updated status]
- ... update your domain ...
```

**Recent Decisions:**
```markdown
## 📝 Recent Decisions

Link to recent ADRs (most recent first):
- [ADR-00X: Your Decision](docs/adr/00X-your-decision.md) - Brief summary
```

**Blockers (if any):**
```markdown
## 🚧 Blockers & Issues

### Current Blockers
- **[Blocker description]**
  - Impact: Who/what is blocked
  - Context: What you were doing
  - Needs: What's required to unblock
  - Owner: Who should resolve
```

**Next Agent Handoff:**
```markdown
## 📋 Next Agent Handoff

### For Next Agent
- **Recommended Agent**: [Which agent should continue]
- **Context Needed**: [What they need to know]
- **Expected Task**: [What you expect them to do]
- **Files to Review**:
  - [file1.cs] - [why relevant]
  - [file2.ts] - [why relevant]
- **Preparation Required**: [Any setup needed]
- **Known Issues**: [Any caveats or problems]
```

#### Step 3.4: Final Checklist

Before ending session:
- [ ] WORK_LOG.md updated with session entry
- [ ] PROJECT_STATE.md updated (status, handoff, blockers)
- [ ] ADRs created for significant decisions
- [ ] Code committed with clear messages
- [ ] Tests passing
- [ ] Documentation updated
- [ ] Next agent has clear path forward

---

## 🤝 Agent-to-Agent Delegation

### When Agents Can Delegate Directly

Agents **should delegate** to other specialized agents when:
- Task requires expertise outside their domain
- Work can be done in parallel to unblock progress
- Discovered issue needs specialist attention
- Need input/review from another domain expert

### Delegation Protocol

**Step 1: Recognize Need**
Identify that another agent's expertise is needed:
- "I need database schema changes" → Database Engineer
- "I need API contract defined" → API Designer  
- "I found a security vulnerability" → Security Engineer
- "This code needs review" → Code Reviewer

**Step 2: Document Request in PROJECT_STATE.md**
```markdown
## 🤝 Pending Delegations

### Request from [Your Agent] to [Target Agent]
- **Requested by**: Backend Developer Agent
- **Target Agent**: Database Engineer Agent
- **Request**: Add `LastModifiedDate` column to Products table
- **Context**: Implementing audit trail feature, need schema update
- **Priority**: Medium (blocking CreateProductCommand completion)
- **Files Relevant**: 
  - src/Project.Core/Entities/Product.cs (shows new property)
  - docs/adr/007-audit-trail-strategy.md (explains requirement)
- **Expected Deliverable**: Migration script adding column with default value
- **Can Continue Without**: No, blocking current work
```

**Step 3: Target Agent Responds**
Target agent reads PROJECT_STATE.md, sees delegation:
```markdown
### Response from Database Engineer
- **Status**: ✅ Completed
- **Work Done**: Created migration `20260210_AddProductAuditFields`
- **Files Modified**: 
  - src/Project.Infrastructure/Migrations/20260210_AddProductAuditFields.cs
  - src/Project.Core/Entities/Product.cs (updated entity)
- **Notes**: Also added `CreatedDate` for consistency
- **Handback to**: Backend Developer Agent
```

**Step 4: Original Agent Continues**
Original agent sees completion, continues their work.

### Delegation Examples

#### Example 1: Backend → Database Engineer
```markdown
### Request: Schema Change Needed
**From**: Backend Developer Agent  
**To**: Database Engineer Agent  
**Need**: Add index on `Products.Category` for query performance  
**Why**: GetProductsByCategoryQuery is slow with 10k+ products  
**Context**: [link to slow query code]  
**Blocking**: No, can continue with other features  
```

#### Example 2: Frontend → API Designer
```markdown
### Request: API Contract Clarification
**From**: Frontend Developer Agent  
**To**: API Designer Agent  
**Need**: Clarify pagination format for GET /api/products  
**Why**: OpenAPI spec unclear on page vs offset-based  
**Context**: docs/api/products-api.yaml lines 45-67  
**Blocking**: Yes, cannot implement product list component  
```

#### Example 3: Any Agent → Security Engineer
```markdown
### Request: Security Review
**From**: Backend Developer Agent  
**To**: Security Engineer Agent  
**Need**: Review authentication implementation for security issues  
**Why**: First time implementing JWT, want validation  
**Context**: src/Project.API/Auth/* (all files)  
**Blocking**: No, but should review before merging  
```

#### Example 4: Backend → Frontend (Coordination)
```markdown
### Notification: API Contract Changed
**From**: Backend Developer Agent  
**To**: Frontend Developer Agent  
**Changed**: Updated Product DTO with new `imageUrls` array field  
**Why**: Supporting multiple product images  
**Context**: docs/api/products-api.yaml (updated)  
**Action Needed**: Update ProductCardComponent to display image gallery  
**Breaking Change**: No, added optional field with default empty array  
```

### Cross-Agent Collaboration Patterns

#### Pattern A: Parallel Work with Sync Points
```markdown
## Active Work

### Backend Developer - Product API Implementation
- Status: In progress (60% complete)
- Waiting on: Database Engineer to add indexes
- Can continue: Yes, working on other endpoints
- Sync needed: Before integration testing

### Database Engineer - Performance Optimization  
- Status: In progress
- Working on: Adding indexes for Backend Developer
- Expected completion: 1 hour
- Will notify: Backend Developer when complete
```

#### Pattern B: Round-Trip Refinement
```markdown
1. Backend Developer implements feature
2. Code Reviewer finds issues → delegates back to Backend Developer
3. Backend Developer fixes issues
4. Code Reviewer approves → delegates to QA/Testing
5. QA/Testing finds bug → delegates to Backend Developer  
6. Backend Developer fixes → delegates to QA/Testing
7. QA/Testing approves → complete
```

#### Pattern C: Multi-Agent Feature Implementation
```markdown
**Feature**: User Authentication

**Coordination Plan**:
1. Lead Engineer designs architecture → creates ADR
2. API Designer creates auth endpoints spec
3. Database Engineer creates Users/Roles schema
4. Security Engineer implements JWT/password hashing
5. Backend Developer implements controllers using Security Engineer's auth
6. Frontend Developer implements login/register UI
7. QA/Testing tests entire auth flow
8. DevOps Engineer adds secrets management
```

---

## ⚖️ Conflict Resolution & Escalation

### Decision-Making Authority Levels

#### Level 1: Agent (Autonomous Decisions)
**Can decide without escalation:**
- Implementation details within their domain
- Variable naming, file organization
- Specific library usage (within approved tech stack)
- Test strategy for their code
- Refactoring their own code
- Bug fixes that don't change architecture

**Examples:**
- Backend Dev: Choose to use LINQ vs foreach loop
- Frontend Dev: Component structure and internal state management
- Database Engineer: Index type (B-tree vs Hash)

#### Level 2: Peer Consultation (Agent-to-Agent)
**Should consult other agents:**
- Changes affecting another agent's domain
- API contract modifications
- Shared code or interfaces
- Cross-cutting concerns

**Process**: Use delegation protocol above, discuss in PROJECT_STATE.md

**Examples:**
- Backend Dev wants to change API response format → consult Frontend Dev
- Frontend Dev needs new API endpoint → consult API Designer + Backend Dev
- Database Engineer wants to change entity relationships → consult Backend Dev

#### Level 3: Lead Engineer (Architectural Decisions)
**Must escalate to Lead:**
- Architectural pattern changes
- Technology stack additions
- Cross-domain design decisions
- Performance vs. maintainability trade-offs
- Security architecture decisions
- When peer agents disagree

**Process**: Document issue in PROJECT_STATE.md → Blockers section

**Examples:**
- Should we cache at database or application layer?
- Should we use WebSockets or SSE for real-time features?
- How to handle multi-tenancy at data layer?
- Backend and Frontend disagree on pagination approach

#### Level 4: Human (Business/Requirements Decisions)
**Lead Engineer escalates to Human:**
- Business logic or requirements ambiguity
- Feature prioritization
- Non-functional requirements (SLA, performance targets)
- Cost vs. complexity trade-offs
- When Lead cannot determine "correct" technical choice

**Process**: Lead documents options, marks as needing human decision

**Examples:**
- "Eventually consistent" vs "strongly consistent" data (business impact)
- Free tier feature limits (business decision)
- Support for older browsers (user base decision)
- Two equally valid technical approaches with different trade-offs

### Escalation Process

#### For Agent-to-Agent Disagreements

**Scenario**: Backend and Frontend agents disagree on error handling approach

**Step 1: Agents Try to Resolve (15-30 min)**
```markdown
## 🤝 Active Discussion

### Topic: Error Response Format
**Participants**: Backend Developer, Frontend Developer  
**Issue**: Backend wants RFC 7807 Problem Details, Frontend wants simpler format  
**Backend Position**: Standardized, extensible, industry standard  
**Frontend Position**: Simpler to consume, less boilerplate parsing  
**Status**: Cannot reach agreement  
```

**Step 2: Escalate to Lead Engineer**
```markdown
## 🚧 Blockers & Issues

### Escalation: Error Response Format Decision Needed
- **Issue**: Backend and Frontend disagree on error format
- **Impact**: Blocking error handling implementation on both sides
- **Options**:
  - Option A: RFC 7807 Problem Details (Backend preference)
    - Pros: Standard, extensible, detailed
    - Cons: More complex to parse in frontend
  - Option B: Simple {error, message, code} (Frontend preference)
    - Pros: Simple, easy to consume
    - Cons: Non-standard, less extensible
- **Recommendation**: [Optional - agents can suggest]
- **Decision Needed By**: Lead Software Engineer
- **Urgency**: Medium - can work on other features for 2-3 days
```

**Step 3: Lead Engineer Reviews and Decides**
```markdown
### Decision: Error Response Format
- **Decision Maker**: Lead Software Engineer
- **Decision**: Use RFC 7807 Problem Details with TypeScript helper class
- **Rationale**: 
  - Standard format enables tooling and middleware
  - Frontend concern addressed by creating typed wrapper
  - Can add convenience accessor properties
  - Future-proof as app grows in complexity
- **Action Items**:
  - Backend: Implement RFC 7807 response format
  - Backend: Create frontend-friendly documentation with examples
  - Frontend: Create ErrorResponse utility class with simple API
- **ADR Created**: docs/adr/008-error-response-format.md
```

#### For Technical Decisions Lead Cannot Resolve

**Scenario**: Lead uncertain about multi-tenancy approach

**Lead Engineer Documents for Human:**
```markdown
## ❓ Human Decision Required

### Decision: Multi-Tenancy Data Isolation Approach

**Context**: Building SaaS app with multiple customer tenants. Need to decide data isolation strategy.

**Technical Options** (Lead's Analysis):

**Option 1: Separate Databases per Tenant**
- Pros: Complete isolation, easy backup/restore per tenant, meets compliance easier
- Cons: Higher operational cost, migration complexity, resource overhead
- Cost: ~$50/month per tenant (estimation)

**Option 2: Shared Database with Row-Level Security**
- Pros: Lower cost, simpler operations, better resource utilization
- Cons: Risk of data leakage if bugs, more complex queries, backup all-or-nothing
- Cost: ~$200/month total (estimation)

**Option 3: Separate Schemas per Tenant**  
- Pros: Balance of isolation and cost, logical separation
- Cons: Limited by DB schema count, still shared resources
- Cost: ~$200-300/month total (estimation)

**Lead's Assessment**:
- All three are technically viable
- Option 1 best for enterprise/compliance-heavy customers
- Option 2 best for high-volume, price-sensitive market
- Option 3 middle ground

**Business Questions Needed**:
1. What is target customer profile? (Enterprise vs SMB vs Indie)
2. What's the expected tenant count? (10s, 100s, 1000s?)
3. Any compliance requirements? (HIPAA, SOC2, GDPR-specific)
4. What's acceptable operational cost per tenant?
5. Is data isolation a competitive differentiator?

**Recommended**: [Lead can suggest, but needs business input]

**Urgency**: High - blocks multi-tenant architecture design

**Impact**: This decision affects:
- Database architecture
- API request handling
- Testing strategy  
- Deployment complexity
- Cost structure
```

**Human Responds:**
```markdown
### Human Decision: Multi-Tenancy Approach

**Decision**: Option 1 - Separate Databases per Tenant

**Business Context**:
- Target: Enterprise B2B customers (10-50 tenants expected)
- Compliance: Yes, healthcare data (HIPAA required)
- Pricing: $500+ per tenant/month, cost is acceptable
- Differentiator: "Complete data isolation" is key selling point

**Additional Requirements**:
- Must support tenant-specific backup/restore
- Each tenant may have custom schema extensions later
- Some tenants will require on-premise database option

**Action for Lead**: Design architecture for Option 1, create ADR, delegate implementation
```

### Conflict Resolution Guidelines

**For Agents in Disagreement:**
- **Stay Professional**: Focus on technical merits, not personal preference
- **Document Clearly**: Write out your reasoning with concrete pros/cons
- **Be Open**: Consider the other agent's perspective seriously
- **Time-Box**: Don't debate endlessly, escalate after 30 minutes
- **Accept Decisions**: Once Lead decides, implement faithfully

**For Lead Engineer:**
- **Understand Both Sides**: Read each agent's full reasoning
- **Consider Context**: What's the project phase, timeline, constraints?
- **Make Timely Decisions**: Don't leave agents blocked for days
- **Explain Reasoning**: Help agents learn from the decision
- **Document in ADR**: Preserve rationale for future reference
- **Escalate to Human**: When business context is needed for technical choice

**For Human Decision-Maker:**
- **Provide Business Context**: Help technical team understand priorities
- **Trust Technical Expertise**: Lead has vetted the options technically
- **Be Timely**: Technical team is waiting on you
- **Ask Questions**: If options are unclear, ask Lead to clarify
- **Document Decision**: Rationale helps team understand and execute

---

## 🔄 Agent Coordination Patterns

### Pattern 1: Sequential Work (One Agent at a Time)

**Scenario**: Backend Developer → QA/Testing Agent

**Backend Developer end-of-session:**
```markdown
### For Next Agent
- **Recommended Agent**: QA/Testing Agent
- **Context Needed**: Just implemented CreateProductCommand and handler
- **Expected Task**: Write unit tests for CreateProductCommandHandler
- **Files to Review**:
  - src/Project.Application/Products/Commands/CreateProductCommand.cs
  - src/Project.Application/Products/Commands/CreateProductCommandHandler.cs
- **Preparation Required**: None
- **Known Issues**: None
```

**QA/Testing Agent start-of-session:**
1. Reads PROJECT_STATE.md → sees handoff is for them
2. Reviews the files mentioned
3. Reads backend-developer.agent.md handler example
4. Implements tests following CQRS testing patterns
5. Updates PROJECT_STATE.md when done

### Pattern 2: Parallel Preparation (Multiple Agents Can Work Simultaneously)

**Scenario**: API Designer creates spec → Backend and Frontend work in parallel

**Lead Engineer coordinates:**
```markdown
### For Next Agents

**API Designer** (do first):
- Create OpenAPI spec for Products API
- Expected: 2 hours

**Then in parallel:**

**Backend Developer**:
- Implement Products endpoints per spec
- Read: docs/api/products-api.yaml

**Frontend Developer**:
- Implement Products UI components
- Read: docs/api/products-api.yaml
- Mock API responses initially

**Coordinate**: Both agents should verify contract matches
```

### Pattern 3: Iterative Refinement (Agent Revisits Work)

**Scenario**: Code Reviewer finds issues → Backend Developer fixes

**Code Reviewer:**
```markdown
### For Next Agent
- **Recommended Agent**: Backend Developer Agent
- **Context Needed**: Code review found validation issues in CreateProductCommand
- **Expected Task**: Add price validation (must be positive), handle null name
- **Files to Review**:
  - src/Project.Application/Products/Commands/CreateProductCommandValidator.cs (needs fixes)
- **Preparation Required**: Read my review comments in the code
- **Known Issues**: Tests don't cover edge cases
```

### Pattern 4: Escalation to Lead

**Scenario**: Agent encounters architectural decision

**Any Specialized Agent:**
```markdown
## 🚧 Blockers & Issues

### Current Blockers
- **Need architectural decision on data partitioning strategy**
  - Impact: Cannot proceed with multi-tenant implementation
  - Context: Implementing tenant data isolation, unclear if should use separate schemas or row-level security
  - Needs: Architectural decision from Lead Engineer
  - Options: [Brief description of options]
  - Attempted Resolution: Consulted Database Engineer, no consensus
  
### For Next Agent
- **Recommended Agent**: Lead Software Engineer
- **Context Needed**: Need to decide multi-tenant data isolation strategy
- **Expected Task**: Evaluate options, create ADR, delegate implementation back
```

### Pattern 5: Agent Delegation (Direct Collaboration)

**Scenario**: Backend Developer needs schema change from Database Engineer

**Backend Developer (delegates):**
```markdown
## 🤝 Pending Delegations

### Request to Database Engineer
- **From**: Backend Developer Agent
- **Need**: Add foreign key constraint on OrderItems → Orders
- **Why**: Preventing orphaned order items after Order deletion
- **Context**: src/Project.Core/Entities/OrderItem.cs
- **Blocking**: No, can continue with other features
- **Expected**: Migration script with CASCADE delete
```

**Database Engineer (responds):**
```markdown
### Completed Delegation
- **Original Request**: Add FK constraint OrderItems → Orders  
- **Status**: ✅ Done
- **Deliverable**: Migration_AddOrderItemsForeignKey.cs
- **Note**: Used CASCADE delete as requested, also added index for performance
- **Handback to**: Backend Developer Agent (can now complete Order deletion feature)
```

### Pattern 6: Multi-Agent Conflict Resolution

**Scenario**: Backend and Frontend disagree, escalate to Lead, Lead escalates to Human

**Phase 1 - Agent Disagreement:**
```markdown
## 🤝 Active Discussion

### Real-time Updates vs Polling
**Backend Developer**: Prefers WebSockets for real-time order updates  
**Frontend Developer**: Prefers polling every 5s (simpler, works with existing infra)  
**Status**: Cannot agree, escalating to Lead
```

**Phase 2 - Lead Analysis:**
```markdown
## 🚧 Escalation Analysis

### Technical Decision: Real-time Updates Implementation

**Lead's Technical Assessment**:
- WebSockets: Better UX, more complex infra, requires sticky sessions
- Polling: Simpler, scales easier, 5s delay acceptable for orders
- Both technically sound

**Lead Cannot Decide Because**: Need to know if 5-second delay acceptable for business

**Escalating to Human with Business Question**:
"Is 5-second delay in order status updates acceptable for users, or do we need instant updates?"
```

**Phase 3 - Human Decision:**
```markdown
## ✅ Human Decision

**Question**: 5-second delay vs instant updates?  
**Answer**: 5-second delay is acceptable. Users typically refresh order page manually.

**Business Context**: 
- Most users check orders 1-2 times, not continuously
- Instant updates not mentioned in user feedback
- Simpler infrastructure reduces operational risk for v1.0

**Decision**: Use polling approach (Frontend preference)  
**Rationale**: Meets user needs, simpler to operate, can upgrade to WebSockets later if data shows need
```

**Phase 4 - Lead Implements Decision:**
```markdown
## 📝 Recent Decisions

- [ADR-012: Order Status Updates via Polling](docs/adr/012-order-status-polling.md)
  - Decision: Polling every 5 seconds
  - Rationale: Business confirmed delay acceptable, operational simplicity priority
  - Can evolve to WebSockets in future if needed

### For Next Agent
- **Recommended Agent**: Frontend Developer Agent  
- **Task**: Implement 5-second polling for order status page
- **Context**: Human confirmed polling approach acceptable
```

---

## 🎓 Best Practices

### Context Window Management

**DO:**
- ✅ Read PROJECT_STATE.md first (always)
- ✅ Use CONTEXT_INDEX.md to find what you need
- ✅ Read only relevant ADRs
- ✅ Search semantically for concepts
- ✅ Link to files instead of copying content

**DON'T:**
- ❌ Read all code files "just in case"
- ❌ Copy entire files into documents
- ❌ Re-explain decisions (link to ADR)
- ❌ Read WORK_LOG.md unless you need history
- ❌ Load more context than your task requires

### Making Decisions

**Create ADR when:**
- Architecture or design patterns
- Technology selection
- Security approaches
- Integration patterns
- Performance strategies

**Don't create ADR when:**
- Implementation details
- Variable naming
- Refactoring existing code
- Bug fixes (unless revealing architectural issue)

### Writing for Future Agents

**Good documentation:**
- Concise with clear action items
- Links to details instead of duplicating
- Explains "why" not just "what"
- Anticipates questions

**Poor documentation:**
- Vague ("do the thing")
- Missing context
- Overly verbose
- No clear next steps

### Handling Uncertainty

**When stuck:**
1. Check CONTEXT_INDEX.md - is documentation available?
2. Search semantically for examples
3. Check WORK_LOG.md - was this attempted before?
4. Try agent-to-agent delegation if another domain is involved
5. Document the uncertainty in PROJECT_STATE.md
6. Escalate to Lead Engineer with specific question and options

**Escalation Decision Tree:**
```
Problem Encountered
  |
  ├─ Implementation detail in my domain?
  │  └─ YES → Decide autonomously
  |
  ├─ Affects another agent's domain?
  │  └─ YES → Delegate/consult other agent
  │            |
  │            ├─ Agents agree? → Proceed
  │            └─ Agents disagree? → Escalate to Lead
  |
  ├─ Architectural decision?
  │  └─ YES → Escalate to Lead with options
  │            |
  │            ├─ Lead can decide? → Lead decides
  │            └─ Needs business input? → Lead escalates to Human
  |
  └─ Requirements unclear?
     └─ YES → Escalate to Human with specific questions
```

**Don't:**
- Make guesses about architecture or business requirements
- Implement without understanding requirements
- Skip documentation because unclear
- Ignore blockers hoping they resolve
- Debate with other agents indefinitely (time-box to 30 min)

---

## 🚨 Common Pitfalls

### Pitfall 1: Context Overload
**Problem**: Agent reads everything, consumes entire context window  
**Solution**: Follow Phase 1 workflow, read only what's needed for task

### Pitfall 2: Poor Handoffs
**Problem**: Next agent doesn't know what to do  
**Solution**: Use handoff template, be specific and actionable

### Pitfall 3: Undocumented Decisions
**Problem**: Future agents don't understand why something was done  
**Solution**: Create ADRs for significant decisions, reference in code

### Pitfall 4: Stale PROJECT_STATE.md
**Problem**: Current state is wrong, agents work on outdated information  
**Solution**: Update PROJECT_STATE.md at start and end of every session

### Pitfall 5: Parallel Conflicts
**Problem**: Two agents modify same files simultaneously  
**Solution**: Lead Engineer coordinates, use "Active Work" section to claim work

### Pitfall 6: Endless Debates
**Problem**: Two agents argue back and forth without resolution  
**Solution**: Time-box disagreements to 30 minutes, then escalate to Lead

### Pitfall 7: Premature Escalation
**Problem**: Agent escalates simple questions that could be resolved with research  
**Solution**: Check CONTEXT_INDEX and search semantically before escalating

### Pitfall 8: Silent Blockers
**Problem**: Agent is blocked but doesn't document it, work stalls  
**Solution**: Document blockers immediately in PROJECT_STATE.md, request help

---

## 📊 Success Metrics

Evaluate workflow effectiveness:

### Context Efficiency
- ⏱️ Time to Context: < 10 minutes to understand current state
- 📄 Context Size: Agent loads < 20% of total codebase context
- 🎯 Context Relevance: > 80% of loaded content is used

### Handoff Quality
- ✅ Clear Next Steps: Next agent starts work immediately
- 📝 Complete Information: No back-tracking for missing context
- 🔄 Smooth Transitions: < 5 minutes to understand handoff

### Collaboration Efficiency
- 🤝 Delegation Response Time: < 4 hours for non-blocking requests
- ⚖️ Conflict Resolution: Disagreements resolved within 24 hours
- 📊 Escalation Clarity: 100% of escalations include clear options

### Documentation Quality
- 📚 ADR Coverage: All major decisions documented
- 🔍 Findability: Can find any decision in < 2 minutes
- ⏳ Timeless: Docs remain useful 6+ months later

---

## 🎯 Quick Reference

### Start of Session Checklist
- [ ] Read PROJECT_STATE.md
- [ ] Check CONTEXT_INDEX.md for relevant docs
- [ ] Load task-specific context only
- [ ] Update "Active Work" in PROJECT_STATE.md
- [ ] Verify you understand the task

### During Session Checklist
- [ ] Follow your agent-specific guidelines
- [ ] Document significant decisions (ADR if major)
- [ ] Keep notes for WORK_LOG entry
- [ ] Track any blockers encountered
- [ ] Maintain clear commit messages

### End of Session Checklist
- [ ] Tests passing
- [ ] Code committed
- [ ] WORK_LOG.md updated
- [ ] PROJECT_STATE.md updated
- [ ] ADRs created if needed
- [ ] Clear handoff prepared
- [ ] Documentation updated

---

## 📞 Getting Help

**Need clarification?**
→ Check CONTEXT_INDEX.md first

**Architecture/design decision needed?**
→ Escalate to Lead Software Engineer with specific question

**Missing documentation?**
→ Note in PROJECT_STATE.md, consider creating it if you have the knowledge

**Blocked on another agent's work?**
→ Document in Blockers section, coordinate via project_state.md

**System not working as expected?**
→ Propose improvement, discuss with Lead Engineer

---

**Remember**: This system exists to enable efficient, high-quality work. If something isn't working, improve it!

---

Last Updated: 2026-02-10
