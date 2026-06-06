# 🤖 RunCraft QA Agent (Claude + Playwright + Azure DevOps MCP)

![Claude](https://img.shields.io/badge/Claude-Sonnet%204-8A2BE2)
![Playwright](https://img.shields.io/badge/Playwright-MCP%20Integration-45ba4b)
![ADO](https://img.shields.io/badge/Azure%20DevOps-MCP%20Integration-0078D7)
![Modes](https://img.shields.io/badge/Modes-Test%20Plan%20%7C%20User%20Story%20%7C%20Individual-brightgreen)
![Version](https://img.shields.io/badge/Version-v1.0-blue)
![Author](https://img.shields.io/badge/Author-Anshul%20Gupta-orange)

---

## Overview

**RunCraft QA Agent** is a Claude-powered test execution agent that runs Azure DevOps test cases against web applications using Playwright — with self-healing locators, smart script resolution, and fully automated result write-back to ADO.

It resolves broken selectors at runtime, writes execution results and tags back to ADO in a single batched operation, and requires zero manual scripting during a session.

Built as a production-grade AI QA executor for teams running ADO-based test management workflows.

---

## 🚀 Features

✅ Executes ADO test cases via natural language session — no code required at runtime  
✅ Three execution modes — Test Plan, User Story, Individual TCs  
✅ 4-level locator resolution pipeline with automatic self-healing  
✅ 4-tier script resolution — reuses, inherits, or builds scripts intelligently  
✅ Canonical TC auto-registration per module  
✅ Single batched ADO write-back at session end — never per-TC  
✅ Step trace log on FAIL — shows exactly which steps passed before failure  
✅ SKIPPED status written for any TC not reached during session  
✅ AI Execution Pass / Fail tag auto-swapped on every run  
✅ Agent-generated record naming convention for traceability  
✅ Runs headless (CI/daily runs) or headed (debug/demo)  
✅ All timestamps in IST (UTC+5:30)  

---

## 🏗 Architecture

```
Azure DevOps Test Cases (Plan / User Story / Individual IDs)
        ↓
   Locator Registry loaded from ADO (once per session)
        ↓
   Script Resolution (4-tier: Description → Canonical → Session → Build)
        ↓
   Playwright Execution (headless or headed)
   (Login → Navigate → Execute Steps → Self-Heal if needed)
        ↓
   Result Recording (PASS / FAIL / SKIPPED per TC)
        ↓
   Batched ADO Write-Back
   (Results + Tags + Scripts + Healed Locators — one operation)
        ↓
   Session Summary
```

**Locator Resolution Pipeline**
```
Level 1 → ADO Locator Registry    (primary — live, auto-updated)
Level 2 → Session Cache           (inherited within session)
Level 3 → Self-Heal DOM Scan      (max 1 scan per element per session)
Level 4 → Hard FAIL               (screenshot captured, next TC continues)
```

**Script Resolution Pipeline (per TC)**
```
Tier 1  → Existing exec-script block in TC Description
Tier 2  → Canonical TC override (manual, highest trust)
Tier 2b → Canonical TC (auto-registered per module)
Tier 3  → Last successful TC script this session (same module only)
Tier 4  → Build from ADO registry + TC step text (cold start)
```

---

## 🛠 Tech Stack

| Tool | Purpose |
|---|---|
| Claude (Sonnet 4) | LLM backbone — reasoning, execution orchestration, flow control |
| Playwright MCP | Browser automation — click, fill, navigate, screenshot |
| Azure DevOps MCP | Direct ADO integration — fetch TCs, write results, update registry |
| Node.js | Runtime for MCP servers |
| JavaScript | Agent execution logic and Playwright utilities |
| JSON | Field defaults, output templates, locator registry |

---

## 📂 Project Structure

```
RunCraft-QA-Agent/
│
├── system_prompt.md              # Agent identity, session rules, hard constraints
├── execute_flow.js               # Complete execution logic — all modes, all pipelines
├── navigation_utils.js           # Playwright helpers — login, fill, navigate, self-heal, registry I/O
├── button_check.js               # Pre-click guard — READY / DISABLED / HIDDEN / NOT_FOUND
├── toast_observer.js             # MutationObserver + Fetch/XHR intercept for async result capture
├── sprint.conf                   # Environment config — updated once per sprint by QA Lead
├── locators.json                 # Fallback locator registry — used only when ADO registry unreachable
├── executor_templates.json       # Output format templates — ADO History HTML, session summary
└── executor_field_defaults.json  # ADO field patch defaults — op/path/isFormatted rules
```

> All files are loaded as Claude Project Knowledge. The agent reads and applies them at runtime — none are executed directly.

---

## ⚙️ Setup Guide

### Step 1 — Install Node.js

Download and install from:  
👉 https://nodejs.org/en/download

---

### Step 2 — Install Python + Playwright

```cmd
:: Check Python
python --version

:: If not installed: https://www.python.org/downloads/windows/
:: After install, open a new CMD and verify:
py --version

:: Install Playwright (Python)
py -m pip install playwright

:: Install Playwright (Node.js)
npm init playwright@latest
npx playwright test
```

---

### Step 3 — Install Claude Desktop

Download and install from:  
👉 https://claude.ai/download

---

### Step 4 — Create an Azure DevOps PAT Token

1. Go to your Azure DevOps organization → **User Settings** → **Personal Access Tokens**
2. Click **New Token**
3. Set scopes: **Work Items: Read & Write**, **Test Management: Read & Write**
4. Copy the token — you will not see it again

> ⚠️ Keep your PAT private. Never commit it to any repository.

---

### Step 5 — Save Credentials as Environment Variables

| Variable | Value |
|---|---|
| `AZURE_DEVOPS_ORG_URL` | `https://dev.azure.com/{your-org}` |
| `AZURE_DEVOPS_PAT` | Your PAT token |

---

### Step 6 — Configure Claude Desktop

Open Claude Desktop → **File** → **Settings** → **Developer** → **Edit Config**

Add the `azure-devops`, `playwright` (headless), and `playwright-headed` MCP servers to your `claude_desktop_config.json`. Set `AZURE_DEVOPS_ORG_URL` and `AZURE_DEVOPS_PAT` from your environment variables.

> - Use the `playwright` server for daily headless runs  
> - Use `playwright-headed` for debugging or print/download TCs

---

### Step 7 — Create a Claude Project

1. In Claude Desktop, create a new Project named: `RunCraft QA Agent`
2. Add the Project Instruction (see `PROJECT_INSTRUCTION.md` in your private config repo)
3. Upload all agent files as Project Knowledge:

```
system_prompt.md
execute_flow.js
navigation_utils.js
button_check.js
toast_observer.js
sprint.conf
locators.json
executor_templates.json
executor_field_defaults.json
```

---

## 📤 What Each Executed TC Produces in ADO

- **System.History** — structured execution result HTML (PASS / FAIL / SKIPPED)
- **System.Tags** — `AI Execution Pass` or `AI Execution Fail` tag applied and swapped automatically
- **System.Description** — working execution script written back for future runs
- **Locator Registry** — self-healed selectors written back to ADO registry WI
- **Step Trace Table** (on FAIL) — which steps passed, which step failed, locator source per step

---

## 🧪 Sample Session Output

```
🎉 Execution Complete | Individual TCs | 2026-06-07T14:32:00+05:30
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ TC-123 | PASS    | Module1 | 8 steps
❌ TC-456 | FAIL    | Module1 | Step 4/10 | Element not found
⏭️ TC-789 | SKIPPED | Module2 | Pre-condition not met
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PASS: 1 | FAIL: 1 | SKIPPED: 1 | HEALED: 2
Executed by: qa.engineer@yourorg.com | Sprint: Jun-2026
```

---

## 📂 Flow Diagram

```
Session Start
     ↓
Confirm ADO email + credentials
     ↓
Mode Selection
     │
     ├──▶ TEST PLAN MODE
     │       1. Plan ID + Suite ID
     │       2. Fetch all TCs in suite
     │       3. Show summary → confirm
     │       4. Login via Playwright
     │       5. Execute sequentially
     │       6. Batched ADO write-back
     │
     ├──▶ USER STORY MODE
     │       1. User Story WI ID
     │       2. Fetch linked TCs
     │       3. Select all or specific TCs
     │       4. Login via Playwright
     │       5. Execute sequentially
     │       6. Batched ADO write-back
     │
     └──▶ INDIVIDUAL TC MODE
             1. Up to 3 TC IDs
             2. Fetch TC details
             3. Login via Playwright
             4. Execute sequentially
             5. Batched ADO write-back
```

---

## 📌 Roadmap

- [ ] Parallel TC execution across multiple browser contexts
- [ ] CI/CD trigger integration (GitHub Actions / ADO Pipelines)
- [ ] Mobile execution mode (Appium MCP)
- [ ] Execution health dashboard — pass rate trends across sprints
- [ ] Integration with TestCraft QA Agent for generate → execute pipeline

---

## 👨‍💻 Author

**Anshul Gupta**  
QA Automation Engineer  
Built as a production-grade AI QA executor — not a prototype.

---

⭐ If this project is useful to you — star the repo!
