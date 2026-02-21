# Eagle-Eye Recon Agent

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: OpenClaw](https://img.shields.io/badge/Platform-OpenClaw-red.svg)](https://openclaw.ai)
[![Model: claude-sonnet-4-5](https://img.shields.io/badge/Model-claude--sonnet--4--5-blue.svg)](https://anthropic.com)
[![Local LLM: Qwen2.5:7b](https://img.shields.io/badge/LocalLLM-Qwen2.5%3A7b-green.svg)](https://ollama.ai)
[![Phase: Recon](https://img.shields.io/badge/Phase-Reconnaissance-orange.svg)]()

## Overview

**Eagle-Eye** is an autonomous AI-powered penetration testing reconnaissance agent built on [OpenClaw](https://openclaw.ai).

Running on a lean Ubuntu 24.04 Proxmox VM, Eagle-Eye executes a full 9-phase recon pipeline autonomously: 
- OSINT
- Subdomain Enumeration
- Live Host Discovery
- Port Scanning
- Service Fingerprinting
- URL Crawling
- Vulnerability Scanning
- Report Generation
- Operator Notification

It uses **claude-sonnet-4-5** as its primary reasoning engine for complex decisions, with a local **Qwen2.5:7b** LLM fallback (via Ollama, zero cost) for repetitive subtasks, keeping API usage lean and sessions efficient.  
The agent doesn't just run tools. It reads per-tool decision trees, evaluates output quality, selects the right command variant for the situation, and retries with a different strategy when something fails. All without operator intervention.

### Key Features

- **Autonomous 9-Phase Pipeline**: Full recon workflow from OSINT to final report, executed without hand-holding.
- **Decision Tree Intelligence**: Per-tool knowledge files teach the agent to reason about failures, stealth requirements, and fallback strategies — not just memorize flags.
- **Hybrid LLM Architecture**: `claude-sonnet-4-5` (Pro) for complex reasoning → `claude-haiku-4-5` for lightweight tasks → `Qwen2.5:7b` local (free) for repetitive work.
- **Scope-Aware**: Agent reads `TARGETS.md` before every run. Never scans out-of-scope. Engagement rules are baked in, not bolted on.
- **Telegram Operator Alerts**: Phase summaries and immediate CRITICAL notifications pushed to your phone — under 300 chars, mobile-optimized.
- **Structured Reporting**: Auto-generated Markdown reports with findings table, severity flags, and recommendations.
- **Pro Plan Optimized**: Session-aware design respects the 5-hour Claude Pro limit. Batched commands, context compression after Phase 4, reference-not-repeat patterns throughout.
- **Resource Efficient**: Designed for a CPU-only VM with no GPU. 14GB RAM, 8 vCPU is enough to run the full stack.

## Architecture

```
Operator (pho5nix)
      │
      │ Telegram
      ▼
 Eagle-Eye Agent (agent-007)
 Ubuntu 24.04 LTS | OpenClaw
      │
      ├── Primary:  claude-sonnet-4-5  (Anthropic Pro)
      ├── Fallback: claude-haiku-4-5   (Anthropic, lightweight)
      └── Local:    Qwen2.5:7b-q4_K_M (Ollama, free, CPU-only)
      │
      ├── workspace/
      │   ├── AGENTS.md       ← Identity, workflow, scope rules
      │   ├── SOUL.md         ← Personality, output style, severity flags
      │   ├── TOOLS.md        ← Tool index, paths, phase-to-tool map
      │   ├── USER.md         ← Operator profile, preferences, plan limits
      │   ├── IDENTITY.md     ← Agent identity card
      │   ├── MEMORY.md       ← Session memory, install log, daily reset
      │   ├── HEARTBEAT.md    ← Disk/health monitoring rules
      │   ├── targets/
      │   │   └── TARGETS.md  ← Engagement scope (read first, always)
      │   ├── skills/
      │   │   └── pentest-recon/
      │   │       └── SKILL.md ← Full 9-phase pipeline with commands
      │   └── tools/           ← Per-tool decision trees (autonomous KB)
      │       ├── subfinder.md
      │       ├── amass.md
      │       ├── dnsx.md
      │       ├── httpx.md
      │       ├── naabu.md
      │       ├── nmap.md
      │       ├── katana.md
      │       ├── nuclei.md
      │       ├── gobuster.md
      │       ├── ffuf.md
      │       ├── nikto.md
      │       └── whatweb.md
      └── scans/              ← Scan output files
      └── reports/            ← Final engagement reports
```

## The 9-Phase Pipeline

| # | Phase | Primary Tools | Output |
|---|-------|--------------|--------|
| 1 | **OSINT & DNS Recon** | theHarvester, dnsrecon, shodan | Emails, ASN, IP ranges |
| 2 | **Subdomain Enumeration** | subfinder, amass, dnsx | Validated subdomain list |
| 3 | **Live Host Discovery** | httpx | Live hosts with status/title |
| 4 | **Port Scanning** | naabu, nmap | Open ports per host |
| 5 | **Service Fingerprinting** | nmap -sV, whatweb, sslscan | Tech stack, versions, TLS |
| 6 | **URL & Content Discovery** | katana, waybackurls, gau, gobuster, ffuf | Endpoints, params, paths |
| 7 | **Vulnerability Scanning** | nuclei | CVEs, misconfigs, exposures |
| 8 | **Report Generation** | — | Markdown report with findings |
| 9 | **Operator Notification** | Telegram | Final summary push |

## Decision Tree Knowledge Base

Eagle-Eye's most important feature isn't the tools — it's the knowledge base that governs how the agent uses them.

Each tool file in `workspace/tools/` is not a cheat sheet. It is a **decision tree**: default run → output interpretation → failure recovery → stealth variants → pipe targets. The agent reads the relevant file before executing or retrying a tool, which gives it the ability to reason autonomously:
This design means the agent recovers from common field conditions without paging the operator.

## Prerequisites

- **Proxmox VE** (or any hypervisor) — this guide targets Proxmox but the agent runs on any Ubuntu 24.04 host.
- **Ubuntu 24.04 LTS** VM: 16GB RAM, 8 vCPU, 60GB disk (agent-007).
- **OpenClaw** installed and configured.
- **Anthropic Claude Pro** plan (for claude-sonnet-4-5 primary model).
- **Ollama** (optional, for local Qwen2.5:7b fallback — separate VM or same host).
- **Telegram Bot** configured for operator notifications.
- All recon tools installed in PATH (see Tools section below).

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/pho5nix/Eagle-Eye.git
cd Eagle-Eye
```

### 2. Deploy Workspace Files

```bash
# Copy workspace to OpenClaw config directory
cp -r workspace/ ~/.openclaw/workspace/

# Set permissions
chmod 700 ~/.openclaw/workspace/
chmod 600 ~/.openclaw/workspace/tools/*.md
```

### 3. Configure openclaw.json

Place your `openclaw.json` at `~/.openclaw/openclaw.json`. Minimum required config:

```json
 "auth": {
    "profiles": {
      "anthropic:default": {
        "provider": "anthropic",
        "mode": "token"
      }
    }
  },
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://OLLAMA-HOST-IP:11434/v1",
        "apiKey": "ollama",
        "api": "openai",
        "models": [
          {
            "id": "qwen2.5:7b",
            "name": "Qwen 2.5 7B",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            }
           }
        ]
      }
    }
  },
  "agents": {
     "defaults": {
       "model": {
         "primary": "anthropic/claude-sonnet-4-5",
         "fallbacks": [
           "ollama/qwen2.5:7b"
         ]
       },
       "models": {
         "anthropic/claude-sonnet-4-5": {
           "alias": "sonnet"
         },
         "anthropic/claude-haiku-4-5": {
           "alias": "haiku"
         },
         "ollama/qwen2.5:7b": {
           "alias": "qwen"
         }
       }
     }
  },

```

> **Security**: `chmod 600 ~/.openclaw/openclaw.json` — this file contains your API keys and gateway token.

### 4. Install Recon Tools

### 5. Configure Targets

Edit `workspace/targets/TARGETS.md` before every engagement:

```markdown
## Active Engagement
- Client: Example Corp
- Type: External Black-box
- Authorization: Letter on file
- Date: 2026-02-21

## In-Scope
- example.com
- *.example.com
- 203.0.113.0/24

## Out of Scope
- mail.example.com
- third-party.example.com
```

The agent reads this file **first** before any phase executes.

### 6. Set Environment Variables

```bash
export TARGET="example.com"
export DATE=$(date +%Y-%m-%d)
export SCANDIR="~/.openclaw/workspace/scans"
export REPORTDIR="~/.openclaw/workspace/reports"
mkdir -p $SCANDIR $REPORTDIR
```

### 7. Optional — Local LLM (Ollama)

For a zero-cost fallback that handles repetitive tasks without hitting your Pro plan:

```bash
# On Ollama VM (separate Proxmox VM recommended)
curl -fsSL https://ollama.ai/install.sh | sh

# Pull and configure Qwen2.5 with extended context
ollama pull qwen2.5:7b
```

## Usage

### Starting a Recon Run

```bash
# Open OpenClaw and let the agent load its workspace
openclaw

# The agent will:
# 1. Read AGENTS.md, SOUL.md, TOOLS.md, USER.md (brain stem — always loaded)
# 2. Read TARGETS.md (scope verification — mandatory)
# 3. Execute Phase 1 through 9 autonomously
# 4. Send Telegram updates per phase
# 5. Generate report in workspace/reports/
```

### Operator Commands

```
/new       → Start fresh session (new engagement)
/reset     → Reset current session state
/compact   → Compress context (use after Phase 4 for long runs)
```

### Monitoring

The agent pushes updates to Telegram at each phase boundary. Format:

```
Eagle-Eye | Phase 2 Complete
Target: example.com
Subdomains: 147 found, 89 validated
Next: Live host probing
⏱ Runtime: 4m 32s
```

Critical findings trigger an immediate alert:

```
CRITICAL | example.com
nuclei: CVE-2024-XXXXX
Service: Apache 2.4.49
Path: /cgi-bin/
Action required
```

## Workspace File Reference

| File | Role | Loaded |
|------|------|--------|
| `AGENTS.md` | Identity, workflow, scope rules, output standards | Always |
| `SOUL.md` | Output style, severity system, communication tone | Always |
| `TOOLS.md` | Tool index, paths, phase mapping | Always |
| `USER.md` | Operator profile, Pro plan limits, preferences | Always |
| `IDENTITY.md` | Agent name, host, model, channel | Always |
| `MEMORY.md` | Session log, install record, daily reset time | Always |
| `HEARTBEAT.md` | Disk/health monitoring rules | Always |
| `targets/TARGETS.md` | Engagement scope — read before any phase | Per engagement |
| `skills/pentest-recon/SKILL.md` | Full 9-phase pipeline with commands | On skill trigger |
| `tools/{toolname}.md` | Per-tool decision trees | On demand by agent |

## Plan Session Strategy

Eagle-Eye is designed around the Claude Pro 5-hour session limit:

- **Before starting**: Agent asks all scope questions in a single message — no back-and-forth.
- **Context compression**: Agent runs `/compact` after Phase 4 if context exceeds ~50K tokens.
- **Batching**: All Phase 1 OSINT commands run in a single bash block, not one at a time.
- **Reference, don't repeat**: Agent references scan files by path rather than pasting output.
- **Scheduling**: Designed to complete Phases 1–4 in 30–60 minutes, leaving Phases 5–7 for the next session if needed.
- **Weekly tracking**: Monitor usage at Settings → Usage to avoid hitting weekly limits mid-engagement.

## Troubleshooting

**Agent reads wrong scope / forgets target**
→ Check `TARGETS.md` is correctly filled. Run `/reset` and start fresh.

**Nuclei finds too many false positives**
→ Add `-exclude-tags fuzz,dos,info` to the nuclei command in `SKILL.md`.

**Subfinder returns zero results**
→ Check API keys in `~/.config/subfinder/config.yaml`. Run `subfinder -ls` to list active sources.

**Ollama fallback not connecting**
→ Verify Ollama is running: `curl http://localhost:11434/api/tags`. Check firewall rules if on separate VM.

**Session limit hit mid-engagement**
→ Note the last completed phase. Start a new session, tell the agent which phase to resume from, and reference the existing scan files by path.

**OpenClaw gateway token compromised**
→ `openclaw doctor --generate-gateway-token`. Rotate immediately and `chmod 600 ~/.openclaw/openclaw.json`.

## License

MIT License. See [LICENSE](LICENSE) for details.

## Acknowledgments

- Built on [OpenClaw](https://openclaw.ai) — the agentic AI framework that makes autonomous tool orchestration possible.
- Powered by [Anthropic Claude](https://anthropic.com) (claude-sonnet-4-5) as the primary reasoning engine.
- Local intelligence via [Qwen2.5](https://qwen.readthedocs.io) running on [Ollama](https://ollama.ai).
- Recon toolkit by [ProjectDiscovery](https://projectdiscovery.io) — subfinder, httpx, nuclei, naabu, dnsx, katana, amass.
- Part of the **pho5nix** security toolkit ecosystem alongside [Red-Threat-Redemption](https://github.com/pho5nix/Red-Threat-Redemption) (open-source SIEM stack).

---
