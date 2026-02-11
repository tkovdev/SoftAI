# ADR 013: Agent Delegation Protocol

## Status
Accepted

## Context
The Lead Software Engineer Agent has instructions to delegate work to specialized agents, but was not consistently using the `runSubagent` tool to actually spin up and coordinate with these agents. This created confusion between describing delegation conceptually vs. actually performing delegation.

## Decision

### Mandatory Delegation Using runSubagent Tool

The Lead Software Engineer Agent **MUST** use the `runSubagent` tool when delegating implementation work. Describing delegation without invoking the tool is not acceptable.

### When to Use runSubagent

**MUST delegate (use runSubagent):**
- Implementation of features by specialized agents (Backend Dev, Frontend Dev, Database Engineer)
- Test creation and test suite maintenance (QA/Testing Agent)
- Detailed code reviews checking patterns and style (Code Reviewer Agent)
- API specification creation (API Designer Agent)
- Security implementations like auth/authz (Security Engineer Agent)
- CI/CD pipeline setup and infrastructure (DevOps Engineer Agent)
- Technical documentation beyond architecture (Documentation Agent)
- Creating new agent specifications (Instructor Agent)
- Any work that falls within a specialized agent's domain

**Do NOT delegate (handle personally):**
- Architectural decisions and design
- Technology stack selection
- Architecture Decision Records (ADRs)
- High-level code reviews for architectural alignment
- Requirements clarification and technical feasibility
- System design and integration patterns
- Coordination between agents
- Final approval of delegated work

### Delegation Pattern

When delegating, follow this explicit pattern:

```
1. State the delegation clearly:
   "I'm delegating [task] to the [Agent Name]..."

2. Invoke runSubagent tool with:
   - description: Short 3-5 word task description
   - prompt: Detailed specification including:
     * What needs to be done
     * Acceptance criteria
     * Patterns/standards to follow
     * Integration points
     * Which other agents to coordinate with (if any)
     * What to return/report back

3. After agent returns, review and communicate results:
   "The [Agent Name] has completed [task].
    [Summary of what was delivered]
    [Any issues or follow-up needed]"
```

### Example Delegation

**Good Example:**
```
I'm delegating the Customer API implementation to the Backend Developer Agent.

[Invokes runSubagent with detailed specs about endpoints, CQRS patterns, etc.]

The Backend Developer Agent has implemented the Customer API with CQRS handlers.
Reviewing the implementation for architectural alignment...
[Reviews output and provides feedback]
```

**Bad Example (DO NOT DO THIS):**
```
I'll delegate the Customer API implementation to the Backend Developer Agent.
The implementation should follow CQRS patterns...
[Proceeds to implement it themselves without using runSubagent]
```

### Communication Requirements

When working on multi-step tasks:

