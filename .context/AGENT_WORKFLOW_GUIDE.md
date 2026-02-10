# Agent Workflow Guide

**Complete guide for AI agents working on this project**

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
  
### For Next Agent
- **Recommended Agent**: Lead Software Engineer
- **Context Needed**: Need to decide multi-tenant data isolation strategy
- **Expected Task**: Evaluate options, create ADR, delegate implementation
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
4. Document the uncertainty in PROJECT_STATE.md
5. Escalate to Lead Engineer with specific question

**Don't:**
- Make guesses about architecture
- Implement without understanding requirements
- Skip documentation because unclear
- Ignore blockers hoping they resolve

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
