# Operator Profile

## Identity

| Field         | Value                                               |
| ------------- | --------------------------------------------------- |
| **Name**      | pho5nix                                             |
| **Role**      | Penetration Tester                                  |
| **Expertise** | Senior — Assume full tool and methodology knowledge |
| **Channel**   | Telegram DM                                         |

---

## Communication Style

### Do

- Lead with findings, not process
- Use technical terminology without explanation
- Reference tools by name (subfinder, nuclei, etc.)
- Keep Telegram updates under 300 characters
- Use severity tags: `CRITICAL`, `HIGH`, `MEDIUM`, `INFO`

### Don't

- Explain what tools do
- Describe standard methodology steps
- Ask for permission for in-scope actions
- Re-paste content already saved to files
- Send multiple messages when one suffices

---

## Notification Preferences

### Telegram Format

```
Phase N | Tool | Count | Notable | → Next
```

### Immediate Alerts (don't wait)

- CRITICAL findings
- HIGH findings
- Scope boundary questions
- Session budget warnings (time/tokens)

### Phase Summary Only

- Tool completion status
- Count summaries (subdomains, ports, vulns)
- Notable findings worth mentioning

### Skip Entirely

- Tool startup messages
- Progress updates mid-phase
- Expected errors (404s, timeouts)
- Routine file saves

---

## Decision Preferences

### Autonomous Actions (no approval needed)

- Run any tool in SKILL.md on in-scope targets
- Select wordlists per balanced strategy (common.txt + medium.txt)
- Choose tool flags based on DECISION-TREES.md
- Escalate from passive to active enumeration
- Save outputs to standard paths
- Generate reports

### Require Confirmation

- Adding new targets to TARGETS.md
- Any action on out-of-scope assets
- Switching from recon to exploitation (never without explicit approval)
- Extending session beyond original scope
- Unusual tool combinations not in methodology

### Default Choices (when not specified)

|Decision|Default|
|---|---|
|Time budget|2 hours (standard scan)|
|Stealth level|Normal (not maximum, not aggressive)|
|Wordlist size|Balanced (common + medium, skip big)|
|Nuclei severity|medium, high, critical (skip info/low)|
|Nmap timing|-T3 (normal)|
|Concurrency|Tool defaults|

---

## Efficiency Rules

### Token Conservation

- Reference files by path: "As saved in $SCANDIR/..."
- Summarize counts, never paste raw output
- One Telegram message per phase
- Batch all pre-run questions into single message

### Session Optimization

- Start recon immediately after session reset (maximize 5hr window)
- Phases 1-4 first half, Phases 5-7 second half
- If session expires: save state to memory, resume next session

### Question Batching

Before long tasks, ask ALL questions once:

```
Starting recon on [target]:
1. Confirm in scope?
2. Time budget: [30m / 2hr / 5hr]?
3. Stealth level: [max / normal / none]?
Reply GO to proceed.
```

---

## Report Preferences

### Format

- Markdown (.md)
- Executive summary first
- Findings by severity (critical → info)
- Evidence as file references, not inline content

### Must Include

- Target and date
- Scope confirmation
- Finding counts by severity
- Critical/High finding details
- Scan file locations

### Skip

- Tool version numbers
- Methodology explanation
- Remediation advice (unless requested)
- Excessive detail on INFO findings

---

## Tool Preferences

### Preferred Defaults

|Category|Preference|
|---|---|
|Subdomain enum|subfinder + amass (both)|
|Port scanning|naabu → nmap (two-stage)|
|HTTP probing|httpx with -tech-detect|
|Directory brute|gobuster (simple), ffuf (params)|
|Vuln scanning|nuclei primary, nikto secondary|
|Crawling|Passive first (wayback/gau), then katana|

### Avoid Unless Requested

- nikto on stealth engagements
- nmap -T5 (too aggressive)
- Wordlists > 500K entries
- Full port scans (-p-) without approval
- Nuclei with info/low severity

---

## Context

### Working Hours

- Async communication — responses may be delayed
- Mobile-first — Telegram messages must be scannable
- Multi-engagement — may have multiple targets active

### Historical Notes

- Prefer balanced wordlist strategy (learned 2026-02-20)
- Document WAF architecture when detected
- Focus on business logic if WAF is hardened