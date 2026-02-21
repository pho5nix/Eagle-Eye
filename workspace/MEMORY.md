# EagleEye — Memory

## Environment

| Field              | Value                |
| ------------------ | -------------------- |
| **Installed**      | 2026-02-18           |
| **Host**           | agent-007            |
| **OS**             | Ubuntu 24.04 LTS     |
| **Tools Verified** | 2026-02-20           |
| **Skill Version**  | 1.0 (decision trees) |

---

## Session Guidelines

### Starting New Engagement
```
1. Send /new or /reset to clear context
2. Load minimal startup files (see AGENTS.md)
3. Verify target in TARGETS.md
4. Execute 9-phase methodology
5. Update memory at session end
```

### Session Limits
- **Duration**: 5 hours maximum
- **Plan**: Complete phases within single session
- **If interrupted**: Resume from last completed phase

### Post-Engagement
Update workspace files with learnings:
- SKILL.md (new techniques)
- TOOLS.md (tool issues/solutions)
- MEMORY.md (engagement notes)
- memory/YYYY-MM-DD.md (daily log)

---

## Continuous Improvement Learnings

### Multi-Wordlist Strategy (2026-02-20)

**Problem**: Single wordlist misses findings; huge wordlists waste time.

**Solution**: Balanced multi-wordlist approach

| Phase | Wordlists | Time |
|-------|-----------|------|
| Subdomains | top1million-5000.txt + all.txt | 5-10 min |
| Directories | common.txt + medium.txt | 20-25 min |
| **AVOID** | 100K+ single wordlists | Diminishing returns |

**Rule**: ~30-40 min total for discovery phases

---

### WAF Analysis Pattern (2026-02-20)

**Observation**: Cloudflare + Vercel challenge tokens block non-browser clients.

**Lesson**:
- Multi-layer WAF = near-impossible bypass
- Document WAF architecture in reports
- Focus on business logic, not infrastructure, if WAF is hardened
- Check for WAF early (Phase 5) to adjust strategy

---

### Decision Tree Integration (2026-02-20)

**Added**: Comprehensive decision trees for all 12 tools in SKILL.md

**Benefits**:
- Faster tool selection
- Consistent methodology
- Clear fallback options
- Time budget awareness

**Reference**: See SKILL.md for full decision trees

---

### Token Optimization (2026-02-20)

**Problem**: Large tool outputs consume context window.

**Solutions**:
1. Pipeline commands (single tool call)
2. Save to file, report path only
3. Summarize counts, not full output
4. Reference prior context, don't re-paste

**Example**:
```bash
# Bad (3 tool calls, pastes output)
subfinder -d $TARGET
# [paste output]
dnsx -l subs.txt
# [paste output]

# Good (1 pipeline, save to file)
subfinder -d $TARGET -silent | dnsx -a -silent -o resolved.txt
echo "Resolved: $(wc -l < resolved.txt)"
```

---

## Engagement Notes

### Template
```markdown
## [Domain] - [Date]

### Scope
- In-scope: [targets]
- Out-of-scope: [exclusions]

### Key Findings
- [Finding 1]
- [Finding 2]

### Challenges
- [Challenge 1]

### Time Spent
- Phase 1-4: Xm
- Phase 5-7: Xm
- Total: Xm

### Follow-up
- [Action item]
```

### Past Engagements

*(Append notes here after each engagement)*

---

## Quick Reference

### Startup Load Order
```
SOUL.md → USER.md → IDENTITY.md → memory/today.md
```

### On-Demand Files
```
SKILL.md    → When recon starts
TOOLS.md    → When tool reference needed
AGENTS.md   → When workflow reference needed
```

### Memory Search
```
Use memory_search() for past context
Pull specific snippets with memory_get()
Never load entire memory files unprompted
```

---

## Version History

| Date | Change |
|------|--------|
| 2026-02-18 | Initial setup |
| 2026-02-19 | Tools verified working |
| 2026-02-20 | SKILL v2.0 with decision trees |
| 2026-02-20 | Multi-wordlist strategy documented |
| 2026-02-20 | Token optimization patterns added |
