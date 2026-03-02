# ResonantOS Alpha Code Review Findings

Date: 2026-03-02
Scope: Full-system read-only review
Status: Execution Locked (no code changes made for fixes)

## Critical Issues

1. Unauthenticated privileged APIs with open CORS and bind on `0.0.0.0` in `dashboard/server_v2.py`.
2. Missing cryptographic signature verification in wallet/protocol-sensitive endpoints (`mint-nft`, `daily-claim`, `agree-alpha`, `sign-license`, `sign-manifesto`, `grant-xp`, protocol purchase/content access).
3. Path traversal and arbitrary file read/write risk in R-Memory endpoints (`/api/r-memory/document`, lock/unlock routes, layer lock routes).
4. Sensitive data exposure from unauth endpoints (`/api/agents` workspace files, `/api/conversations`, `/api/config`).
5. Public LLM proxy abuse risk via `/api/widget/chat` and unauth chatbot admin CRUD.

## High Priority

1. Protocol purchase checks balance but does not debit/burn/escrow `$RES` (explicit TODO).
2. Chatbot DB schema bootstrap missing; no `CREATE TABLE` path found.
3. Frontend/backend API contract mismatch (frontend calls multiple routes with no backend implementation).
4. Invalid mainnet RPC URL typos (`https://api.mainnet-beta.com`) in multiple locations.
5. `shield/data_leak_scanner.py` uses wrong proto path (`logician/poc/...`) and has incomplete pushed-range scan logic.
6. `/api/agents/<id>/model` writes config schema that resolver does not read.
7. `extensions/r-awareness.js` path handling can escape `ssotRoot`.
8. `extensions/gateway-lifecycle` fails open on exception.

## Medium

1. Unbounded knowledge file upload into DB.
2. XSS risk from `innerHTML` interpolation in `dashboard/static/js/crypto-payment.js`.
3. DevNet-only policy is inconsistently enforced.
4. Linux incompatibility in `api_rmemory_open_log` (`osascript` path).
5. `/api/wallet/user` hardcodes `canClaim: True`.
6. `REPO_DIR` hardcoded to `~/resonantos-augmentor` may mismatch deployment.
7. Go mangle service global mutable `programInfo` race/nil-deref risk.

## Low / Suggestions

1. Broad exception swallowing reduces observability.
2. Config key mismatch (`maxDocs` vs `maxDocsPerTurn`) between installer and extension.
3. Minor dead/redundant logic paths.

## Additional Confirmed Mismatches

Frontend references missing backend routes, including:
- `/api/tasks`
- `/api/activity`
- `/api/chat/{id}`
- `/api/widget/generate`
- `/api/widget/init/{id}`
- `/widget/v/{version}/widget.min.js`
- chatbot AI-config/test endpoints

## Antigravity Plan State

- 15 locked execution tasks were prepared (T-001 to T-015).
- No implementation performed.
- Awaiting explicit command format: `GO <Task ID>`.
