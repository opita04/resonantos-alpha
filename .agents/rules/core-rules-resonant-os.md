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