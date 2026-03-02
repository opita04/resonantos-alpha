---
trigger: always_on
---

## Core Behavioral Rules (Agent Protocols)

### 1. Persistent Server Management
- **Rule**: Whenever a code change is made to the dashboard (`dashboard/`) or backend services, the agent must **automatically restart** the service without being prompted.
- **Verification**: Check if port 19100 is in use, kill the existing process, and respawn the server.

### 2. System Changes Logging
- **Rule**: Every time a modification is made to the system (architecture, configuration, or core logic), the agent MUST append a record of the change to [SYSTEM_CHANGES.md](file:///c:/AI/Openclaw-Projects/resonantos-alpha/SYSTEM_CHANGES.md).
- **Protocol**: Include the date, the category of change, a brief description of what was changed, the reason, and the effect.

### 3. Automatic GitHub Synchronization
- **Rule**: After every 5 entries added to `SYSTEM_CHANGES.md`, the agent MUST automatically commit and push all local changes to the GitHub repository [opita04/resonantos-alpha](https://github.com/opita04/resonantos-alpha).
- **Protocol**:
  1. Count the number of entries in `SYSTEM_CHANGES.md` (each header like `## [YYYY-MM-DD]` or individual bullet point counts toward the logic).
  2. Perform a `git pull origin main` to ensure sync.
  3. Commit with a descriptive message referencing the recent changes.
  4. Push to `main`.