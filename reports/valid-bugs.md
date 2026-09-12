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

- 3 lead(s) marked VALID at 2026-09-07 15:35:13 UTC
  - **Verdict: HOLD** — Misconfig is real and confirmed. Park until: (1) authenticated landing page is tested with valid PHPSESSID to confirm ACAO: * persists post-auth, and (2) JS bundle grep finds an AP
  - | Q2 Attacker reachable? | **PARTIALLY** — API is public/unauth; SID mintable via GET. But cross-tenant test requires 2 valid accounts (2FA-gated) |
  - **Verdict: HOLD** — High-confidence hypothesis with CRITICAL impact if proven, but completely unverified. Requires 2 valid test accounts for the cross-bind matrix (SID-B body + PHPSESSID-A cookie on /

- 1 lead(s) marked VALID at 2026-09-10 11:50:18 UTC
  - | 1 | Dual-backend session confusion (SID-only authz, decorative PHPSESSID) | **HOLD** | Architectural anomaly real, but needs 2 valid test accounts to prove cross-tenant access. HUMAN_ONLY. |

- 5 lead(s) marked VALID at 2026-09-10 18:57:58 UTC
  - | Q4 Provable non-invasively? | **PARTIAL** — Unauthenticated wildcard confirmed via passive header inspection. Authenticated wildcard **not yet proven** (needs valid PHPSESSID from login redirect) |
  - **Verdict: VALID**
  - | 1 | CORS wildcard (kundenkonto) | **HOLD** | Needs 1 read-only step: authenticated GET with valid PHPSESSID → check CORS headers |
  - | 2 | Dual-backend session confusion | **HOLD** | Needs 2 valid test accounts for cross-bind matrix |
  - | **7** | **GraphQL introspection (shop)** | **VALID** | **Report-ready. CVSS 5.3. Proof: single introspection query.** |

- 6 lead(s) marked VALID at 2026-09-10 21:27:27 UTC
  - | Q4 Provable non-invasively? | PARTIAL — unauth wildcard confirmed via passive header read. Authenticated wildcard **not proven** (needs valid PHPSESSID) |
  - | Q4 Provable non-invasively? | NO — requires 2 valid customer accounts to test cross-bind. Current evidence = only unauth error responses |
  - **Verdict: VALID**
  - | 1 | CORS wildcard (kundenkonto) | **HOLD** | Needs 1 read-only step: authenticated GET with valid PHPSESSID |
  - | 2 | Dual-backend session confusion | **HOLD** | Needs 2 valid test accounts |
  - | 3 | **GraphQL introspection (shop)** | **VALID** | **Report-ready. CVSS 5.3.** |

- 3 lead(s) marked VALID at 2026-09-10 23:24:02 UTC
  - | 1 | CORS wildcard (kundenkonto) | **HOLD** | Needs 1 read-only step: authenticated GET with valid PHPSESSID |
  - | 2 | Dual-backend session confusion | **HOLD** | Needs 2 valid test accounts |
  - | 3 | **GraphQL introspection (shop)** | **VALID** | **Report-ready. CVSS 5.3.** |

- 1 lead(s) marked VALID at 2026-09-11 06:37:16 UTC
  - Once you provide the leads, I'll validate each one systematically against Q1-Q7 and provide verdicts with the required details for valid findings.

- 4 lead(s) marked VALID at 2026-09-12 11:15:16 UTC
  - | Q4 Provable non-invasively? | **NO** — requires 2 valid test accounts to cross-bind SID+PHPSESSID; passive recon exhausted |
  - | Q4 Provable non-invasively? | **NO** — requires POST with valid SID to unconfirmed endpoints |
  - | Q4 Provable non-invasively? | **NO** — needs valid account + phone number to test |
  - | 7 | /call/initiate cross-tenant | **HOLD** | Needs valid account + phone number |
