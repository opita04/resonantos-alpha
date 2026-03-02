# System Changes Log

This file tracks modifications made to the ResonantOS system architecture, configurations, and core dashboard logic.

## [2026-03-01]

### Dashboard Fixes
- **Agent Workspace Discovery**: Modified `dashboard/server_v2.py` to include `OPENCLAW_HOME/agents/[agent_id]` in the search path for workspace files like `USER.md`.
- **Unicode Support Fix**: Replaced Unicode emojis in `dashboard/server_v2.py` print statements with ASCII text (`[START]`, `[OK]`, `[WARN]`) to prevent `UnicodeEncodeError` on Windows consoles.
  - **Reason**: The dashboard was previously restricted to `workspace-[agent_id]` folders, causing agents using the standard `agents/` structure to display empty templates instead of personalized data.
  - **Effect**: Agent details in the dashboard now correctly reflect personalized `USER.md` and `IDENTITY.md` contents.

- **UTF-8 Encoding for File Operations**: Enforced `encoding="utf-8"` across all `read_text()` and `write_text()` operations in `dashboard/server_v2.py`.
  - **Reason**: On Windows systems, Python's default encoding (typically CP1252) misinterprets UTF-8 emoji bytes as garbage characters (e.g., `ðŸ""`).
  - **Effect**: Emojis in `IDENTITY.md`, `USER.md`, and other workspace files now display correctly in the dashboard across all agents.

## [2026-03-02]

### R-Memory — Key Resolution & Windows Compatibility
- **Category**: Core Logic / Configuration
- **Files Modified**:
  - `extensions/r-memory.js` (project source)
  - `~/.openclaw/extensions/r-memory/index.js` (global extension install)
  - `~/.openclaw/agents/main/agent/extensions/r-memory.js` (agent-level copy — **this is the one OpenClaw actually loads at runtime**)
  - `dashboard/server_v2.py`

- **Key Resolution Fix**: Rewrote `resolveApiKeyForProvider` in both `r-memory.js` and `index.js` to search `openclaw.json` → `auth-profiles.json` → `credentials/` directory in order. Skips OAuth placeholder values (`minimax-oauth`, `openai-oauth`) to fall through to actual tokens.
  - **Reason**: The function was returning `minimax-oauth` (a placeholder in `openclaw.json`) instead of the real token from `auth-profiles.json`, causing every narrative tracker call to fail with `"apiKey is not defined"`.
  - **Effect**: R-Memory can now authenticate with Minimax and all other configured providers.

- **Windows Path Fixes**: Replaced all bare `process.env.HOME` references with `process.env.USERPROFILE || process.env.HOME || ""` in `resolveApiKeyForProvider`, `discoverProviders`, and the `init()` function.
  - **Reason**: `process.env.HOME` is undefined on Windows, causing path resolution to fail silently.
  - **Effect**: R-Memory initializes correctly on Windows systems.

- **Syntax Error Fix** (`index.js`): Removed a literal `\n` character on line 1539 that was concatenating two statements into a single broken line.
  - **Reason**: This caused a JavaScript syntax error that prevented `updateNarrativeThread` from being called as a separate statement.
  - **Effect**: Narrative tracker now fires correctly after each `agent_end` event.

- **Open Log Windows Support** (`server_v2.py`): Updated `/api/r-memory/open-log` endpoint to detect `platform.system() == "Windows"` and spawn a PowerShell window with `Get-Content -Wait` instead of using macOS `osascript`.
  - **Reason**: The original implementation used macOS-only `osascript` commands.
  - **Effect**: "Open Log" button in the dashboard now works on Windows.

- **Version Bump**: Updated init log string from `R-Memory V4.8.1` to `R-Memory V5.0.1` to match the actual codebase version header.

