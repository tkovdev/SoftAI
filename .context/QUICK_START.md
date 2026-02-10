# Context Management Quick Start

**⏱️ 2-Minute Quick Reference for AI Agents**

---

## 🎯 Starting a Session? Read This

### Step 1: Understand Current State (5 min)
Read: [PROJECT_STATE.md](PROJECT_STATE.md)

Look for:
- [ ] What's currently being worked on?
- [ ] Is there work assigned to me?
- [ ] Are there any blockers?
- [ ] What's the immediate next step?

### Step 2: Find What You Need (2 min)
Skim: [CONTEXT_INDEX.md](CONTEXT_INDEX.md)

Find:
- [ ] Which docs relate to my task?
- [ ] Where is the relevant code?
- [ ] What patterns should I follow?

### Step 3: Load Task Context (10 min)
Read only:
- [ ] My agent spec (`.github/agents/[your-agent].agent.md`)
- [ ] Relevant ADRs (linked from PROJECT_STATE.md)
- [ ] Specific files I'll modify

### Step 4: Announce You're Working
Update PROJECT_STATE.md → "Active Work":
```markdown
- [Task Name]: In progress
  - Agent: [Your Name]
  - Started: [Time]
```

---

## 💼 During Work

✅ **Do:**
- Follow your agent-specific guidelines
- Document decisions (create ADR if architectural)
- Keep notes for WORK_LOG entry
- Maintain clear git commit messages

❌ **Don't:**
- Skip testing
- Make undocumented architectural decisions
- Read entire codebase "just in case"

---

## ✅ Ending Session

### Required Updates (10 min)

1. **Add to WORK_LOG.md:**
```markdown
## Session N - [Date]
**Agent**: [You]
**Duration**: [Time]
### Work Completed
- ✅ [What you did]
### Files Modified
- [file.cs] - [what changed]
```

2. **Update PROJECT_STATE.md:**
   - Update status for your domain
   - Add ADR links if created
   - Document blockers if any
   - **Prepare handoff** (most important!)

3. **Handoff Template:**
```markdown
### For Next Agent
- **Recommended Agent**: [Who should continue]
- **Context Needed**: [What they need to know]
- **Expected Task**: [What to do]
- **Files to Review**: [Specific files]
- **Preparation Required**: [Any setup]
- **Known Issues**: [Caveats]
```

---

## 🚨 Common Mistakes

❌ **Don't read everything** → Use semantic search
❌ **Don't skip handoff** → Next agent will be lost
❌ **Don't forget ADRs** → Decisions get lost
❌ **Don't work on stale state** → Always read PROJECT_STATE.md first

---

## 📊 File Priority

**Always Read:**
1. PROJECT_STATE.md (single source of truth)
2. CONTEXT_INDEX.md (find what you need)
3. Your agent spec (your guidelines)

**Read If Relevant:**
4. ADRs for your domain
5. Existing code you'll modify
6. WORK_LOG.md (if you need history)

**Read If Stuck:**
7. AGENT_WORKFLOW_GUIDE.md (complete workflow)
8. AGENT_HANDOFF_PROTOCOL.md (detailed procedures)

---

## 🎯 Success = Minimal Context + Clear Handoff

**Minimal Context**: Load < 20% of codebase
**Clear Handoff**: Next agent starts immediately

---

## 🔗 Key Links

- [PROJECT_STATE.md](PROJECT_STATE.md) - ⭐ START HERE
- [CONTEXT_INDEX.md](CONTEXT_INDEX.md) - Find information
- [AGENT_WORKFLOW_GUIDE.md](AGENT_WORKFLOW_GUIDE.md) - Complete workflow
- [AGENT_HANDOFF_PROTOCOL.md](AGENT_HANDOFF_PROTOCOL.md) - Handoff details
- [../docs/adr/](../docs/adr/) - Architecture decisions

---

**Remember**: If you follow this process, sessions are smooth. If you skip steps, things break down.

Good luck! 🚀