1. **Before delegating**: State which agent will handle what
2. **During delegation**: Use runSubagent tool (don't just describe it)
3. **After delegation**: Review outputs and state results
4. **Be explicit**: Make it clear when you're delegating vs. working yourself

### Delegation Specifications

When invoking runSubagent, provide comprehensive specifications:

**Required in every delegation prompt:**
- Clear description of what to implement/create
- Acceptance criteria (what defines "done")
- Relevant standards, patterns, or conventions to follow
- Where the work fits in the architecture
- What files/components to create or modify
- Expected output or deliverables

**Optional but recommended:**
- Example code showing expected patterns
- Related documentation or ADRs to reference
- Other agents to coordinate with
- Testing requirements
- Performance or security considerations

### Review and Accountability

After receiving output from a delegated agent:

1. **Review for quality**: Does it meet acceptance criteria?
2. **Review for integration**: Does it align with architecture and other components?
3. **Review for standards**: Does it follow established patterns?
4. **Provide feedback**: If issues exist, clearly document them
5. **Accept or request revision**: Make a clear decision

The Lead Software Engineer Agent maintains accountability for all delegated work.

### Verification and Validation Requirements ⚠️ CRITICAL

**NEVER mark a task or story as complete without verification.** This applies to both delegated work and work done by the Lead Engineer directly.

**Mandatory verification steps:**

1. **Build Verification** (Code Implementation)
   - Run `dotnet build` (or equivalent for the technology stack)
   - Verify **ZERO build errors** (warnings are acceptable if documented)
   - If build fails, fix issues before proceeding
   - **Never assume code compiles - always verify**

2. **Runtime Verification** (When Applicable)
   - Start the application/service
   - Verify it runs without crashing
   - Test critical paths and endpoints
   - Check logs for runtime errors

3. **Acceptance Criteria Verification**
   - Go through each acceptance criterion line by line
   - Test or inspect to verify each one is met
   - Document which are complete, which are not
   - **Never mark "complete" without checking all criteria**

4. **Integration Verification**
   - Verify changes integrate with existing code
   - Check for breaking changes
   - Test interactions with other components
   - Verify dependencies are resolved

5. **Version Control** (After All Verification Complete)
   - Once all verification steps pass, **commit changes to git using `run_in_terminal`**
   - Stage all changes: `git add -A`
   - Commit with meaningful message: `git commit -m "[Story ID] Description..."`
   - **DO NOT just provide commit commands for user to run - actually execute them**
   - Provide a clear summary of what was committed after completion
   - Never proceed to next task without committing completed work
   - **Format**: "Committing changes... ✅ Committed: [hash] - [summary]"

**When delegating implementation work**, instruct the agent to:
- Build and verify their code compiles
- Test that code actually runs
- Verify all acceptance criteria
- Report any issues found

**When receiving delegated work**, always:
- Request verification results ("Did it build? Did you test it?")
- If agent doesn't report verification, ask them to verify
- Don't accept work that hasn't been verified
- Re-verify yourself if in doubt

**Lesson Learned (2026-02-11):**
> During Sprint 1 Story F-3 implementation, the Lead Engineer completed code changes 
> but marked the task complete without verifying the build succeeded. The user had 
> to point out build errors existed. This violated the principle that work is never 
> complete until it's verified to actually work.
>
> **Rule**: If you haven't run `dotnet build` (or equivalent) and seen "Build succeeded", 
> you haven't finished the implementation work.

**Red Flags to Watch For:**
- ❌ "I've implemented..." without mentioning build/test results
- ❌ Marking task complete without verification steps
- ❌ Assuming code works without running it
- ❌ Skipping verification "to save time"
- ❌ Delegating work without requiring verification from agent
- ❌ Moving to next task without committing completed work

**Correct Approach:**
- ✅ "I've implemented X. Building... Build succeeded with 0 errors."
- ✅ "I've asked the Backend Agent to implement and verify the build."
- ✅ "The agent reports: Build succeeded, tests passing, all ACs met."
- ✅ "Let me verify the build before marking this complete..."
- ✅ "All verification complete. Ready to commit. Suggested commit message: `[Story F-3] Implement API infrastructure`"
- ✅ "[After user commits] Moving to next story..."

### Creating New Specialized Agents

When a needed capability doesn't exist in the current agent team:

1. Recognize the gap in specialization
2. Delegate to Instructor Agent with requirements for the new agent
3. Review the created agent specification
4. Use the new agent for the required work

## Consequences

### Positive
- Clear distinction between planning and execution
- Specialized agents handle implementation in their domains
- Lead agent focuses on architecture and coordination
- Explicit audit trail of who did what
- Better simulation of real team dynamics
- Consistent use of the runSubagent tool

### Negative
- Additional overhead of agent coordination
- Need to write detailed specifications for each delegation
- Potential for miscommunication between agents
- Need to review all delegated work

### Mitigation
- Provide comprehensive specifications to reduce back-and-forth
- Use clear acceptance criteria to align expectations
- Maintain architectural documentation that agents can reference
- Review delegated work promptly and provide clear feedback

## Compliance Check

Before submitting any response involving implementation work, the Lead Software Engineer Agent should ask:

1. ✅ Is this implementation work that a specialized agent should handle?
2. ✅ Did I use the `runSubagent` tool to actually delegate?
3. ✅ Did I provide clear specifications and acceptance criteria?
4. ✅ Did I review the agent's output before accepting it?
5. ✅ **Did I verify the code builds and runs successfully?** (NEW - Critical)
6. ✅ **Did I verify all acceptance criteria are met?** (NEW - Critical)
7. ✅ **Did I test that the implementation actually works?** (NEW - Critical)
8. ✅ **Did I ask the user to commit the changes to git?** (NEW - Critical)

If any answer is "no" when it should be "yes", **STOP and fix it before proceeding**.

**Special emphasis on verification (added 2026-02-11):**
- If you wrote code: Did you run the build command and see success?
- If you delegated code: Did the agent report successful build/test results?
- If marking something complete: Have you verified every acceptance criterion?
- **After all verification passes: Did you ask user to commit before moving on?**

**Never mark work complete without verification. This is non-negotiable.**
**Never move to the next task without asking user to commit completed work.**

## Related ADRs
- ADR-006: Technology Stack
- ADR-010: Testing Strategy
- All ADRs that define standards agents must follow

## References
- runSubagent tool documentation
- Lead Software Engineer Agent mode instructions

---
**Author**: Lead Software Engineer Agent  
**Date**: 2026-02-11  
**Last Updated**: 2026-02-11

## Changelog

### 2026-02-11 (Evening) - Added Git Commit Requirement
**Context**: User clarified that after verification is complete, the Lead Engineer should ask to commit changes to git before proceeding to the next task.

**Changes Added**:
- Added step 5 "Version Control" to verification requirements
- Requirement to ask user to commit after all verification passes
- Added to red flags: moving to next task without committing
- Added to compliance checklist: check #8 about git commits
- Updated correct approach examples to show commit workflow

**Impact**: Ensures completed work is committed to version control before moving forward. Prevents loss of work and maintains proper version history.

### 2026-02-11 (Afternoon) - Added Verification and Validation Requirements
**Context**: During Sprint 1 Story F-3 implementation, work was marked complete without verifying the build succeeded, leading to undiscovered errors.

**Changes Added**:
- New section: "Verification and Validation Requirements"
- Mandatory build verification before marking work complete
- Requirement to verify all acceptance criteria are met
- Enhanced compliance checklist with verification checks (now 8 total)
- Documented lesson learned and anti-patterns to avoid

**Impact**: All future delegated work and personal implementation must include verification steps. This ensures work quality and prevents incomplete implementations from being accepted.

**Rationale**: "Works on my machine" is not acceptable. Code must provably compile, run, and meet acceptance criteria before being considered complete.
