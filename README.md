# RunCraft QA Agent

**Author:** Anshul Gupta  
**Version:** v1.0  
**Stack:** Claude AI · Playwright MCP · Azure DevOps MCP

An AI-powered QA test execution agent that runs Azure DevOps test cases against web applications using Playwright — with self-healing locators, smart script resolution, and fully automated result write-back to ADO.

---

## What It Does

- Executes ADO test cases via natural language session (no code required at runtime)
- Resolves and self-heals broken element locators automatically
- Writes execution results, tags, and scripts back to ADO in a single batched operation
- Supports three execution modes: Test Plan, User Story, Individual TCs
- Runs headless (CI/CD) or headed (debug/demo)

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Claude AI Agent                    │
│            (Session Orchestrator)                   │
└────────────┬────────────────────┬───────────────────┘
             │                    │
    ┌────────▼────────┐  ┌───────▼────────┐
    │  Playwright MCP │  │  Azure DevOps  │
    │  (Browser)      │  │  MCP (ADO)     │
    └────────┬────────┘  └───────┬────────┘
             │                    │
    ┌────────▼────────┐  ┌───────▼────────┐
    │  Web App Under  │  │ Locator        │
    │  Test           │  │ Registry (ADO) │
    └─────────────────┘  └────────────────┘
```

**Locator Resolution Pipeline**
```
Level 1 → ADO Locator Registry    (primary — live, auto-updated)
Level 2 → Session Cache           (inherited within session)
Level 3 → Self-Heal DOM Scan      (max 1 scan per element per session)
Level 4 → Hard FAIL               (screenshot + continue to next TC)
```

**Script Resolution Pipeline (per TC)**
```
Tier 1  → Existing exec-script in TC Description
Tier 2  → Canonical TC override (manual)
Tier 2b → Canonical TC (auto-registered)
Tier 3  → Last successful TC script this session (same module)
Tier 4  → Build from registry + TC step text (cold start)
```

---

## Execution Modes

| Mode | Input Required |
|------|----------------|
| Test Plan | Plan ID + Suite ID |
| User Story | User Story Work Item ID |
| Individual TCs | Up to 3 TC IDs |

---

## Session Flow

```
1. Load locator registry from ADO
2. Collect credentials from user
3. Fetch test cases → show summary
4. Login via Playwright
5. Execute TCs sequentially
6. Batched write-back to ADO:
   → Mark unexecuted TCs as SKIPPED
   → Write self-healed locators to registry
   → Register new canonical TCs
   → Write execution scripts to TC Description
   → Write results + tags to all TCs
   → Render session summary
```

---

## Agent File Roles

| File | Role | Load When |
|------|------|-----------|
| `system_prompt.md` | Agent identity, session rules, hard constraints | Every session |
| `execute_flow.js` | Complete execution logic — all modes, all pipelines | Mode selected |
| `navigation_utils.js` | Playwright helpers — login, fill, navigate, self-heal, registry I/O | First browser action |
| `button_check.js` | Pre-click guard — returns READY / DISABLED / HIDDEN / NOT_FOUND | Every significant click |
| `toast_observer.js` | MutationObserver + Fetch/XHR intercept for toast/API result capture | Button clicks with async response |
| `sprint.conf` | Environment config — updated once per sprint | Session start |
| `locators.json` | Fallback locator registry — used only when ADO registry unreachable | ADO registry failure |
| `executor_templates.json` | Output format templates — ADO History HTML, session summary | First TC execution |
| `executor_field_defaults.json` | ADO field patch defaults — op/path/isFormatted rules | First ADO write |

> Files are loaded as Claude Project Knowledge. None are executed directly — the agent reads and applies them at runtime.

---

## Setup

### 1. Prerequisites

**Python**
```cmd
python --version

:: If not installed: https://www.python.org/downloads/windows/
:: After install, open a new CMD and verify:
py --version
```

**Playwright**
```cmd
:: Python
py -m pip install playwright

:: Node.js
npm init playwright@latest
npx playwright test
```

### 2. Claude Desktop

- Install [Claude Desktop](https://claude.ai/download)
- Configure MCP servers: **azure-devops** and **playwright** (headless + headed)
- Set your ADO org URL and PAT as environment variables

### 3. Claude Project

- Create a new Claude Project named `RunCraft QA Agent`
- Paste the contents of `PROJECT_INSTRUCTION.md` into the Project Instructions field
- Upload all agent files as Project Knowledge

### 4. Run

Open the Claude Project and start a conversation. The agent will guide you through mode selection, credential collection, TC fetch, and execution.

---

## Key Design Decisions

- **Single batch write-back** — all ADO updates happen once at session end, never per-TC
- **Registry-first locators** — selectors live in ADO, not in code, so they survive UI changes
- **Self-heal closes the loop** — broken selectors are fixed at runtime and written back automatically
- **Module isolation** — scripts never inherit across modules; canonical TC per module ensures consistency
- **`isFormatted: false`** — mandatory on all ADO Description and Steps writes to prevent silent data loss

---

## Changelog

### v1.0
- Three execution modes: Test Plan · User Story · Individual TCs
- 4-level locator pipeline with self-healing
- 4-tier script resolution with canonical TC auto-registration
- Batched session-end ADO write-back
- Step trace log on FAIL (stepsPassedLog)
- SKIPPED status for unexecuted TCs
- AI Execution Pass / Fail tag swap on every run
- Record creation naming convention for agent-generated records

---

*Built with Claude AI · Playwright MCP · Azure DevOps MCP*