- **Model Object Schema Fix** (`buildModelObject` fallback): When `getModel()` throws (provider not registered in pi-ai), the fallback model object was missing required fields. Aligned with the exact schema `getModel` returns: added `name`, `api`, `baseUrl`, `maxTokens`, `reasoning`, `input`, `cost`. Values are read from `openclaw.json` provider config.
  - **Reason**: First caused `"No API provider registered for api: undefined"` (missing `api` field), then `"Cannot read properties of undefined (reading 'includes')"` (missing `baseUrl`/`name` fields that OpenClaw's internal routing checks).
  - **Effect**: Minimax-portal correctly routes via `anthropic-messages` protocol to `https://api.minimax.io/anthropic`.

### Dashboard — TODO Modal Fix
- **Category**: UI Bug Fix
- **File Modified**: `dashboard/templates/todo.html`
- **CSS Class Collision**: Renamed the TODO modal's inner card from `class="modal"` to `class="todo-modal-content"`, and updated all corresponding CSS selectors (`.modal h3`, `.modal label`, `.modal input`, `.modal select`, `.modal textarea`).
  - **Reason**: The global `dashboard.css` defines `.modal { position: fixed; opacity: 0; visibility: hidden; }` for a completely different modal pattern. The TODO template's inner `<div class="modal">` inherited those rules, making the modal card invisible while the dark overlay (`.modal-overlay.active`) displayed correctly — resulting in a darkened screen with no visible form.
  - **Effect**: Clicking "+ New TODO" now correctly displays the add/edit modal form.

### R-Memory — ParseError Fix (index.js Corruption)
- **Category**: Plugin Load Error
- **File Modified**: `~/.openclaw/extensions/r-memory/index.js`
- **Root Cause**: Line 1 of the deployed `index.js` had a corrupted prefix `var apiKey = null; \n` prepended before the module's `/**` comment block. The literal `\n` was parsed by the JavaScript engine as a malformed Unicode escape sequence (`\uXXXX`), causing `ParseError: Expecting Unicode escape sequence`.
- **Fix**: Replaced the corrupted `index.js` with the clean source from `~/.openclaw/agents/main/agent/extensions/r-memory.js`, then restored three Windows-specific patches:
  1. Windows npm path for `@mariozechner/pi-ai` dependency resolution
  2. Minimax-portal provider handling + API key injection in `buildModelObject`
  3. `narrativeModel` default set to `minimax-portal/MiniMax-M2.1`
- **Effect**: R-Memory plugin loads without errors.

### Dashboard — R-Memory Stat Fixes
- **Category**: UI Bug Fix
- **File Modified**: `dashboard/templates/index.html`, `dashboard/server_v2.py`
- **Dashboard Display Issues**:
  - `null msgs` display: Removed the literal "null msgs" text from the compression progress bar. The backend no longer counts message history in `conversationEstimate`, leaving `msgCount` as `None`, which caused a string concatenation bug (`${est.msgCount} msgs`). Now uses a conditional render.
  - Hardcoded version: Updated `server_v2.py` to display "V5.0.1 running" instead of the old "V4.6.1" label.
  - Architecture note: R-Memory V5.0.1 uses *on-demand* compression. It does not compress when context passes the R-Memory trigger (e.g., 36k), but waits until OpenClaw fires `session_before_compact` at its own threshold (near the model's 200k max limit), at which point R-Memory compresses all queued blocks down to 36k at once.
  - "Stuck" yellow bar fix: The "Conversation → Compression" UI progress bar was originally calculating its percentage based on the target (36k), causing it to prematurely hit 100% and appear stuck for a long time. It has been updated to calculate against the 200k context limit (when OpenClaw actually fires the trigger), and the label changed to "Context Filling (will compress down to 36k at 200k)" for clarity.

## [2026-03-01]
### R-Awareness Configuration fix
- **Category**: Bug Fix
- **File Modified**: `~/.openclaw/workspace/r-awareness/config.json`
- **Description**: Updated `ssotRoot` from relative path `resonantos-augmentor/ssot` to absolute path `C:\\AI\\Openclaw-Projects\\resonantos-alpha\\ssot`.
- **Reason**: The relative path did not exist and caused R-Awareness to constantly error out with `ssotRoot directory not found` and fail to load documents.
- **Effect**: R-Awareness can successfully find the local SSoT directory and inject documents correctly.

### Dashboard SSoT Root Fix
- **Category**: Bug Fix
- **File Modified**: `dashboard/server_v2.py`
- **Description**: Updated the hardcoded `SSOT_ROOT` from `WORKSPACE / "resonantos-augmentor" / "ssot"` to point to the correct local repository path `C:\AI\Openclaw-Projects\resonantos-alpha\ssot`.
- **Reason**: The dashboard was looking for the SSoT files in the wrong directory, preventing it from displaying SSoT files accurately in the UI. 
- **Effect**: Server now points to the actual SSoT library. Server restarted to apply changes.
## [2026-03-02]
### Code Review Security & Bug Fixes
- **Category**: Security / Bug Fix
- **Description**: Addressed critical and high-priority code review findings across the system.
  - **Path Traversal**: Fixed dashboard/server_v2.py and extensions/r-awareness.js to strictly validate paths inside workspace directories using os.path and path.resolve.
  - **Database Bootstrap**: Added CREATE TABLE IF NOT EXISTS for chatbot schemas in dashboard/server_v2.py.
  - **Fail Closed**: Updated gateway-lifecycle.js guard to block operations on error instead of failing open.
  - **Unbounded Uploads**: Implemented 5MB file size and 20 document limits in chatbot knowledge uploads /api/chatbots/<bot_id>/knowledge.
  - **Cross-Site Scripting (XSS)**: Stripped innerHTML vulnerabilities in crypto-payment.js by adding an escapeHTML wrapper for injected values.
  - **DevNet Enforcement**: Added DevNet fallbacks and warnings to symbiotic_client.py and wallet transfer routes to strictly enforce DevNet-only interactions.
  - **Config Parsing Error**: Renamed installer config key from maxDocs to maxDocsPerTurn so the R-Awareness extension can read limits correctly.
  - **Hardcoded Paths**: Replaced esonantos-augmentor hardcoded strings across the repository with dynamic path references (Path(__file__).resolve().parent.parent).
  - **Go Mangle Service Race**: Refactored service.go to remove a global mutable programInfo variable, scoping it to a struct and wrapping accesses with mutex locks.
- **Reason**: Recommended by the code review in CODE_REVIEW_FINDINGS.md to secure backend endpoints, correct misconfigurations, and prevent system compromise.
- **Effect**: Enhanced security against path traversals and XSS. Better DevNet safety rails and resolved several operational errors including broken configurations and Linux compatibility logic.
