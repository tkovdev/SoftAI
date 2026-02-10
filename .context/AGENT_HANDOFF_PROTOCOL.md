# Agent Handoff Protocol

Standard procedure for AI agents to hand off work, minimize context window size, and enable seamless continuation.

---

## 🎯 Purpose

Enable any agent to:
1. Quickly understand current project state (< 5 minutes reading)
2. Pick up exactly where previous agent left off
3. Maintain continuity without massive context windows
4. Preserve decisions and rationale

---

## 📋 Handoff Checklist

### Before Starting Work

- [ ] Read `PROJECT_STATE.md` for current state
- [ ] Check `WORK_LOG.md` for recent sessions
- [ ] Review `CONTEXT_INDEX.md` for relevant documentation
- [ ] Read recent ADRs (if making architectural decisions)
- [ ] Check for blockers or open questions
- [ ] Understand what the previous agent completed

### During Work

- [ ] Update `PROJECT_STATE.md` → "Active Work" section
- [ ] Document decisions in ADRs if significant
- [ ] Keep notes for work log entry
- [ ] Track any blockers encountered
- [ ] Maintain code comments for context

### After Completing Work

- [ ] Update `PROJECT_STATE.md` with new status
- [ ] Add session entry to `WORK_LOG.md`
- [ ] Create/update ADRs for architectural decisions
- [ ] Update `CONTEXT_INDEX.md` if new concepts introduced
- [ ] Prepare handoff section in PROJECT_STATE.md
- [ ] Commit code with descriptive commit messages
- [ ] Document any technical debt or future improvements

---

## 📝 Standard Handoff Format

When completing work, update PROJECT_STATE.md → "Next Agent Handoff" section:

```markdown
### For Next Agent
- **Recommended Agent**: [Which agent should continue]
- **Context Needed**: [What they need to know]
- **Expected Task**: [What you expect them to do]
- **Files to Review**: [Specific files to read]
- **Preparation Required**: [Any setup steps]
- **Known Issues**: [Any problems or warnings]
```

---

## 🎓 Context Window Management

### Principles
1. **Minimal Context**: Agent should read only what's needed for current task
2. **Progressive Disclosure**: Start with PROJECT_STATE.md, drill down as needed
3. **Link, Don't Duplicate**: Reference ADRs and docs instead of repeating
4. **Summary over Detail**: Provide summaries with links to details

### Context Reading Order
1. **PROJECT_STATE.md** (5 min) - Current state, immediate context
2. **CONTEXT_INDEX.md** (2 min) - Map of documentation
3. **Relevant ADRs** (10 min) - Only ADRs related to current work
4. **Specific Code Files** (as needed) - Only files being modified
5. **WORK_LOG.md** (if needed) - When deeper history is required

### Avoiding Context Bloat
- ❌ Don't copy entire files into PROJECT_STATE.md
- ❌ Don't re-explain decisions (link to ADR instead)
- ❌ Don't read all code (use semantic search for specific needs)
- ✅ Do provide clear, concise status updates
- ✅ Do link to detailed information
- ✅ Do highlight only what changed

---

## 🔄 Agent-Specific Guidelines

### Lead Software Engineer
- **Starting session**: Review all files in .context/ to understand full state
- **During session**: Coordinate work across multiple agents via PROJECT_STATE.md
- **Ending session**: Provide clear delegation instructions for specialized agents
- **Context scope**: Broad architectural overview, delegate details

### Backend Developer Agent
- **Starting session**: Check PROJECT_STATE.md → "Backend" status
- **Context scope**: Backend code, relevant ADRs, database schema (via Database Engineer notes)
- **Coordination**: Check API Designer's specs before implementing endpoints
- **Ending session**: Document new endpoints, update API status

### Frontend Developer Agent
- **Starting session**: Check PROJECT_STATE.md → "Frontend" status
- **Context scope**: Frontend code, API contracts, UI/UX requirements
- **Coordination**: Verify API contracts with Backend Developer notes
- **Ending session**: Document new components, update UI status

### Database Engineer Agent
- **Starting session**: Check PROJECT_STATE.md → "Database" status
- **Context scope**: Schema files, migrations, database ADRs
- **Coordination**: Ensure backend developer knows about schema changes
- **Ending session**: Document migrations, update schema version

### QA/Testing Agent
- **Starting session**: Check what features were recently implemented
- **Context scope**: Test files, feature implementations to test
- **Coordination**: Verify test coverage expectations with Lead Engineer
- **Ending session**: Report test coverage, document any bugs found

### DevOps Engineer Agent
- **Starting session**: Check PROJECT_STATE.md → "DevOps" status
- **Context scope**: Infrastructure files, CI/CD configs, deployment ADRs
- **Coordination**: Verify environment requirements with Backend/Frontend
- **Ending session**: Document deployment changes, update pipeline status

### Security Engineer Agent
- **Starting session**: Review recent code changes for security implications
- **Context scope**: Authentication/authorization code, security ADRs
- **Coordination**: Work with Backend Developer on auth implementation
- **Ending session**: Document security findings, update security checklist

### Code Reviewer Agent
- **Starting session**: Identify which files/PRs need review
- **Context scope**: Files under review, project coding standards, relevant ADRs
- **Coordination**: Provide feedback to implementing agent
- **Ending session**: Summarize review findings, approve or request changes

### API Designer Agent
- **Starting session**: Check what APIs need design
- **Context scope**: OpenAPI specs, business requirements, API ADRs
- **Coordination**: Share specs with Backend and Frontend developers
- **Ending session**: Publish updated OpenAPI specs

### Documentation Agent
- **Starting session**: Check what needs documentation
- **Context scope**: Code to document, existing docs, documentation standards
- **Coordination**: Gather information from implementing agents
- **Ending session**: Publish documentation updates

---

## 🚨 Handling Blockers

If you encounter a blocker:

1. **Document immediately** in PROJECT_STATE.md → "Blockers & Issues"
2. **Provide context**: What you were doing, what blocked you, what you tried
3. **Suggest solutions**: Even if uncertain, propose potential approaches
4. **Identify who can help**: Which agent or stakeholder can unblock
5. **Don't halt entirely**: Work on non-blocked tasks if possible
6. **Update handoff**: Make blocker visible to next agent

---

## ✅ Quality Standards

### Good Handoff Example
```markdown
### For Next Agent
- **Recommended Agent**: Backend Developer Agent
- **Context Needed**: Need to implement authentication endpoints
- **Expected Task**: Create Login and Register commands with JWT token generation
- **Files to Review**: 
  - docs/adr/003-jwt-authentication.md
  - src/ECommerce.API/Controllers/AuthController.cs (stubbed)
- **Preparation Required**: None, Security Engineer has configured JWT settings
- **Known Issues**: None
```

### Poor Handoff Example ❌
```markdown
### For Next Agent
- **Recommended Agent**: Someone
- **Context Needed**: Do the auth stuff
- **Expected Task**: Make it work
```

---

## 📊 Handoff Metrics

Track handoff quality (agents should self-assess):
- **Context Completeness**: Did next agent have all needed information? (Yes/No)
- **Smooth Transition**: Could next agent start immediately? (Yes/No)
- **Blocker Communication**: Were blockers clearly documented? (Yes/No)

These help improve handoff quality over time.

---

**Remember**: Good handoffs enable efficient work. Poor handoffs waste time and context windows.
