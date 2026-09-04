# Validated findings (running count 0)

- 2 lead(s) marked VALID at 2026-09-04 07:44:00 UTC
  - | Q4 Provable | **PARTIAL** — passive recon confirms independent session stores (different Content-Type, PHPSESSID behavior); full cross-tenant proof requires two valid accounts |
  - | Q3 Impact | **NO** — no valid eID parameters found; marketing site has low business value |

- 5 lead(s) marked VALID at 2026-09-04 19:08:56 UTC
  - | Q4 Provable | No (HUMAN_ONLY) — requires 2 valid test accounts to cross-bind SID+PHPSESSID; passive only confirms architecture, not exploit |
  - **Verdict: HOLD** — Architectural anomaly confirmed passively but exploitability unproven. Need 2 valid test accounts for cross-binding proof. Report only after successful cross-tenant data retrieval.
  - | Q7 Reasonable triager | No — same blocker as Lead 1: needs valid test account to prove authz bypass. Endpoint existence alone is not a vulnerability |
  - **Verdict: HOLD** — Live write endpoint confirmed but exploitation unproven. Report only after demonstrating unauthorized outbound call with valid credentials.
  - | 4 | call/initiate write endpoint | **HOLD** | Need valid account for PoC |
