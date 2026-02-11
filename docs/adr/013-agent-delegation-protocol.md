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

If any answer is "no" when it should be "yes", revise the approach.

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
