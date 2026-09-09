## 2026-09-03 16:40:14 UTC [target] (model bigpickle)
[NEW] kundenkonto.fonial.de — Customer portal + API host (Cloudflare-fronted, version v2026.09.01-1)
[NEW] kundenkonto.fonial.de/api/2.0 — Live REST API (POST-only, JSON body, session-based auth)
[NEW] CORS wildcard `access-control-allow-origin: *` on ALL API endpoints including /session/authenticate
[NEW] Two backend systems: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID)
[NEW] www.fonial.de/shop/ — Hardware e-commerce (Magento-like, redirects from fonial.de/shop)
[NEW] www.fonial.de/hilfe/ — Help center (separate PHP app)
[NEW] www.fonial.de — TYPO3 CMS, PHP/8.3.3, nginx/1.31.2
[CHANGED] fonial.de → 301 to www.fonial.de (was previously unresolved; nginx/1.31.2 confirmed)
[CHANGED] app.fonial.de — Transport error (DNS/SSL unreachable)
[CHANGED] admin.fonial.de — Transport error (DNS/SSL unreachable)
[CHANGED] staging.fonial.de — Timeout (unreachable)
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] API session auth lacks brute-force protection enabling credential stuffing
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/session/authenticate
confidence: 72
reasoning: Auth endpoint returns `{"authenticated":false}` with no delay, lockout, or CAPTCHA after repeated failures. API docs reveal username/password auth with only email-PIN 2FA as secondary. No rate limiting observed across 5+ rapid requests. SIDs are not invalidated on failed auth. Two distinct backends (session vs data) suggest potential session-binding inconsistency.
evidence_needed: 1) Confirm no rate limiting under sustained load (20+ requests), 2) Verify 2FA PIN cannot be bypassed or brute-forced, 3) Check if session expires after N failed attempts
verify_steps: 1) POST /session to get SID, 2) Send 20 rapid POST /session/authenticate with varying passwords, 3) Monitor for 429/lockout/delay, 4) Check if SID remains valid after failures
impact: Credential stuffing against customer accounts. API exposes call records (CDR), phone numbers, device lists, and can initiate outbound calls. Severity: HIGH
testability: AUTH_HELPED
[HYP] Dual-session architecture allows cross-session confusion
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 58
reasoning: Two distinct backend systems detected: session/auth endpoints return text/json (no PHPSESSID cookie), while data endpoints (devices/get, evn/get, call/initiate) return text/json;charset=UTF-8 and set PHPSESSID with secure;HttpOnly. This suggests the auth layer and data layer use independent session stores. The SID from the session endpoint may not be properly validated against the PHPSESSID on data endpoints, potentially allowing session confusion or state desynchronization.
evidence_needed: 1) Authenticate a session, 2) Make data requests and compare SID binding vs PHPSESSID binding, 3) Test if a valid SID without matching PHPSESSID is accepted on data endpoints
verify_steps: 1) POST /session → get SID1, 2) POST /session/authenticate with valid creds → SID1 authenticated, 3) POST /devices/get with SID1 (capture PHPSESSID), 4) From different browser session, POST /devices/get with SID1 but no PHPSESSID, 5) Check if data is returned
impact: If data endpoints accept SID without PHPSESSID validation, attacker with stolen SID can access full account data without triggering PHP session tracking. Severity: HIGH
testability: AUTH_HELPED
[HYP] API exposes cross-tenant data via session enumeration
class: IDOR
asset: kundenkonto.fonial.de/api/2.0/evn/get
confidence: 45
reasoning: API uses UUID v4 SIDs (good randomness). However, the dual-backend architecture (session endpoint independent of PHP data endpoints) raises the question of whether the PHPSESSID on data endpoints properly scopes data to the authenticated SID. If the PHP session doesn't tightly bind to the SID, session fixation or swapping could expose cross-tenant CDR/phone data.
evidence_needed: 1) Authenticate two different accounts, 2) Verify each gets unique PHPSESSID, 3) Confirm that SID-to-PHPSESSID binding is enforced on data endpoints
verify_steps: 1) Account A: POST /session + authenticate, 2) Extract PHPSESSID from /devices/get response, 3) Account B: same flow, extract different PHPSESSID, 4) Try Account B's SID with Account A's PHPSESSID on /evn/get
impact: Cross-tenant call records (CDR), phone number, and device data exposure. Severity: CRITICAL
testability: AUTH_HELPED
[PARKED] API CORS wildcard: `access-control-allow-origin: *` without `access-control-allow-credentials: true` means browsers won't send cookies cross-origin. SID is in request body, not cookies. Low direct exploitability. Park until chained with another finding (e.g., SID leak via referer).
[FINAL] Survivors ranked:
[NEXT] PROBE: POST https://kundenkonto.fonial.de/api/2.0/session/authenticate — send 20 rapid requests with different passwords to the same SID to confirm no rate limiting or lockout. Measure response times and check for 429/lockout. This is passive rate-limit testing only.
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface.
[RISK] fonial: 40 — Cloudflare-fronted, rate limiting may exist at edge. German company with EnBW parent (enterprise-grade compliance). 2FA on login. However, API lacks visible rate limiting and dual-backend architecture is unusual.
## 2026-09-03 19:26:37 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] API session auth lacks brute-force protection enabling credential stuffing
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/session/authenticate
confidence: 72
reasoning: Auth endpoint returns `{"authenticated":false}` with no delay, lockout, or CAPTCHA after repeated failures. API docs reveal username/password auth with only email-PIN 2FA as secondary. No rate limiting observed across 5+ rapid requests. SIDs are not invalidated on failed auth. Two distinct backends (session vs data) suggest potential session-binding inconsistency.
evidence_needed: 1) Confirm no rate limiting under sustained load (20+ requests), 2) Verify 2FA PIN cannot be bypassed or brute-forced, 3) Check if session expires after N failed attempts
verify_steps: 1) POST /session to get SID, 2) Send 20 rapid POST /session/authenticate with varying passwords, 3) Monitor for 429/lockout/delay, 4) Check if SID remains valid after failures
impact: Credential stuffing against customer accounts. API exposes call records (CDR), phone numbers, device lists, and can initiate outbound calls. Severity: HIGH
testability: AUTH_HELPED
[FINAL] API session auth lacks brute-force protection enabling credential stuffing: confidence 72, above 40, has verify_steps. Keep.
[NEXT] PROBE: GET https://kundenkonto.fonial.de/api/2.0/ to check for error messages or documentation links.
[RISK] fonial: 40 — Cloudflare-fronted, rate limiting may exist at edge. German company with EnBW parent (enterprise-grade compliance). 2FA on login. However, API lacks visible rate limiting and dual-backend architecture is unusual.
## 2026-09-03 21:55:00 UTC [target] (model bigpickle)
[NEW] Two backend systems: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID)
[NEW] www.fonial.de/shop/ — Hardware e-commerce (Magento-like, redirects from fonial.de/shop)
[NEW] www.fonial.de/hilfe/ — Help center (separate PHP app)
[NEW] www.fonial.de — TYPO3 CMS, PHP/8.3.3, nginx/1.31.2
[CHANGED] fonial.de → 301 to www.fonial.de (was previously unresolved; nginx/1.31.2 confirmed)
[CHANGED] app.fonial.de — Transport error (DNS/SSL unreachable)
[CHANGED] admin.fonial.de — Transport error (DNS/SSL unreachable)
[CHANGED] staging.fonial.de — Timeout (unreachable)
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] API session auth lacks brute-force protection enabling credential stuffing
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/session/authenticate
confidence: 72
reasoning: Auth endpoint returns `{"authenticated":false}` with no delay, lockout, or CAPTCHA after repeated failures. API docs reveal username/password auth with only email-PIN 2FA as secondary. No rate limiting observed across 5+ rapid requests. SIDs are not invalidated on failed auth. Two distinct backends (session vs data) suggest potential session-binding inconsistency.
evidence_needed: 1) Confirm no rate limiting under sustained load (20+ requests), 2) Verify 2FA PIN cannot be bypassed or brute-forced, 3) Check if session expires after N failed attempts
verify_steps: 1) POST /session to get SID, 2) Send 20 rapid POST /session/authenticate with varying passwords, 3) Monitor for 429/lockout/delay, 4) Check if SID remains valid after failures
impact: Credential stuffing against customer accounts. API exposes call records (CDR), phone numbers, device lists, and can initiate outbound calls. Severity: HIGH
testability: AUTH_HELPED
[HYP] Dual-session architecture allows cross-session confusion
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 58
reasoning: Two distinct backend systems detected: session/auth endpoints return text/json (no PHPSESSID cookie), while data endpoints (devices/get, evn/get, call/initiate) return text/json;charset=UTF-8 and set PHPSESSID with secure;HttpOnly. This suggests the auth layer and data layer use independent session stores. The SID from the session endpoint may not be properly validated against the PHPSESSID on data endpoints, potentially allowing session confusion or state desynchronization.
evidence_needed: 1) Authenticate a session, 2) Make data requests and compare SID binding vs PHPSESSID binding, 3) Test if a valid SID without matching PHPSESSID is accepted on data endpoints
verify_steps: 1) POST /session → get SID1, 2) POST /session/authenticate with valid creds → SID1 authenticated, 3) POST /devices/get with SID1 (capture PHPSESSID), 4) From different browser session, POST /devices/get with SID1 but no PHPSESSID, 5) Check if data is returned
impact: If data endpoints accept SID without PHPSESSID validation, attacker with stolen SID can access full account data without triggering PHP session tracking. Severity: HIGH
testability: AUTH_HELPED
[HYP] API exposes cross-tenant data via session enumeration
class: IDOR
asset: kundenkonto.fonial.de/api/2.0/evn/get
confidence: 45
reasoning: API uses UUID v4 SIDs (good randomness). However, the dual-backend architecture (session endpoint independent of PHP data endpoints) raises the question of whether the PHPSESSID on data endpoints properly scopes data to the authenticated SID. If the PHP session doesn't tightly bind to the SID, session fixation or swapping could expose cross-tenant CDR/phone data.
evidence_needed: 1) Authenticate two different accounts, 2) Verify each gets unique PHPSESSID, 3) Confirm that SID-to-PHPSESSID binding is enforced on data endpoints
verify_steps: 1) Account A: POST /session + authenticate, 2) Extract PHPSESSID from /devices/get response, 3) Account B: same flow, extract different PHPSESSID, 4) Try Account B's SID with Account A's PHPSESSID on /evn/get
impact: Cross-tenant call records (CDR), phone number, and device data exposure. Severity: CRITICAL
testability: AUTH_HELPED
[PARKED] API CORS wildcard: `access-control-allow-origin: *` without `access-control-allow-credentials: true` means browsers won't send cookies cross-origin. SID is in request body, not cookies. Low direct exploitability. Park until chained with another finding (e.g., SID leak via referer).
[FINAL] Survivors ranked:
[NEXT] PROBE: POST https://kundenkonto.fonial.de/api/2.0/session/authenticate — send 20 rapid requests with different passwords to the same SID to confirm no rate limiting or lockout. Measure response times and check for 429/lockout. This is passive rate-limit testing only.
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface.
[RISK] fonial: 40 — Cloudflare-fronted, rate limiting may exist at edge. German company with EnBW parent (enterprise-grade compliance). 2FA on login. However, API lacks visible rate limiting and dual-backend architecture is unusual.
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] API session auth lacks brute-force protection enabling credential stuffing
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/session/authenticate
confidence: 72
reasoning: Auth endpoint returns `{"authenticated":false}` with no delay, lockout, or CAPTCHA after repeated failures. API docs reveal username/password auth with only email-PIN 2FA as secondary. No rate limiting observed across 5+ rapid requests. SIDs are not invalidated on failed auth. Two distinct backends (session vs data) suggest potential session-binding inconsistency.
evidence_needed: 1) Confirm no rate limiting under sustained load (20+ requests), 2) Verify 2FA PIN cannot be bypassed or brute-forced, 3) Check if session expires after N failed attempts
verify_steps: 1) POST /session to get SID, 2) Send 20 rapid POST /session/authenticate with varying passwords, 3) Monitor for 429/lockout/delay, 4) Check if SID remains valid after failures
impact: Credential stuffing against customer accounts. API exposes call records (CDR), phone numbers, device lists, and can initiate outbound calls. Severity: HIGH
testability: AUTH_HELPED
[FINAL] API session auth lacks brute-force protection enabling credential stuffing: confidence 72, above 40, has verify_steps. Keep.
[NEXT] PROBE: GET https://kundenkonto.fonial.de/api/2.0/ to check for error messages or documentation links.
[RISK] fonial: 40 — Cloudflare-fronted, rate limiting may exist at edge. German company with EnBW parent (enterprise-grade compliance). 2FA on login. However, API lacks visible rate limiting and dual-backend architecture is unusual.
[PRIO] kundenkonto.fonial.de/api/2.0,9.1,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.5,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,4.0,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints (/session, /session/authenticate) use SID in JSON body with text/json responses (no PHPSESSID). Data endpoints (/devices/get, /evn/get, /call/initiate) set PHPSESSID cookie with Secure;HttpOnly and return text/json;charset=UTF-8. Independent session stores suggest SID-to-PHPSESSID binding may not be enforced on data layer.
evidence_needed: Proof that data endpoint accepts valid SID without matching PHPSESSID, or that PHPSESSID from Account A works with SID from Account B
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 in body → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
testability: AUTH_HELPED
[HYP] Authentication endpoint lacks rate limiting enabling credential stuffing
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/session/authenticate
confidence: 72
reasoning: Auth endpoint returns `{"authenticated":false}` with no observable delay, lockout, or CAPTCHA after repeated failures. API uses email+password with optional PIN 2FA. No rate limiting observed across 5+ rapid requests. SIDs not invalidated on failed auth. Dual backend increases risk if session layer lacks protection.
evidence_needed: Confirm no 429/lockout/delay after 20+ rapid failed attempts; verify 2FA PIN cannot be brute-forced; check SID invalidation on failures
verify_steps: 1) POST /api/2.0/session → get SID; 2) Send 20 rapid POST /api/2.0/session/authenticate with same SID, varying passwords; 3) Monitor response codes, timing, headers for 429/Retry-Aelay/lockout; 4) Verify SID remains valid after failures
impact: Credential stuffing against customer accounts → full account takeover (CDR, SIP creds, billing, call control) — HIGH
testability: AUTH_HELPED
[HYP] Overly permissive CORS on authenticated customer portal endpoints
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 85
reasoning: Response headers show `access-control-allow-origin: *` on login page (sets PHPSESSID) and likely on authenticated landing page. Wildcard CORS with credentials creates risk if any authenticated JSON endpoint reflects user data without `Vary: Origin`.
evidence_needed: Confirm authenticated JSON endpoints (if any) also return ACAO: * without Vary: Origin
verify_steps: GET https://kundenkonto.fonial.de/ (with PHPSESSID from login redirect) → check CORS headers on response; inspect JS source for API endpoint references
impact: If authenticated API endpoints inherit wildcard CORS, malicious site could exfiltrate call records, SIP credentials, billing data — HIGH
testability: PASSIVE
[PARKED] GraphQL endpoint present but non-functional — potential hidden attack surface: confidence 60 but verify_steps only check public endpoints; no evidence of actual GraphQL API existing on kundenkonto. Low actionability without auth.
[PARKED] API exposes cross-tenant data via session enumeration: confidence 45, depends on unproven session-binding flaw; subsumed by dual-backend session confusion hypothesis above.
[FINAL] 1. Dual-backend session confusion enables cross-tenant data access (75) 2. Authentication endpoint lacks rate limiting enabling credential stuffing (72) 3. Overly permissive CORS on authenticated customer portal endpoints (85)
[NEXT] PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID, then POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credentials (or known invalid) to capture PHPSESSID from data endpoint response, then test cross-bind with second account's SID + first account's PHPSESSID on /api/2.0/evn/get
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. Auth endpoint shows no visible rate limiting. CORS wildcard on auth domain confirmed. Main API not publicly documented but live. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] API session auth lacks brute-force protection enabling credential stuffing
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/session/authenticate
confidence: 72
reasoning: Auth endpoint returns `{"authenticated":false}` with no delay, lockout, or CAPTCHA after repeated failures. API docs reveal username/password auth with only email-PIN 2FA as secondary. No rate limiting observed across 5+ rapid requests. SIDs are not invalidated on failed auth. Two distinct backends (session vs data) suggest potential session-binding inconsistency.
evidence_needed: 1) Confirm no rate limiting under sustained load (20+ requests), 2) Verify 2FA PIN cannot be bypassed or brute-forced, 3) Check if session expires after N failed attempts
verify_steps: 1) POST /session to get SID, 2) Send 20 rapid POST /session/authenticate with varying passwords, 3) Monitor for 429/lockout/delay, 4) Check if SID remains valid after failures
impact: Credential stuffing against customer accounts. API exposes call records (CDR), phone numbers, device lists, and can initiate outbound calls. Severity: HIGH
testability: AUTH_HELPED
[FINAL] API session auth lacks brute-force protection enabling credential stuffing: confidence 72, above 40, has verify_steps. Keep.
[NEXT] PROBE: GET https://kundenkonto.fonial.de/api/2.0/ to check for error messages or documentation links.
[RISK] fonial: 40 — Cloudflare-fronted, rate limiting may exist at edge. German company with EnBW parent (enterprise-grade compliance). 2FA on login. However, API lacks visible rate limiting and dual-backend architecture is unusual.
[NEW] API dual-session binding confirmed: data endpoints (/devices/get, /evn/get) set PHPSESSID AND read authz SID from body — unauthenticated SID -> `"reason":"session unauthenticated"`; absent/unknown SID -> `"reason":"session invalid"`.
[NEW] /api/2.0/session without cookie returns `{"status":"ok","sid":"<uuid4>"}` and sets NO PHPSESSID; presenting a bogus PHPSESSID header causes it to reply with `PHPSESSID=deleted; Max-Age=0`.
[NEW] Confirmed SID is cleartext UUID v4 returned in body; data authz is bound to body SID auth-state, NOT to PHPSESSID cookie (parallel/session-confusion surface).
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] Data-layer SID authz bound to body SID, PHPSESSID cookie parallel + not cross-checked -> cross-tenant if SID not bound to credential
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (data endpoints)
confidence: 55
reasoning: Data endpoints (/devices/get, /evn/get) authorize purely by body `sid` auth-state (`session unauthenticated` vs `session invalid`) and independently set a PHPSESSID cookie. The session endpoint issues cleartext UUID sid. If the authenticated sid-to-account binding lives only in the session store and PHPSESSID is decorative, a stolen/replayed sid could be used cross-tenant without any cookie match.
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected or accepted on /evn/get; confirm sid is bound to credential.
verify_steps: (AUTH_HELPED→HUMAN) 1) /session -> sid A; 2) /session/authenticate A -> auth; 3) /devices/get sid A -> capture PHPSESSID A; 4) /session -> sid B [+ PHPSESSID A cookie]; 5) /evn/get with sid B + PHPSESSID A -> observe data scoping.
impact: Cross-tenant CDR, phone/SIP creds, device lists, outbound call control. Severity CRITICAL if sid not credential-bound.
testability: HUMAN_ONLY (needs 2 valid accounts)
[HYP] CORS wildcard on auth domain lacks Vary: Origin - chaining surface for SID exfiltration if any endpoint reflects body-echoed sid to JS cross-origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 35
reasoning: All endpoints return `access-control-allow-origin: *` with NO `Vary: Origin` (confirmed this run on /, /api/2.0/session, /devices/get). No `allow-credentials`, browser won't send cookies cross-origin. Direct exfil blocked unless an endpoint returns the body sid (not cookie) to a malicious origin — currently no demonstrated JS-readable SID leak.
evidence_needed: find an API/JSON endpoint invoked from JS that returns sid/account data without credentials header.
verify_steps: inspect page JS for fetch() to /api/2.0/* referencing returned sid.
impact: low unless chained with sid leak. Severity LOW-MED.
testability: PASSIVE
[PARKED] CORS wildcard @ kundenkonto.fonial.de: no `Vary: Origin` + no `allow-credentials`; SID in body not cookie; no demonstrated JS sid-leak chain -> confidence 35, park.
[PARKED] Rate-limit/credential-stuffing brute-force class: REJECTED per program scope (brute-force/rate-limit/lockout policy out of scope). Drop as primary.
[FINAL] 1. Data-layer SID-authz cross-tenant (55, HUMAN) 2. None else >40 actionable without creds.
[NEXT] HUMAN: obtain two valid test accounts on kundenkonto.fonial.de sandbox to execute the cross-binding test (sid-vs-PHPSESSID) on /api/2.0/evn/get. Passive automation exhausted without credentials.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: data endpoints authorize by body SID only (`session unauthenticated` vs `session invalid`), PHPSESSID set decoratively in parallel; session endpoint issues cleartext UUID sid and deletes unknown PHPSESSID. Cross-binding between sid and credential is the real open question.
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: out of scope (rate-limit/lockout policy) per program.
[RISK] fonial: 55 — dual-backend (body-SID vs PHPSESSID) architecture creates session-confusion surface; SID cleartext in body; high-value telephony data behind it. But exploitation blocked behind valid-credential step (HUMAN), 2FA present, Cloudflare edge. Not as high as nemotron3's 70 given no demonstrated cross-tenant leak yet.
[NEW] Data endpoints (/devices/get, /evn/get) authorize by body SID only: unauthenticated SID -> `"reason":"session unauthenticated"`; absent/unknown -> `"reason":"session invalid"`. PHPSESSID set decoratively in parallel (not authz-driving).
[NEW] /api/2.0/session (no cookie) -> `{"status":"ok","sid":"<uuid4>"}` no PHPSESSID; bogus PHPSESSID header -> reply `PHPSESSID=deleted; Max-Age=0`. SID is cleartext UUID in body.
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] Data-layer SID authz bound to body SID, PHPSESSID parallel + not cross-checked -> cross-tenant potential
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (data endpoints)
confidence: 55
reasoning: Data endpoints (/devices/get, /evn/get) authorize purely by body `sid` auth-state (`session unauthenticated` vs `session invalid`) and independently set a PHPSESSID cookie. Session endpoint issues cleartext UUID sid. If authenticated sid-to-account binding lives only in session store and PHPSESSID is decorative, a stolen/replayed sid could be used without any cookie match on data endpoints.
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected/accepted on /evn/get; confirm sid is credential-bound.
verify_steps: (HUMAN) 1) /session -> sid A; 2) /session/authenticate A -> auth; 3) /devices/get sid A -> PHPSESSID A; 4) /session -> sid B [+ PHPSESSID A cookie]; 5) /evn/get sid B + PHPSESSID A -> observe scoping.
impact: Cross-tenant CDR, phone/SIP creds, device lists, outbound call control. Severity CRITICAL if sid not credential-bound.
testability: HUMAN_ONLY
[PARKED] CORS wildcard @ kundenkonto.fonial.de: no `Vary: Origin` + no `allow-credentials`; SID in body not cookie; no demonstrated JS sid-leak chain -> confidence 35.
[PARKED] Rate-limit/brute-force/credential-stuffing @ /session/authenticate: REJECTED class (out of scope policy).
[FINAL] 1. Data-layer SID-authz cross-tenant (55, HUMAN_ONLY). Passive presents no other >40 without creds.
[NEXT] HUMAN: obtain two valid test accounts to execute sid-vs-PHPSESSID cross-binding test on /api/2.0/evn/get.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid and deletes unknown PHPSESSID. Cross-binding between sid and credential is the open question.
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: out of scope (rate-limit/lockout policy).
[RISK] fonial: 55 — dual-backend (body-SID vs PHPSESSID) session-confusion surface; SID cleartext in body; high-value telephony data. Exploitation gated behind valid-credential (HUMAN) step; 2FA + Cloudflare reduce immediacy. No demonstrated cross-tenant leak yet.
[HYP] Data-layer SID authz bound to body SID; PHPSESSID parallel/not cross-checked → cross-tenant potential
## 2026-09-03 23:45:12 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] Data-layer SID authz bound to body SID, PHPSESSID parallel + not cross-checked → cross-tenant potential
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (data endpoints)
confidence: 55
reasoning: Data endpoints (/devices/get, /evn/get) authorize purely by body `sid` auth-state (`session unauthenticated` vs `session invalid`) and independently set a PHPSESSID cookie. Session endpoint issues cleartext UUID sid. If authenticated sid-to-account binding lives only in session store and PHPSESSID is decorative, a stolen/replayed sid could be used without any cookie match on data endpoints.
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected/accepted on /evn/get; confirm sid is credential-bound.
verify_steps: (HUMAN) 1) /session → sid A; 2) /session/authenticate A → auth; 3) /devices/get sid A → capture PHPSESSID A; 4) /session → sid B [+ PHPSESSID A cookie]; 5) /evn/get with sid B + PHPSESSID A → observe scoping.
impact: Cross-tenant CDR, phone/SIP creds, device lists, outbound call control. Severity CRITICAL if sid not credential-bound.
testability: HUMAN_ONLY
[PARKED] CORS wildcard @ kundenkonto.fonial.de: no `Vary: Origin` + no `allow-credentials`; SID in body not cookie; no demonstrated JS sid-leak chain → confidence 35, park.
[PARKED] Rate-limit/brute-force/credential-stuffing @ /session/authenticate: REJECTED class (out of scope policy).
[FINAL] 1. Data-layer SID-authz cross-tenant (55, HUMAN_ONLY). Passive presents no other >40 without creds.
[NEXT] HUMAN: obtain two valid test accounts on kundenkonto.fonial.de sandbox to execute the cross-binding test (sid-vs-PHPSESSID) on /api/2.0/evn/get. Passive automation exhausted without credentials.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid and deletes unknown PHPSESSID. Cross-binding between sid and credential is the open question.
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: out of scope (rate-limit/lockout policy).
[RISK] fonial: 55 — dual-backend (body-SID vs PHPSESSID) session-confusion surface; SID cleartext in body; high-value telephony data. Exploitation gated behind valid-credential (HUMAN) step; 2FA + Cloudflare reduce immediacy. No demonstrated cross-tenant leak yet.
## 2026-09-04 02:39:02 UTC [target] (model bigpickle)
[NEW] /api/2.0 endpoint namespace inference: 4 confirmed endpoints → ~30 likely siblings based on /{resource}/{action} pattern
[NEW] CRUD verb set {get, add, update, delete} likely applies — write paths may have different authz behavior
[NEW] File/audio sub-endpoints may hit separate backend — SSRF/path-traversal candidates
[NEW] Session-backend candidates: /session/validate, /session/refresh, /password/*, /2fa/* may exist
[NEW] TYPO3 eID handlers (?eID=xxx) on www.fonial.de untested
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] Data-layer SID authz cross-tenant via session confusion
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (data endpoints)
confidence: 55
reasoning: Data endpoints authorize purely by body `sid` auth-state; PHPSESSID decorative; session issues cleartext UUID sid. If sid-to-account binding lives only in session store, stolen/replayed sid could be used without cookie match.
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected/accepted
verify_steps: (HUMAN) 1) /session → sid A; 2) /session/authenticate A; 3) /devices/get sid A → PHPSESSID A; 4) /session → sid B [+ PHPSESSID A]; 5) /evn/get sid B + PHPSESSID A
impact: Cross-tenant CDR, SIP creds, device lists, call control. CRITICAL if sid not credential-bound.
testability: HUMAN_ONLY
[HYP] Undocumented API endpoints expose write operations with weak SID-only authz
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (CRUD expansion)
confidence: 40
reasoning: /{resource}/{action} pattern implies ~30 endpoints. Write paths (add/delete/update) accepting same body-SID authz could allow cross-tenant mutations if SID binding is permissive. Mass-assignment possible.
evidence_needed: (PASSIVE) GET/HEAD → 405 (exists) vs 404; (HUMAN) POST with SID to write endpoints
verify_steps: (PASSIVE) GET /api/2.0/devices/add, /evn/delete, /extension/get, /cdr/get, /user/get
impact: Account takeover, data exfiltration, unauthorized call control. HIGH.
testability: PASSIVE (discovery) / HUMAN_ONLY (exploitation)
[HYP] File/audio sub-endpoints hit separate backend with different authz
class: SSRF
asset: kundenkonto.fonial.de/api/2.0 (file/audio retrieval)
confidence: 35
reasoning: File-serving backends often use path-based lookup, opening SSRF/path-traversal if input unsanitized.
evidence_needed: (PASSIVE) identify endpoint existence; (HUMAN) test path traversal/SSRF
verify_steps: (PASSIVE) GET /voicemail/getAudio, /fax/download, /cdr/getAudio
impact: SSRF to cloud metadata, file disclosure. CRITICAL if achieved.
testability: HUMAN_ONLY
[PARKED] CORS wildcard @ kundenkonto.fonial.de: confidence 35, no JS sid-leak chain
[PARKED] Rate-limit/brute-force: REJECTED class (out of scope)
[PARKED] File/audio sub-endpoints: confidence 35 < 40, pending discovery
[FINAL] 1. Data-layer SID-authz cross-tenant (55, HUMAN_ONLY). 2. Undocumented API CRUD (40, PASSIVE+HUMAN). 3. File/audio endpoints (35, PARKED).
[NEXT] PROBE: GET https://kundenkonto.fonial.de/api/2.0/devices/add — check 405 vs 404. If 405, expand to GET /api/2.0/extension/get, /user/get, /contact/get, /account/get, /cdr/get, /invoice/get to map full namespace.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
[LEARN] ACCEPTED CRUD pattern: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings.
[RISK] fonial: 55 — dual-backend session-confusion surface; SID cleartext in body; high-value telephony data. Exploitation gated behind valid-credential (HUMAN) step; 2FA + Cloudflare reduce immediacy. No demonstrated cross-tenant leak yet.
## 2026-09-04 07:24:58 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] Data-layer SID authz cross-tenant via session confusion
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (data endpoints)
confidence: 55
reasoning: Data endpoints (/devices/get, /evn/get) authorize purely by body `sid` auth-state (`session unauthenticated` vs `session invalid`) and independently set a PHPSESSID cookie. Session endpoint issues cleartext UUID sid. If authenticated sid-to-account binding lives only in session store and PHPSESSID is decorative, a stolen/replayed sid could be used without any cookie match on data endpoints.
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected/accepted on /evn/get; confirm sid is credential-bound.
verify_steps: (HUMAN) 1) /session → sid A; 2) /session/authenticate A → auth; 3) /devices/get sid A → capture PHPSESSID A; 4) /session → sid B [+ PHPSESSID A cookie]; 5) /evn/get with sid B + PHPSESSID A → observe scoping.
impact: Cross-tenant CDR, phone/SIP creds, device lists, outbound call control. Severity CRITICAL if sid not credential-bound.
testability: HUMAN_ONLY
[HYP] Undocumented API endpoints expose write operations with weak SID-only authz
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (CRUD expansion)
confidence: 40
reasoning: /{resource}/{action} pattern implies ~30 endpoints. Write paths (add/delete/update) accepting same body-SID authz could allow cross-tenant mutations if SID binding is permissive. Mass-assignment possible.
evidence_needed: (PASSIVE) GET/HEAD → 405 (exists) vs 404; (HUMAN) POST with SID to write endpoints
verify_steps: (PASSIVE) GET /api/2.0/devices/add, /evn/delete, /extension/get, /cdr/get, /user/get
impact: Account takeover, data exfiltration, unauthorized call control. HIGH.
testability: PASSIVE (discovery) / HUMAN_ONLY (exploitation)
[HYP] File/audio sub-endpoints hit separate backend with different authz
class: SSRF
asset: kundenkonto.fonial.de/api/2.0 (file/audio retrieval)
confidence: 35
reasoning: File-serving backends often use path-based lookup, opening SSRF/path-traversal if input unsanitized.
evidence_needed: (PASSIVE) identify endpoint existence; (HUMAN) test path traversal/SSRF
verify_steps: (PASSIVE) GET /voicemail/getAudio, /fax/download, /cdr/getAudio
impact: SSRF to cloud metadata, file disclosure. CRITICAL if achieved.
testability: HUMAN_ONLY
[PARKED] CORS wildcard @ kundenkonto.fonial.de: no `Vary: Origin` + no `allow-credentials`; SID in body not cookie; no demonstrated JS sid-leak chain → confidence 35, park.
[PARKED] Rate-limit/brute-force/credential-stuffing @ /session/authenticate: REJECTED class (out of scope policy).
[PARKED] File/audio sub-endpoints: confidence 35 < 40, pending discovery → park until namespace expanded.
[FINAL]
[NEXT] PROBE: GET https://kundenkonto.fonial.de/api/2.0/devices/add — check 405 vs 404. If 405, expand to GET /api/2.0/extension/get, /user/get, /contact/get, /account/get, /cdr/get, /invoice/get to map full namespace.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
[LEARN] ACCEPTED CRUD pattern: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings.
[RISK] fonial: 55 — dual-backend session-confusion surface; SID cleartext in body; high-value telephony data. Exploitation gated behind valid-credential (HUMAN) step; 2FA + Cloudflare reduce immediacy. No demonstrated cross-tenant leak yet.
## 2026-09-04 12:16:16 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0, 9.2, a=9 b=10 t=8 g=8 c=9 f=10
[PRIO] www.fonial.de (TYPO3 CMS + login), 6.5, a=6 b=7 t=5 g=4 c=6 f=8
[PRIO] www.fonial.de/shop/, 5.8, a=6 b=5 t=5 g=3 c=5 f=7
[HYP] Data-layer SID authz cross-tenant via session confusion
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (data endpoints)
confidence: 55
reasoning: Data endpoints (/devices/get, /evn/get) authorize purely by body `sid` auth-state (`session unauthenticated` vs `session invalid`) and independently set a PHPSESSID cookie. Session endpoint issues cleartext UUID sid. If authenticated sid-to-account binding lives only in session store and PHPSESSID is decorative, a stolen/replayed sid could be used without any cookie match on data endpoints.
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected/accepted on /evn/get; confirm sid is credential-bound.
verify_steps: (HUMAN) 1) /session → sid A; 2) /session/authenticate A → auth; 3) /devices/get sid A → capture PHPSESSID A; 4) /session → sid B [+ PHPSESSID A cookie]; 5) /evn/get with sid B + PHPSESSID A → observe scoping.
impact: Cross-tenant CDR, phone/SIP creds, device lists, outbound call control. Severity CRITICAL if sid not credential-bound.
testability: HUMAN_ONLY
[HYP] Undocumented API endpoints expose write operations with weak SID-only authz
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (CRUD expansion)
confidence: 40
reasoning: /{resource}/{action} pattern implies ~30 endpoints. Write paths (add/delete/update) accepting same body-SID authz could allow cross-tenant mutations if SID binding is permissive. Mass-assignment possible.
evidence_needed: (PASSIVE) GET/HEAD → 405 (exists) vs 404; (HUMAN) POST with SID to write endpoints
verify_steps: (PASSIVE) GET /api/2.0/devices/add, /evn/delete, /extension/get, /cdr/get, /user/get
impact: Account takeover, data exfiltration, unauthorized call control. HIGH.
testability: PASSIVE (discovery) / HUMAN_ONLY (exploitation)
[HYP] File/audio sub-endpoints hit separate backend with different authz
class: SSRF
asset: kundenkonto.fonial.de/api/2.0 (file/audio retrieval)
confidence: 35
reasoning: File-serving backends often use path-based lookup, opening SSRF/path-traversal if input unsanitized.
evidence_needed: (PASSIVE) identify endpoint existence; (HUMAN) test path traversal/SSRF
verify_steps: (PASSIVE) GET /voicemail/getAudio, /fax/download, /cdr/getAudio
impact: SSRF to cloud metadata, file disclosure. CRITICAL if achieved.
testability: HUMAN_ONLY
[PARKED] CORS wildcard @ kundenkonto.fonial.de: no `Vary: Origin` + no `allow-credentials`; SID in body not cookie; no demonstrated JS sid-leak chain → confidence 35, park.
[PARKED] Rate-limit/brute-force/credential-stuffing @ /session/authenticate: REJECTED class (out of scope policy).
[PARKED] File/audio sub-endpoints: confidence 35 < 40, pending discovery → park until namespace expanded.
[FINAL]
[NEXT] PROBE: GET https://kundenkonto.fonial.de/api/2.0/devices/add — check 405 vs 404. If 405, expand to GET /api/2.0/extension/get, /user/get, /contact/get, /account/get, /cdr/get, /invoice/get to map full namespace.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
[LEARN] ACCEPTED CRUD pattern: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings.
[RISK] fonial: 55 — dual-backend session-confusion surface; SID cleartext in body; high-value telephony data. Exploitation gated behind valid-credential (HUMAN) step; 2FA + Cloudflare reduce immediacy. No demonstrated cross-tenant leak yet.
## 2026-09-04 16:29:56 UTC [target] (model bigpickle)
[NEW] API surface is 5 endpoints, NOT ~30: session, session/authenticate, devices/get, evn/get, call/initiate — all other guessed names return HTML 404 (web app fallback)
[NEW] /call/initiate confirmed LIVE WRITE endpoint: POST returns JSON {"sid":null,"reason":"session invalid"} without auth
[NEW] Frontend HTML exposes routes: /wizard/, /settings/menu/toggle, /help/suggest/__search__, /settings/ticket/
[CHANGED] CRUD expansion hypothesis (70→30): ~50 guessed {resource}/{action} names all HTML-404; API uses non-standard naming or narrow surface
[PRIO] kundenkonto.fonial.de/api/2.0, 8.5, a=7 b=10 t=7 g=7 c=8 f=10
[PRIO] kundenkonto.fonial.de (frontend portal), 6.0, a=6 b=7 t=5 g=7 c=5 f=8
[PRIO] www.fonial.de, 4.0, a=4 b=5 t=3 g=10 c=4 f=5
[HYP] call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/call/initiate
confidence: 55
reasoning: /call/initiate returns same session-invalid JSON as read endpoints (devices/get, evn/get). This is a WRITE endpoint (initiates outbound calls to real phone numbers). If SID authz behaves identically to read endpoints, a stolen SID enables unauthorized outbound calls (billing fraud, toll fraud). If dual-backend SID/PHPSESSID confusion applies here, cross-tenant call initiation may be possible. Critical: this endpoint is higher-severity than read endpoints because it initiates real telephony actions.
evidence_needed: (HUMAN) authenticate → POST /call/initiate with valid SID and caller/callee params → observe if call connects; (HUMAN) cross-bind test with Account A SID + Account B PHPSESSID
verify_steps: (HUMAN) 1) POST /session → SID_A; 2) POST /session/authenticate → auth; 3) POST /call/initiate with {"sid":"SID_A","number":"+49...","caller":"..."} → observe 200/error; 4) If call connects: unauthorized outbound call = CRITICAL
impact: Unauthorized outbound calls → toll fraud, billing manipulation, wiretapping via call routing. CRITICAL if SID not properly bound.
testability: HUMAN_ONLY (needs valid account + phone number)
[HYP] Session confusion cross-tenant on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 55
reasoning: All 3 data endpoints share identical JSON error behavior and PHPSESSID decoration. If SID-to-account binding is weak, all 3 are simultaneously vulnerable to cross-tenant access. The attack surface is narrow (5 endpoints) but high-value (telephony data + outbound call control).
evidence_needed: (HUMAN) authenticate 2 accounts; cross-bind SID A + PHPSESSID B on each endpoint
verify_steps: (HUMAN) Same as prior: sid-vs-PHPSESSID cross-binding test on all 3 data endpoints
impact: Cross-tenant CDR + device data + outbound call control. CRITICAL.
testability: HUMAN_ONLY
[HYP] Frontend wizard/settings routes expose unauthenticated actions
class: MISCONFIG
asset: kundenkonto.fonial.de/wizard/, kundenkonto.fonial.de/settings/
confidence: 35
reasoning: Frontend HTML exposes /wizard/, /settings/menu/toggle, /help/suggest/ routes. If these are accessible without authentication, they could leak configuration data or allow unauthorized setup changes. However, the HTML 404 on /api/2.0/ root suggests the frontend may require PHPSESSID auth.
evidence_needed: (PASSIVE) GET /wizard/ and /settings/menu/toggle without auth → check if 200/403/302
verify_steps: (PASSIVE) GET https://kundenkonto.fonial.de/wizard/ → check status; GET https://kundenkonto.fonial.de/settings/menu/toggle → check status
impact: Configuration disclosure, unauthorized setup. MEDIUM if accessible.
testability: PASSIVE
[PARKED] Frontend wizard/settings routes: confidence 35 < 40; unproven accessibility; MEDIUM impact at best. Park.
[FINAL]
[NEXT] HUMAN: obtain 2 valid test accounts on kundenkonto.fonial.de to execute cross-binding test: (1) authenticate Account A → SID A → devices/get → PHPSESSID A; (2) authenticate Account B → SID B; (3) POST /call/initiate with SID B + Cookie: PHPSESSID=A; (4) POST /devices/get with SID B + PHPSESSID=A; (5) POST /evn/get with SID B + PHPSESSID=A. If any returns data = cross-tenant CRITICAL.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 guessed names all HTML-404.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz suspected.
[LEARN] REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
[RISK] fonial: 55 — Narrow API surface (5 endpoints) limits total attack area, but call/initiate is a high-severity WRITE endpoint (outbound telephony). Dual-backend session confusion persists as primary vector. 2FA + Cloudflare reduce automated exploitation. No demonstrated cross-tenant leak yet; exploitation gated behind valid credentials (HUMAN_ONLY).
[NEW] API surface is 5 endpoints, NOT ~30: session, session/authenticate, devices/get, evn/get, call/initiate — all other guessed names return HTML 404
[NEW] /call/initiate confirmed LIVE WRITE endpoint: POST returns JSON session-invalid without auth
[NEW] Frontend HTML exposes routes: /wizard/, /settings/menu/toggle, /help/suggest/, /settings/ticket/
[CHANGED] CRUD expansion hypothesis (70→30): ~50 guessed {resource}/{action} names all HTML-404; API uses non-standard naming or narrow surface
[PRIO] kundenkonto.fonial.de/api/2.0, 8.5, a=7 b=10 t=7 g=7 c=8 f=10
[PRIO] kundenkonto.fonial.de (frontend portal), 6.0, a=6 b=7 t=5 g=7 c=5 f=8
[PRIO] www.fonial.de, 4.0, a=4 b=5 t=3 g=10 c=4 f=5
[HYP] call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/call/initiate
confidence: 55
reasoning: /call/initiate returns same session-invalid JSON as read endpoints. This is a WRITE endpoint (initiates outbound calls). If SID authz behaves identically, stolen SID enables unauthorized outbound calls (toll fraud). If dual-backend SID/PHPSESSID confusion applies, cross-tenant call initiation possible. Higher-severity than read endpoints.
evidence_needed: (HUMAN) authenticate → POST /call/initiate with valid SID and caller/callee params; cross-bind test
verify_steps: (HUMAN) 1) /session → SID_A; 2) /session/authenticate → auth; 3) /call/initiate with {"sid":"SID_A","number":"+49...","caller":"..."} → observe; 4) Cross-bind: SID B + PHPSESSID A
impact: Unauthorized outbound calls → toll fraud, billing manipulation. CRITICAL if SID not bound.
testability: HUMAN_ONLY
[HYP] Session confusion cross-tenant on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 55
reasoning: All 3 data endpoints share identical JSON error behavior and PHPSESSID decoration. If SID-to-account binding is weak, all 3 are simultaneously vulnerable.
evidence_needed: (HUMAN) authenticate 2 accounts; cross-bind SID A + PHPSESSID B on each endpoint
verify_steps: (HUMAN) Same as prior: sid-vs-PHPSESSID cross-binding test on all 3 data endpoints
impact: Cross-tenant CDR + device data + outbound call control. CRITICAL.
testability: HUMAN_ONLY
[PARKED] Frontend wizard/settings routes: confidence 35 < 40. Park.
[FINAL] 1. call/initiate write endpoint SID authz (55, HUMAN) 2. Session confusion cross-tenant on 3 endpoints (55, HUMAN)
[NEXT] HUMAN: obtain 2 valid test accounts; cross-bind SID B + PHPSESSID A on /call/initiate, /devices/get, /evn/get
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist; ~50 guessed names all HTML-404.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; same session-invalid JSON pattern.
[LEARN] REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative.
[RISK] fonial: 55 — Narrow API surface (5 endpoints) limits total attack area, but call/initiate is high-severity WRITE (outbound telephony). Dual-backend session confusion persists as primary vector. 2FA + Cloudflare reduce automated exploitation. No demonstrated cross-tenant leak; gated behind valid credentials.
## 2026-09-04 19:06:53 UTC [target] (model bigpickle)
## 2026-09-04 21:33:03 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0, 8.5, a=7 b=10 t=7 g=7 c=8 f=10
[PRIO] kundenkonto.fonial.de (auth portal), 6.0, a=6 b=7 t=5 g=7 c=5 f=8
[PRIO] www.fonial.de, 4.0, a=4 b=5 t=3 g=10 c=4 f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID → "session unauthenticated", unknown → "session invalid"); PHPSESSID set decoratively in parallel and /session deletes unknown PHPSESSIDs; SID is a cleartext UUID returned client-side. If SID↔credential binding is weak, one SID can ride another tenant's PHPSESSID state on 3 endpoints incl. WRITE call/initiate.
evidence_needed: (HUMAN) authenticate 2 accounts; cross-bind SID-A + PHPSESSID-B on devices/get, evn/get, call/initiate; any data return = cross-tenant CRITICAL.
verify_steps: (HUMAN) 1) /session → SID-A → /session/authenticate(A) → note PHPSESSID-A; 2) repeat → SID-B + PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} with Cookie PHPSESSID=A; 4) same cross-bind on /evn/get, /call/initiate.
impact: Cross-tenant CDR + device + outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] Frontend wizard/settings routes expose unauthenticated actions
class: MISCONFIG
asset: kundenkonto.fonial.de/wizard/, /settings/*
confidence: 25
reasoning: Routes exist in HTML SPA bundle; all return 302 → /login without PHPSESSID; login-required 302 implies app-level gate, consistent with SPA expecting authenticated session. No unauth behavior observed.
evidence_needed: none — tested; all auth-gated.
verify_steps: none (probed).
impact: none observed. LOW.
testability: PASSIVE
[PARKED] Frontend wizard/settings routes: probed — all 302 → /login; no unauthenticated surface; confidence 25 < 40. Kill.
[FINAL] 1. Dual-backend SID cross-binding cross-tenant on 3 data endpoints (75, HUMAN_ONLY) 2. call/initiate WRITE SID authz / cross-tenant call control (55, HUMAN_ONLY, folded into FINAL-1 verify matrix)
[NEXT] HUMAN: obtain 2 valid test accounts; execute 2-account cross-bind matrix: for each of /devices/get, /evn/get, /call/initiate POST SID-B body + Cookie PHPSESSID=A; any response ≠ {"sid":null,"status":"error","reason":"session invalid"} = cross-tenant CRITICAL. Passive surface exhausted — all remaining value is gated behind credentials.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow-credentials anywhere → no cookie cross-origin exfil channel; SID-in-body required.
[LEARN] REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/setup surface.
[LEARN] ACCEPTED narrow-API @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; this closes passive discovery on the API (split-frontend + SPA routes enumerated).
[RISK] fonial: 50 — Surface fully mapped and narrow (5 endpoints + auth-gated SPA). CORS misconfig is real but non-exploitable as configured (no creds, SID-in-body). Dual-backend SID/PHPSESSID confusion remains the single high-value vector but is unproven and gated behind 2 valid accounts (2FA + Cloudflare). call/initiate adds toll-fraud upside if cross-riding works. Passive phase complete; risk caps at 50 until human cross-bind test.
## 2026-09-04 23:17:16 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0, 8.5, a=7 b=10 t=7 g=7 c=8 f=10
[PRIO] kundenkonto.fonial.de (frontend/SPA bundle), 6.0, a=6 b=7 t=5 g=7 c=5 f=8
[PRIO] www.fonial.de, 4.0, a=4 b=5 t=3 g=10 c=4 f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth → "session unauthenticated", unknown → "session invalid"); PHPSESSID set decoratively in parallel; /session deletes unknown PHPSESSIDs and returns cleartext UUID sid in body. If SID↔credential binding is weak, one SID can ride another tenant's PHPSESSID state on 3 endpoints incl. WRITE call/initiate.
evidence_needed: (HUMAN) authenticate 2 accounts; cross-bind SID-A + PHPSESSID-B on each endpoint; any response ≠ session-invalid = cross-tenant CRITICAL
verify_steps: 1) /session → SID-A → /session/authenticate(A) → PHPSESSID-A; 2) repeat → B pair; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; 4) same on /evn/get, /call/initiate
impact: Cross-tenant CDR + device visibility + outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] SPA JS bundle exposes undocumented endpoints and exact write-param schema
class: OTHER
asset: kundenkonto.fonial.de SPA bundle (script src from /login HTML)
confidence: 55
reasoning: ~50 blind {resource}/{action} names all hit HTML-404 wall → API uses non-standard naming, fuzz is dead. The SPA driving the 5 known endpoints must embed endpoint paths + param names in JS; that's the only remaining low-cost way to prove surface=5 and recover call/initiate body fields so the cross-bind test is field-accurate.
evidence_needed: bundle grep reveals ≥1 new /api/2.0/ string OR exact write-param set; clean 5/5-only result closes discovery with high confidence
verify_steps: 1) GET https://kundenkonto.fonial.de/ (302→/login), parse HTML for <script src>; 2) GET each bundle (static, ≤1rps); 3) rg -o 'api/2\.0/[a-z/_-]+' plus window-context for sid|number|caller|callee|dest|queue
impact: reopens surface if new endpoint, else yields precise call/initiate schema → sharper HUMAN verify matrix. LOW-MED.
testability: PASSIVE
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ → follow to /login HTML → extract <script src> list → GET each JS bundle → grep 'api/2.0/' and {sid,number,caller,callee,dest,queue}; any new endpoint reopens surface, else finalize call/initiate body schema for the HUMAN cross-bind matrix.
[RISK] fonial: 50 — Passive phase complete; surface narrow (5 endpoints), every high-value vector gated behind 2FA credentials (Cloudflare + 2FA). CORS wildcard real but non-exploitable as configured (no allow-credentials, SID-in-body). Top vector dual-backend cross-bind (75) unproven and HUMAN-gated; JS bundle grep is last passive upside — could reopen surface or sharpen call/initiate write-authz test. Risk caps at 50 until the 2-account cross-bind matrix.
## 2026-09-05 01:06:51 UTC [target] (model bigpickle)
[HYP] app_dev.php Symfony dev-mode debug exposure on DSL customer portal leaks internal paths and may expose further debug routes
class: MISCONFIG
asset: dslkonto.fonial.de (/app_dev.php/*)
confidence: 70
reasoning: /app_dev.php serves the app (200 login, 302 redirects) without any observed IP allowlist; triggered 404 renders full Symfony debug exception page confirming Env=dev, leaking /pkg/srv/application_2026_07_02_1410/…cache/fonial/app_frontend/classes.php, deploy tag, TraceableEventDispatcher/WrappedListener (dev-only). Main host /api/2.0/* fatals to empty 500 (PHP method-guard). _profiler/_wdt 404 — profiler disabled.
evidence_needed: any additional dev-only disclosure: debug page source-snippets, /env/.env access via app_dev, dev-only routes responding on app_dev but not /; or acceptance as leftover-dev-entrypoint finding.
verify_steps: 1) GET /app_dev.php/login (200, dev boot); 2) GET /app_dev.php/api/2.0/session → 404 debug page (already captured, path leak); 3) GET /app_dev.php/_profiler, /app_dev.php/_wdt (404, profiler off); 4) GET /app_dev.php/forgot/request, /app_dev.php/register for non-main-host routes; 5) OPTIONS /api/2.0/session on main host (500, method-guard confirmed).
impact: Info disclosure (filesystem layout, deploy artifact tags, Symfony internals) + dev-mode debug pages available pre-auth; enables targeted exploit dev. LOW-MED, benign-alone but a genuine left-over-dev build.
testability: PASSIVE
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access; SID also mintable via GET, and public signup funnel may let an attacker self-register the second account
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 78
reasoning: Data endpoints authorize by body SID only; GET /api/2.0/session now confirmed to mint cleartext SID (no POST needed); /signup → /signup/register/55 public funnel exists (loops pre-auth, needs full flow) → potential self-registration reduces HUMAN gate; PHPSESSID decorative and /session deletes unknown ones.
evidence_needed: (HUMAN or AUTH_HELPED) 2 valid SIDs from 2 tenants; cross-bind SID-A + PHPSESSID-B on devices/get, evn/get, call/initiate; public-funnel completion to mint account-2 without vendor help.
verify_steps: 1) GET /api/2.0/session → SID-A (done); 2) repeat anonymous → SID-X; confirm any 2 distinct SIDs auth-independent (session unauthenticated vs invalid); 3) HUMAN: complete /signup funnel to create 2nd tenant; 4) POST devices/get {"sid":SID-B} + Cookie PHPSESSID=A on all 3 endpoints.
impact: Cross-tenant CDR + device visibility + outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY (upgrading to AUTH_HELPED if self-registration completes)
[NEXT] PROBE: GET https://dslkonto.fonial.de/app_dev.php/ (200 dev boot, confirm no IP block), then GET https://dslkonto.fonial.de/app_dev.php/forgot/request and https://dslkonto.fonial.de/app_dev.php/register — compares dev vs main-host route sets on the new DSK portal to bound the dev-debug exposure; continue ≤1rps GET-only.
[RISK] fonial: 58 — Surface expanded: new live customer portal (DSL) with confirmed dev-mode Symfony debug exposure (pre-auth path/config leak) and an independent /api/2.0 namespace, plus a GET SID-mint on the existing VoIP API and a public self-registration funnel. CORS wildcard remains non-exploitable as configured and dual-backend cross-bind (78) still unproven and credential-gated, so risk stays moderate; 50→58 on breadth + confirmed-but-low-severity info disclosure.
## 2026-09-05 05:52:15 UTC [target] (model bigpickle)
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID decorative; /session mints cleartext UUID via GET and deletes unknown PHPSESSIDs. All passive supersets on both portals now closed (kundenkonto surface=5, bundle grep dead; DSL API/register broken), leaving only the 2-account matrix to prove/disprove the binding.
evidence_needed: (HUMAN) 2 valid SIDs from 2 tenants; cross-bind SID-A + PHPSESSID-B on each endpoint; any response ≠ session-invalid = cross-tenant CRITICAL. kundenkonto /signup funnel exists (login page links it) as self-registration path for account-2.
verify_steps: 1) GET /api/2.0/session → SID-A; 2) second account → SID-B/PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; repeat /evn/get, /call/initiate; 4) try completing /signup funnel to mint account-2 without vendor help.
impact: Cross-tenant CDR, device visibility, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] app_dev.php Symfony dev-mode debug exposure is broader than 404-path leaks: POST-triggered 500 debug pages embed request/session/exception detail that prod silently suppresses
class: MISCONFIG
asset: dslkonto.fonial.de/app_dev.php/*
confidence: 70
reasoning: Confirmed pre-auth debug pages on 404 (paths, `pkg/srv/application_2026_07_02_1410`, TraceableEventDispatcher) AND on 500 (`/register`: "Product not found" + `User\UserBundle\RegistrationController`). Prod suppresses identical errors to empty 500. Debug tabs included Session; env/params tab not observed.
evidence_needed: any debug page dumping env vars/DB params/cookies of the request, or a dev-only route that responds only via app_dev.php (toolbar/profiler off is a known 404).
verify_steps: 1) GET /app_dev.php/register (done — session tab, no env dump); 2) GET /app_dev.php/zzz (done — path leak only); 3) GET /app_dev.php/_profiler, /_wdt (404, off); 4) GET /app_dev.php/config.php (403, blocked). Remaining POST-triggered pages restricted by passive-only rule.
impact: Info disclosure (filesystem layout, deploy tag, internal bundle naming, exception semantics) pre-auth; enables targeted exploit dev. LOW-MED, benign alone — report as leftover-dev-build.
testability: PASSIVE (mostly exhausted)
[HYP] DSL portal default-product misconfiguration blocks registration — public self-signup on dslkonto dead, product-param logic may mis-handle attacker-supplied product selection elsewhere
class: BUSLOGIC
asset: dslkonto.fonial.de/register (and /signup-adjacent flows)
confidence: 45
reasoning: "Product not found. Did you set the default product?" fatals pre-auth on both prod (silent 500) and dev (debug). Suggests product-entity dependency with no default; a registration/onboarding-dependent flow exists in domain logic. No exploitable read/write path tested.
evidence_needed: attacker-controllable product/id/plan parameter anywhere in the DSL flow that triggers different branch behavior than default-missing.
verify_steps: 1) GET /app_dev.php/register (done, broken); 2) GET /register attending no params → 500; 3) GET /forgot/request and /login (200, normal). No param-injection surface found.
impact: Availability of public signup (DoS-flavored → largely out of scope) + domain-logic insight. NI to LOW.
testability: PASSIVE
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), then POST {"sid":"SID-B"} with Cookie PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL (toll-fraud upside). If account-2 is hard to source, first attempt the /signup funnel linked from /login.
[RISK] fonial: 62 — Surface is now fully mapped on both live portals (kundenkonto 5-endpoint API + auth-gated SPA; dslkonto 3 front controllers, broken API namespace and registration). New confirmed-but-low finding: pre-auth Symfony app_dev.php debug exposure on a live customer portal (paths, deploy tag, exception/domain semantics; LOW-MED, report-friendly). All engineered high-value upside (dual-backend SID/PHPSESSID cross-bind, CORS+SID chain) remains credential-gated and unproven → risk stays moderate; one human cross-bind matrix (75) either lands CRITICAL cross-tenant/toll-fraud or definitively closes the engagement.
## 2026-09-05 09:51:52 UTC [target] (model bigpickle)
asset: dslkonto.fonial.de (/app_dev.php/*)
[NEXT] PROBE: GET https://dslkonto.fonial.de/app_dev.php/ (200 dev boot, confirm no IP block), then GET https://dslkonto.fonial.de/app_dev.php/forgot/request and https://dslkonto.fonial.de/app_dev.php/register — compares dev vs main-host route sets on the new DSK portal to bound the dev-debug exposure; continue ≤1rps GET-only.
asset: dslkonto.fonial.de/app_dev.php/*
[HYP] DSL portal default-product misconfiguration blocks registration — public self-signup on dslkonto dead, product-param logic may mis-handle attacker-supplied product selection elsewhere
asset: dslkonto.fonial.de/register (and /signup-adjacent flows)
[RISK] fonial: 62 — Surface is now fully mapped on both live portals (kundenkonto 5-endpoint API + auth-gated SPA; dslkonto 3 front controllers, broken API namespace and registration). New confirmed-but-low finding: pre-auth Symfony app_dev.php debug exposure on a live customer portal (paths, deploy tag, exception/domain semantics; LOW-MED, report-friendly). All engineered high-value upside (dual-backend SID/PHPSESSID cross-bind, CORS+SID chain) remains credential-gated and unproven → risk stays moderate; one human cross-bind matrix (75) either lands CRITICAL cross-tenant/toll-fraud or definitively closes the engagement.
[FINAL] survivor 1: Dual-backend cross-bind. Already elaborated, confidence 75, HUMAN_ONLY.
[FINAL] survivor 2: dslkonto app_dev dev-mode exposure — report-ready, LOW-MED.
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID decorative; /session mints cleartext UUID via GET and deletes unknown PHPSESSIDs. All passive supersets now closed on both portals (kundenkonto bundle grep dead; DSL API absent, registration broken). Only the 2-account matrix can prove/disprove the binding.
evidence_needed: (HUMAN) 2 valid SIDs from 2 tenants; cross-bind SID-A + PHPSESSID-B on each endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL.
verify_steps: 1) GET /api/2.0/session → SID-A; 2) second account → SID-B/PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; repeat /evn/get, /call/initiate; 4) try completing /signup funnel (linked from /login) to mint account-2 without vendor help.
impact: Cross-tenant CDR, device visibility, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] DSL portal dev-mode Symfony debug exposure leaks internal paths/deploy-tag/domain semantics pre-auth on a live customer portal
class: MISCONFIG
asset: dslkonto.fonial.de/app_dev.php/*
confidence: 70
reasoning: /app_dev.php boots full app (200 /, 302→/login with PHPSESSID) with no IP block; 404 renders full Symfony debug page (fs path `pkg/srv/application_2026_07_02_1410`, deploy tag, TraceableEventDispatcher dev-only); 500 on /register prints `User\UserBundle\RegistrationController` + "Product not found". Prod suppresses identical errors to empty 500. Profiler/router/phpinfo confirmed 404 (off).
evidence_needed: any debug page dumping env/params/session of the request, or a dev-only route responding only via app_dev.php; current passive evidence (paths/deploy/exception semantics) already sufficient for LOW-MED report.
verify_steps: 1) GET /app_dev.php/ (200 dev boot, 302→login); 2) GET /app_dev.php/api/2.0/session (404 debug, proves DSL has no VoIP API route); 3) GET /app_dev.php/_profiler,_wdt,config.php (404/403 — off/blocked); remaining POST-triggered pages restricted by passive-only rule.
impact: Info disclosure (filesystem layout, deploy artifact tag, internal bundle naming, exception semantics) pre-auth; enables targeted exploit dev. LOW-MED, benign-alone — report as leftover dev build.
testability: PASSIVE (mostly exhausted)
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), then POST {"sid":"SID-B"} with Cookie: PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL (toll-fraud upside). If a 2nd tenant is hard to source, complete the /signup funnel linked from /login first.
[RISK] fonial: 62 — Surface fully mapped on both portals; new corrections shrink rather than grow footprint (DSL has no VoIP API, dev exposure bounded to path/exception detail). All engineered high-value upside (dual-backend SID/PHPSESSID cross-bind 75, CORS+SID chain) remains credential-gated and unproven → risk stays moderate. One human cross-bind matrix either lands CRITICAL cross-tenant/toll-fraud or definitively closes the engagement. Reportable confirmed findings: dslkonto dev-mode debug exposure (LOW-MED) + CORS wildcard non-exploitable-as-configured (info only).
## 2026-09-05 13:13:44 UTC [target] (model bigpickle)
[HYP] Frontend wizard/settings routes expose unauthenticated actions
class: MISCONFIG
asset: kundenkonto.fonial.de/wizard/, /settings/*
confidence: 25
reasoning: Routes exist in HTML SPA bundle; all return 302 → /login without PHPSESSID; login-required 302 implies app-level gate, consistent with SPA expecting authenticated session. No unauth behavior observed.
evidence_needed: none — tested; all auth-gated.
verify_steps: none (probed).
impact: none observed. LOW.
testability: PASSIVE
[PARKED] Frontend wizard/settings routes: probed — all 302 → /login; no unauthenticated surface; confidence 25 < 40. Kill.
[FINAL] 1. Dual-backend SID cross-binding cross-tenant on 3 data endpoints (75, HUMAN_ONLY) 2. call/initiate WRITE SID authz / cross-tenant call control (55, HUMAN_ONLY, folded into FINAL-1 verify matrix)
[NEXT] HUMAN: obtain 2 valid test accounts; execute 2-account cross-bind matrix: for each of /devices/get, /evn/get, /call/initiate POST SID-B body + Cookie PHPSESSID=A; any response ≠ {"sid":null,"status":"error","reason":"session invalid"} = cross-tenant CRITICAL. Passive surface exhausted — all remaining value is gated behind credentials.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow-credentials anywhere → no cookie cross-origin exfil channel; SID-in-body required.
[LEARN] REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/setup surface.
[LEARN] ACCEPTED narrow-API @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; this closes passive discovery on the API (split-frontend + SPA routes enumerated).
[RISK] fonial: 50 — Surface fully mapped and narrow (5 endpoints + auth-gated SPA). CORS misconfig is real but non-exploitable as configured (no creds, SID-in-body). Dual-backend SID/PHPSESSID confusion remains the single high-value vector but is unproven and gated behind 2 valid accounts (2FA + Cloudflare). call/initiate adds toll-fraud upside if cross-riding works. Passive phase complete; risk caps at 50 until human cross-bind test.
[PRIO] kundenkonto.fonial.de/api/2.0, 8.5, a=7 b=10 t=7 g=7 c=8 f=10
[PRIO] kundenkonto.fonial.de (frontend/SPA bundle), 6.0, a=6 b=7 t=5 g=7 c=5 f=8
[PRIO] www.fonial.de, 4.0, a=4 b=5 t=3 g=10 c=4 f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth → "session unauthenticated", unknown → "session invalid"); PHPSESSID set decoratively in parallel; /session deletes unknown PHPSESSIDs and returns cleartext UUID sid in body. If SID↔credential binding is weak, one SID can ride another tenant's PHPSESSID state on 3 endpoints incl. WRITE call/initiate.
evidence_needed: (HUMAN) authenticate 2 accounts; cross-bind SID-A + PHPSESSID-B on each endpoint; any response ≠ session-invalid = cross-tenant CRITICAL
verify_steps: 1) /session → SID-A → /session/authenticate(A) → PHPSESSID-A; 2) repeat → B pair; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; 4) same on /evn/get, /call/initiate
impact: Cross-tenant CDR + device visibility + outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] SPA JS bundle exposes undocumented endpoints and exact write-param schema
class: OTHER
asset: kundenkonto.fonial.de SPA bundle (script src from /login HTML)
confidence: 55
reasoning: ~50 blind {resource}/{action} names all hit HTML-404 wall → API uses non-standard naming, fuzz is dead. The SPA driving the 5 known endpoints must embed endpoint paths + param names in JS; that's the only remaining low-cost way to prove surface=5 and recover call/initiate body fields so the cross-bind test is field-accurate.
evidence_needed: bundle grep reveals ≥1 new /api/2.0/ string OR exact write-param set; clean 5/5-only result closes discovery with high confidence
verify_steps: 1) GET https://kundenkonto.fonial.de/ (302→/login), parse HTML for <script src>; 2) GET each bundle (static, ≤1rps); 3) rg -o 'api/2\.0/[a-z/_-]+' plus window-context for sid|number|caller|callee|dest|queue
impact: reopens surface if new endpoint, else yields precise call/initiate schema → sharper HUMAN verify matrix. LOW-MED.
testability: PASSIVE
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ → follow to /login HTML → extract <script src> list → GET each JS bundle → grep 'api/2.0/' and {sid,number,caller,callee,dest,queue}; any new endpoint reopens surface, else finalize call/initiate body schema for the HUMAN cross-bind matrix.
[RISK] fonial: 50 — Passive phase complete; surface narrow (5 endpoints), every high-value vector gated behind 2FA credentials (Cloudflare + 2FA). CORS wildcard real but non-exploitable as configured (no allow-credentials, SID-in-body). Top vector dual-backend cross-bind (75) unproven and HUMAN-gated; JS bundle grep is last passive upside — could reopen surface or sharpen call/initiate write-authz test. Risk caps at 50 until the 2-account cross-bind matrix.
[HYP] app_dev.php Symfony dev-mode debug exposure on DSL customer portal leaks internal paths and may expose further debug routes
class: MISCONFIG
asset: dslkonto.fonial.de (/app_dev.php/*)
confidence: 70
reasoning: /app_dev.php serves the app (200 login, 302 redirects) without any observed IP allowlist; triggered 404 renders full Symfony debug exception page confirming Env=dev, leaking /pkg/srv/application_2026_07_02_1410/…cache/fonial/app_frontend/classes.php, deploy tag, TraceableEventDispatcher/WrappedListener (dev-only). Main host /api/2.0/* fatals to empty 500 (PHP method-guard). _profiler/_wdt 404 — profiler disabled.
evidence_needed: any additional dev-only disclosure: debug page source-snippets, /env/.env access via app_dev, dev-only routes responding on app_dev but not /; or acceptance as leftover-dev-entrypoint finding.
verify_steps: 1) GET /app_dev.php/login (200, dev boot); 2) GET /app_dev.php/api/2.0/session → 404 debug page (already captured, path leak); 3) GET /app_dev.php/_profiler, /app_dev.php/_wdt (404, profiler off); 4) GET /app_dev.php/forgot/request, /app_dev.php/register for non-main-host routes; 5) OPTIONS /api/2.0/session on main host (500, method-guard confirmed).
impact: Info disclosure (filesystem layout, deploy artifact tags, Symfony internals) + dev-mode debug pages available pre-auth; enables targeted exploit dev. LOW-MED, benign-alone but a genuine left-over-dev build.
testability: PASSIVE
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access; SID also mintable via GET, and public signup funnel may let an attacker self-register the second account
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 78
reasoning: Data endpoints authorize by body SID only; GET /api/2.0/session now confirmed to mint cleartext SID (no POST needed); /signup → /signup/register/55 public funnel exists (loops pre-auth, needs full flow) → potential self-registration reduces HUMAN gate; PHPSESSID decorative and /session deletes unknown ones.
evidence_needed: (HUMAN or AUTH_HELPED) 2 valid SIDs from 2 tenants; cross-bind SID-A + PHPSESSID-B on devices/get, evn/get, call/initiate; public-funnel completion to mint account-2 without vendor help.
verify_steps: 1) GET /api/2.0/session → SID-A (done); 2) repeat anonymous → SID-X; confirm any 2 distinct SIDs auth-independent (session unauthenticated vs invalid); 3) HUMAN: complete /signup funnel to create 2nd tenant; 4) POST devices/get {"sid":SID-B} + Cookie PHPSESSID=A on all 3 endpoints.
impact: Cross-tenant CDR + device visibility + outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY (upgrading to AUTH_HELPED if self-registration completes)
[NEXT] PROBE: GET https://dslkonto.fonial.de/app_dev.php/ (200 dev boot, confirm no IP block), then GET https://dslkonto.fonial.de/app_dev.php/forgot/request and https://dslkonto.fonial.de/app_dev.php/register — compares dev vs main-host route sets on the new DSK portal to bound the dev-debug exposure; continue ≤1rps GET-only.
[RISK] fonial: 58 — Surface expanded: new live customer portal (DSL) with confirmed dev-mode Symfony debug exposure (pre-auth path/config leak) and an independent /api/2.0 namespace, plus a GET SID-mint on the existing VoIP API and a public self-registration funnel. CORS wildcard remains non-exploitable as configured and dual-backend cross-bind (78) still unproven and credential-gated, so risk stays moderate; 50→58 on breadth + confirmed-but-low-severity info disclosure.
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID decorative; /session mints cleartext UUID via GET and deletes unknown PHPSESSIDs. All passive supersets on both portals now closed (kundenkonto surface=5, bundle grep dead; DSL API/register broken), leaving only the 2-account matrix to prove/disprove the binding.
evidence_needed: (HUMAN) 2 valid SIDs from 2 tenants; cross-bind SID-A + PHPSESSID-B on each endpoint; any response ≠ session-invalid = cross-tenant CRITICAL. kundenkonto /signup funnel exists (login page links it) as self-registration path for account-2.
verify_steps: 1) GET /api/2.0/session → SID-A; 2) second account → SID-B/PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; repeat /evn/get, /call/initiate; 4) try completing /signup funnel to mint account-2 without vendor help.
impact: Cross-tenant CDR, device visibility, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] app_dev.php Symfony dev-mode debug exposure is broader than 404-path leaks: POST-triggered 500 debug pages embed request/session/exception detail that prod silently suppresses
class: MISCONFIG
asset: dslkonto.fonial.de/app_dev.php/*
confidence: 70
reasoning: Confirmed pre-auth debug pages on 404 (paths, `pkg/srv/application_2026_07_02_1410`, TraceableEventDispatcher) AND on 500 (`/register`: "Product not found" + `User\UserBundle\RegistrationController`). Prod suppresses identical errors to empty 500. Debug tabs included Session; env/params tab not observed.
evidence_needed: any debug page dumping env vars/DB params/cookies of the request, or a dev-only route that responds only via app_dev.php (toolbar/profiler off is a known 404).
verify_steps: 1) GET /app_dev.php/register (done — session tab, no env dump); 2) GET /app_dev.php/zzz (done — path leak only); 3) GET /app_dev.php/_profiler, /_wdt (404, off); 4) GET /app_dev.php/config.php (403, blocked). Remaining POST-triggered pages restricted by passive-only rule.
impact: Info disclosure (filesystem layout, deploy tag, internal bundle naming, exception semantics) pre-auth; enables targeted exploit dev. LOW-MED, benign alone — report as leftover-dev-build.
testability: PASSIVE (mostly exhausted)
[HYP] DSL portal default-product misconfiguration blocks registration — public self-signup on dslkonto dead, product-param logic may mis-handle attacker-supplied product selection elsewhere
class: BUSLOGIC
asset: dslkonto.fonial.de/register (and /signup-adjacent flows)
confidence: 45
reasoning: "Product not found. Did you set the default product?" fatals pre-auth on both prod (silent 500) and dev (debug). Suggests product-entity dependency with no default; a registration/onboarding-dependent flow exists in domain logic. No exploitable read/write path tested.
evidence_needed: attacker-controllable product/id/plan parameter anywhere in the DSL flow that triggers different branch behavior than default-missing.
verify_steps: 1) GET /app_dev.php/register (done, broken); 2) GET /register attending no params → 500; 3) GET /forgot/request and /login (200, normal). No param-injection surface found.
impact: Availability of public signup (DoS-flavored → largely out of scope) + domain-logic insight. NI to LOW.
testability: PASSIVE
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), then POST {"sid":"SID-B"} with Cookie PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL (toll-fraud upside). If account-2 is hard to source, first attempt the /signup funnel linked from /login.
[RISK] fonial: 62 — Surface is now fully mapped on both live portals (kundenkonto 5-endpoint API + auth-gated SPA; dslkonto 3 front controllers, broken API namespace and registration). New confirmed-but-low finding: pre-auth Symfony app_dev.php debug exposure on a live customer portal (paths, deploy tag, exception/domain semantics; LOW-MED, report-friendly). All engineered high-value upside (dual-backend SID/PHPSESSID cross-bind, CORS+SID chain) remains credential-gated and unproven → risk stays moderate; one human cross-bind matrix (75) either lands CRITICAL cross-tenant/toll-fraud or definitively closes the engagement.
asset: dslkonto.fonial.de (/app_dev.php/*)
[NEXT] PROBE: GET https://dslkonto.fonial.de/app_dev.php/ (200 dev boot, confirm no IP block), then GET https://dslkonto.fonial.de/app_dev.php/forgot/request and https://dslkonto.fonial.de/app_dev.php/register — compares dev vs main-host route sets on the new DSK portal to bound the dev-debug exposure; continue ≤1rps GET-only.
asset: dslkonto.fonial.de/app_dev.php/*
[HYP] DSL portal default-product misconfiguration blocks registration — public self-signup on dslkonto dead, product-param logic may mis-handle attacker-supplied product selection elsewhere
asset: dslkonto.fonial.de/register (and /signup-adjacent flows)
[RISK] fonial: 62 — Surface is now fully mapped on both live portals (kundenkonto 5-endpoint API + auth-gated SPA; dslkonto 3 front controllers, broken API namespace and registration). New confirmed-but-low finding: pre-auth Symfony app_dev.php debug exposure on a live customer portal (paths, deploy tag, exception/domain semantics; LOW-MED, report-friendly). All engineered high-value upside (dual-backend SID/PHPSESSID cross-bind, CORS+SID chain) remains credential-gated and unproven → risk stays moderate; one human cross-bind matrix (75) either lands CRITICAL cross-tenant/toll-fraud or definitively closes the engagement.
[FINAL] survivor 1: Dual-backend cross-bind. Already elaborated, confidence 75, HUMAN_ONLY.
[FINAL] survivor 2: dslkonto app_dev dev-mode exposure — report-ready, LOW-MED.
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID decorative; /session mints cleartext UUID via GET and deletes unknown PHPSESSIDs. All passive supersets now closed on both portals (kundenkonto bundle grep dead; DSL API absent, registration broken). Only the 2-account matrix can prove/disprove the binding.
evidence_needed: (HUMAN) 2 valid SIDs from 2 tenants; cross-bind SID-A + PHPSESSID-B on each endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL.
verify_steps: 1) GET /api/2.0/session → SID-A; 2) second account → SID-B/PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; repeat /evn/get, /call/initiate; 4) try completing /signup funnel (linked from /login) to mint account-2 without vendor help.
impact: Cross-tenant CDR, device visibility, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] DSL portal dev-mode Symfony debug exposure leaks internal paths/deploy-tag/domain semantics pre-auth on a live customer portal
class: MISCONFIG
asset: dslkonto.fonial.de/app_dev.php/*
confidence: 70
reasoning: /app_dev.php boots full app (200 /, 302→/login with PHPSESSID) with no IP block; 404 renders full Symfony debug page (fs path `pkg/srv/application_2026_07_02_1410`, deploy tag, TraceableEventDispatcher dev-only); 500 on /register prints `User\UserBundle\RegistrationController` + "Product not found". Prod suppresses identical errors to empty 500. Profiler/router/phpinfo confirmed 404 (off).
evidence_needed: any debug page dumping env/params/session of the request, or a dev-only route responding only via app_dev.php; current passive evidence (paths/deploy/exception semantics) already sufficient for LOW-MED report.
verify_steps: 1) GET /app_dev.php/ (200 dev boot, 302→login); 2) GET /app_dev.php/api/2.0/session (404 debug, proves DSL has no VoIP API route); 3) GET /app_dev.php/_profiler,_wdt,config.php (404/403 — off/blocked); remaining POST-triggered pages restricted by passive-only rule.
impact: Info disclosure (filesystem layout, deploy artifact tag, internal bundle naming, exception semantics) pre-auth; enables targeted exploit dev. LOW-MED, benign-alone — report as leftover dev build.
testability: PASSIVE (mostly exhausted)
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), then POST {"sid":"SID-B"} with Cookie: PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL (toll-fraud upside). If a 2nd tenant is hard to source, complete the /signup funnel linked from /login first.
[RISK] fonial: 62 — Surface fully mapped on both portals; new corrections shrink rather than grow footprint (DSL has no VoIP API, dev exposure bounded to path/exception detail). All engineered high-value upside (dual-backend SID/PHPSESSID cross-bind 75, CORS+SID chain) remains credential-gated and unproven → risk stays moderate. One human cross-bind matrix either lands CRITICAL cross-tenant/toll-fraud or definitively closes the engagement. Reportable confirmed findings: dslkonto dev-mode debug exposure (LOW-MED) + CORS wildcard non-exploitable-as-configured (info only).
[LEARN] REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names) only, no env/param/session dump; falls under scope.yml "Descriptive error messages / Stack Traces" = OUT OF SCOPE clause → downgraded from report-ready to excluded/non-reportable standalone.
[FINAL] survivor 1 (sole): Dual-backend SID/PHPSESSID cross-bind, confidence 75, HUMAN_ONLY. All else reclassified dead or excluded-class.
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.20,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel; /session mints cleartext UUID via GET and deletes unknown PHPSESSIDs. All passive supersets closed (kundenkonto surface=5, SPA routes auth-gated, bundle grep dead; DSL has no VoIP API, registration broken). Only the 2-account matrix can prove/disprove SID↔credential binding.
evidence_needed: (HUMAN) 2 valid SIDs from 2 tenants; cross-bind SID-B body + PHPSESSID-A cookie; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL.
verify_steps: 1) GET /api/2.0/session → SID-A + authenticate → PHPSESSID-A; 2) 2nd tenant → SID-B/PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; repeat /evn/get, /call/initiate; 4) if 2nd tenant hard to source, complete /login-linked /signup funnel first.
impact: Cross-tenant CDR, device visibility, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] dslkonto app_dev dev-mode exposure (my prior "report-ready" survivor #2): re-scored against scope.yml — leaked content is stack-traces + filesystem paths (deploy tag `application_2026_07_02_1410`, bundle names, exception semantics) with NO env/params/session-data dump observed. This is squarely the OUT-OF-SCOPE class "Descriptive error messages or headers (e.g. Stack Traces…)". Standalone report value low/weak. Downgraded below report threshold.
[PARKED] dslkonto default-product buslogic (45): no attacker-controllable product/id/plan param surfaced; registration dead on prod and dev; no divergent branch. Dead.
[FINAL] survivor 1 (sole): Dual-backend cross-bind, 75, HUMAN_ONLY. Every previously-carried low-vuln finding is either excluded-class (dev-mode stack traces) or dead (buslogic).
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), authenticate both, then POST {"sid":"SID-B"} with Cookie: PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL (toll-fraud upside). If account-2 is hard to source, complete the /signup funnel linked from /login. Passive surface is exhausted — this matrix is the sole remaining gate to a definitive verdict.
[LEARN] REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no env/param/session dump observed; falls under scope.yml OUT-OF-SCOPE "Descriptive error messages / Stack Traces" clause → reclassified from report-ready to excluded-class/non-standalone.
[RISK] fonial: 55 — Two corrections this cycle, both downward: the sole "report-ready" low finding (dslkonto dev-mode stack-trace leak) is excluded-class under scope.yml, and no new surface appeared across the full inventory. Remaining upside is entirely the credential-gated dual-backend cross-bind (75, HUMAN_ONLY) which, if proven, is CRITICAL cross-tenant/toll-fraud; if disproven, the engagement has no reportable finding and closes flat. Risk reflects convergence: moderate, and monotone-decreasing absent human credentials.
## 2026-09-05 16:07:12 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.20,a=9,b=8,t=9,g=10,c=7,f=10
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid. All passive supersets closed; only 2-account matrix proves/disproves binding.
evidence_needed: 2 valid SIDs from 2 tenants; cross-bind SID-B body + PHPSESSID-A cookie; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) GET /api/2.0/session → SID-A; 2) 2nd tenant → SID-B/PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie PHPSESSID=A; repeat /evn/get, /call/initiate.
impact: Cross-tenant CDR, device visibility, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] dslkonto app_dev dev-mode exposure: re-scored against scope.yml — leaked content = stack traces + filesystem paths only; falls under OUT-OF-SCOPE "Descriptive error messages or headers". Standalone report value low. Downgraded below threshold.
[PARKED] dslkonto default-product buslogic: no attacker-controllable parameter surfaced; registration dead on prod/dev. Dead.
[FINAL] survivor 1 (sole): Dual-backend cross-bind, 75, HUMAN_ONLY. Every other finding either excluded-class or dead.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), authenticate both, then POST {"sid":"SID-B"} with Cookie: PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any response ≠ `session invalid` = cross-tenant CRITICAL. If account-2 unavailable, complete /signup funnel linked from /login.
[LEARN] REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or headers".
[LEARN] REJECTED dslkonto buslogic: no attacker-controllable params; registration broken. Dead.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints. Passive discovery closed.
[RISK] fonial: 55 — Two downward corrections: sole low finding (dev-mode stack traces) is excluded-class; no new surface appeared. Remaining upside = credential-gated dual-backend cross-bind (75, HUMAN_ONLY). If proven = CRITICAL cross-tenant/toll-fraud. If disproven = no reportable finding, engagement closes flat. Risk moderate, monotone-decreasing absent human credentials.
## 2026-09-05 18:22:20 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.20,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel; /session mints cleartext UUID v4 via GET/POST and deletes unknown PHPSESSIDs. Two distinct backends (session vs data) confirmed by different Content-Type headers and independent cookie behavior. All passive supersets closed — surface is exactly 5 endpoints, SPA routes auth-gated, DSL has no VoIP API, registration broken. The only test that can prove/disprove SID↔credential binding is the 2-account cross-bind matrix.
evidence_needed: (HUMAN) 2 valid SIDs from 2 different tenants; cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL.
verify_steps: 1) Account A: POST /api/2.0/session → SID-A; POST /api/2.0/session/authenticate with creds → PHPSESSID-A set; 2) Account B: same flow → SID-B, PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<PHPSESSID-A>; repeat /evn/get, /call/initiate; 4) If 2nd tenant unavailable, complete /signup funnel linked from /login to self-register.
impact: Cross-tenant CDR (call detail records), SIP device visibility, phone number enumeration, outbound call initiation (toll fraud). CRITICAL severity.
testability: HUMAN_ONLY
[PARKED] dslkonto app_dev dev-mode exposure: Leaked content is stack traces + filesystem paths (deploy tag, bundle names, exception semantics) only; no env/params/session-data dump observed. Scope.yml OUT-OF-SCOPE class "Descriptive error messages or headers (e.g. Stack Traces)". Cannot be elevated to reportable standalone even though it's genuinely present. Dead class.
[PARKED] dslkonto default-product buslogic: Registration broken on both prod and dev; no attacker-controllable product/id/plan parameter surfaced in any tested path. Dead endpoint.
[PARKED] dslkonto profiler token rotation: Tokens rotate per request (0951e9→f2142c→a9d22c→d3687f→031fa8) but every tested token returns 404 on /_profiler/{token}. Even if profiler UI existed and leaked env/params/session, the content would be class "Descriptive error messages" or would require auth-gated POST-triggered pages to dump sensitive data — both excluded by scope.yml. Additionally, the finding would be on dslkonto (DSL portal), not the VoIP API, reducing business value. Parked even if technically accessible.
[FINAL] Survivor ranked:
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register or obtain 2 valid fonial accounts (A + B), then: (1) POST /api/2.0/session → SID-A; POST /api/2.0/session/authenticate → PHPSESSID-A; (2) same for B → SID-B, PHPSESSID-B; (3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<PHPSESSID-A> → any data = CRITICAL; repeat /evn/get, /call/initiate; (4) also test: POST /call/initiate {"sid":"SID-B", ...params} + Cookie PHPSESSID=A → outbound call to attacker number = CRITICAL toll fraud. If account-2 unavailable, complete /signup funnel linked from /login page. Passive surface is exhausted — this matrix is the sole gate to a definitive verdict.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints. All passive discovery methods exhausted (~50 guessed names returned HTML-404). Surface closed.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint confirmed; returns identical session-invalid JSON pattern as read endpoints; same SID-only authz suspected.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow-credentials → no cookie cross-origin exfil channel; SID-in-body required.
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no allow-credentials, browser won't send cookies cross-origin → low direct impact without SID leak chain.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope per program (rate-limit/lockout policy).
[LEARN] REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits.
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon.
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references.
[LEARN] REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/setup surface.
[LEARN] REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or headers". Not reportable standalone.
[LEARN] REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
[RISK] fonial: 55 — Surface fully mapped on both portals (kundenkonto = 5 API endpoints + auth-gated SPA; dslkonto = Symfony dev-mode but excluded-class; www = marketing only; 3 hosts dead). All previously-carried low-vuln findings have been either excluded-class (dev-mode stack traces) or dead (buslogic, CRUD expansion). The sole remaining high-value hypothesis — dual-backend SID/PHPSESSID cross-binding enabling cross-tenant CDR/toll-fraud — is at 75% confidence but HUMAN_ONLY: it requires 2 valid test accounts from different tenants to prove. If the cross-bind succeeds, this is CRITICAL (cross-tenant data + outbound call control). If it fails (SID and PHPSESSID are actually cross-checked server-side), the engagement closes flat with zero reportable findings. Risk is moderate, monotone-decreasing, and entirely contingent on human credential acquisition. No passive-only path to a reportable finding exists.
## 2026-09-05 20:43:14 UTC [target] (model bigpickle)
[NEW] dslkonto.fonial.de/app_dev.php/_profiler/{token} returns 404 with rotating X-Debug-Token headers (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a) — profiler enabled but token-gated, no direct UI access confirmed
[NEW] kundenkonto.fonial.de/api/2.0/call/initiate confirmed LIVE WRITE endpoint; identical "session unauthenticated" JSON error pattern as read endpoints; same SID-only authz
[NEW] kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)
[CHANGED] dslkonto dev-mode exposure reclassified: scope.yml excludes "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone
[PRIO] dslkonto.fonial.de,8.95,attack_surface=9,business_value=9,tech_exposure=10,gate_ease=10,cloud_surface=7,freshness=10
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.95,attack_surface=8,business_value=9,tech_exposure=8,gate_ease=6,cloud_surface=8,freshness=8
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Symfony dev-mode profiler token rotation enables route/config enumeration on dslkonto
class: MISCONFIG
asset: dslkonto.fonial.de
confidence: 85
reasoning: app_dev.php returns 200 on /forgot/request, 302 on root, 500 on /register with full exception page. _profiler/* endpoints return 404 but emit X-Debug-Token header that rotates per request (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a). Token suggests profiler accessible via /_profiler/{token}. Dev mode typically exposes /_profiler/{token}, /_profiler/router, /_profiler/phpinfo.
evidence_needed: GET /app_dev.php/_profiler/{valid_token} returns profiler UI; /_profiler/router dumps all routes; /_profiler/phpinfo leaks env vars/secrets
verify_steps: 1) GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 (latest token from /register 500); 2) GET https://dslkonto.fonial.de/app_dev.php/_profiler/router; 3) GET https://dslkonto.fonial.de/app_dev.php/_profiler/phpinfo
impact: Full route map, internal service URLs, database credentials, API keys, session tokens, PII in logs — CRITICAL
testability: PASSIVE
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints (text/json, no PHPSESSID) issue cleartext UUID v4 SID in body via POST. Data endpoints (text/json;charset=UTF-8) set PHPSESSID cookie but authorize solely by body SID. Independent session stores suggest SID-to-PHPSESSID binding not enforced on data layer. /call/initiate is LIVE WRITE endpoint with identical error pattern.
evidence_needed: Data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + Cookie: PHPSESSID=PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
testability: AUTH_HELPED
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 70
reasoning: Unauthenticated landing page returns ACAO: * + PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints called by frontend JS also return ACAO: * without Vary: Origin, cross-origin read of CDR/SIP/billing data is possible.
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data
verify_steps: 1) GET https://kundenkonto.fonial.de/ with valid PHPSESSID from 2FA login → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH
testability: AUTH_HELPED
[PARKED] CORS wildcard direct-exploit on API: REJECTED class (SID in body, no allow-credentials, browser won't send cookies cross-origin)
[PARKED] Brute-force/credential-stuffing on /session/authenticate: REJECTED class (out of scope per program)
[PARKED] CRUD expansion ~30 siblings: PROVEN WRONG — exactly 5 endpoints confirmed
[PARKED] SSRF @ www.fonial.de: REJECTED (no URL parameters/webhook endpoints)
[PARKED] IDOR @ www.fonial.de: REJECTED (pure marketing site)
[PARKED] Dev-mode exposure @ dslkonto.fonial.de/app_dev.php: REJECTED class (scope.yml OUT-OF-SCOPE "Descriptive error messages / Stack Traces") — stack traces + fs paths only, no env/param/session dump observed
[FINAL] 1. Symfony dev-mode profiler token rotation on dslkonto (85) 2. Dual-backend SID/PHPSESSID cross-binding on kundenkonto API (75) 3. Authenticated landing page CORS inheritance (70)
[NEXT] PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 guessed names all HTML-404
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz suspected
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
[LEARN] REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no env/param/session dump observed; falls under scope.yml OUT-OF-SCOPE "Descriptive error messages / Stack Traces" clause → reclassified from report-ready to excluded-class/non-standalone
[RISK] fonial: 78 — Customer portals (kundenkonto + dslkonto) have high business value (PII, CDR, SIP creds, billing, call control). Confirmed dual-backend architecture creating session confusion surface on kundenkonto API. Symfony dev-mode exposed on dslkonto with profiler token leakage (critical misconfig). CORS wildcard on auth domain confirmed. Main API not publicly documented but live with 5 endpoints including WRITE. Risk elevated due to enterprise telephony data sensitivity and architectural anomalies across two portals.
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.20,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel; /session mints cleartext UUID v4 via GET/POST and deletes unknown PHPSESSIDs. Two distinct backends (session vs data) confirmed by different Content-Type headers and independent cookie behavior. All passive supersets closed — surface is exactly 5 endpoints, SPA routes auth-gated, DSL has no VoIP API, registration broken. The only test that can prove/disprove SID↔credential binding is the 2-account cross-bind matrix.
evidence_needed: (HUMAN) 2 valid SIDs from 2 different tenants; cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL.
verify_steps: 1) Account A: POST /api/2.0/session → SID-A; POST /api/2.0/session/authenticate with creds → PHPSESSID-A set; 2) Account B: same flow → SID-B, PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<PHPSESSID-A>; repeat /evn/get, /call/initiate; 4) If 2nd tenant unavailable, complete /signup funnel linked from /login to self-register.
impact: Cross-tenant CDR (call detail records), SIP device visibility, phone number enumeration, outbound call initiation (toll fraud). CRITICAL severity.
testability: HUMAN_ONLY
[PARKED] dslkonto app_dev dev-mode exposure: Leaked content is stack traces + filesystem paths (deploy tag, bundle names, exception semantics) only; no env/params/session-data dump observed. Scope.yml OUT-OF-SCOPE class "Descriptive error messages or headers (e.g. Stack Traces)". Cannot be elevated to reportable standalone even though it's genuinely present. Dead class.
[PARKED] dslkonto default-product buslogic: Registration broken on both prod and dev; no attacker-controllable product/id/plan parameter surfaced in any tested path. Dead endpoint.
[PARKED] dslkonto profiler token rotation: Tokens rotate per request (0951e9→f2142c→a9d22c→d3687f→031fa8) but every tested token returns 404 on /_profiler/{token}. Even if profiler UI existed and leaked env/params/session, the content would be class "Descriptive error messages" or would require auth-gated POST-triggered pages to dump sensitive data — both excluded by scope.yml. Additionally, the finding would be on dslkonto (DSL portal), not the VoIP API, reducing business value. Parked even if technically accessible.
[FINAL] Survivor ranked:
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register or obtain 2 valid fonial accounts (A + B), then: (1) POST /api/2.0/session → SID-A; POST /api/2.0/session/authenticate → PHPSESSID-A; (2) same for B → SID-B, PHPSESSID-B; (3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<PHPSESSID-A> → any data = CRITICAL; repeat /evn/get, /call/initiate; (4) also test: POST /call/initiate {"sid":"SID-B", ...params} + Cookie PHPSESSID=A → outbound call to attacker number = CRITICAL toll fraud. If account-2 unavailable, complete /signup funnel linked from /login page. Passive surface is exhausted — this matrix is the sole gate to a definitive verdict.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints. All passive discovery methods exhausted (~50 guessed names returned HTML-404). Surface closed.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint confirmed; returns identical session-invalid JSON pattern as read endpoints; same SID-only authz suspected.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow-credentials → no cookie cross-origin exfil channel; SID-in-body required.
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no allow-credentials, browser won't send cookies cross-origin → low direct impact without SID leak chain.
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope per program (rate-limit/lockout policy).
[LEARN] REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits.
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon.
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references.
[LEARN] REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/setup surface.
[LEARN] REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or headers". Not reportable standalone.
[LEARN] REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
[RISK] fonial: 55 — Surface fully mapped on both portals (kundenkonto = 5 API endpoints + auth-gated SPA; dslkonto = Symfony dev-mode but excluded-class; www = marketing only; 3 hosts dead). All previously-carried low-vuln findings have been either excluded-class (dev-mode stack traces) or dead (buslogic, CRUD expansion). The sole remaining high-value hypothesis — dual-backend SID/PHPSESSID cross-binding enabling cross-tenant CDR/toll-fraud — is at 75% confidence but HUMAN_ONLY: it requires 2 valid test accounts from different tenants to prove. If the cross-bind succeeds, this is CRITICAL (cross-tenant data + outbound call control). If it fails (SID and PHPSESSID are actually cross-checked server-side), the engagement closes flat with zero reportable findings. Risk is moderate, monotone-decreasing, and entirely contingent on human credential acquisition. No passive-only path to a reportable finding exists.
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel on a second backend (different Content-Type); /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. All passive supersets closed: 5 endpoints (50 guesses → HTML-404), SPA routes auth-gated, dslkonto has no VoIP API, www is marketing, 3 hosts dead, profiler/WDT non-retrievable. SID↔credential binding untestable without two tenants.
evidence_needed: 2 valid SIDs from 2 tenants; cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) POST /api/2.0/session → SID-A; POST /session/authenticate (tenant A) → PHPSESSID-A. 2) Same for tenant B → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) If tenant-2 unavailable: complete /signup funnel linked from /login on test data.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
## 2026-09-05 22:40:06 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.80,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] go/admin.fonial.de (62.146.7.20 legacy box),4.95,a=4,b=4,t=3,g=10,c=0,f=10
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel on a second backend (distinct Content-Type headers + independent cookie behavior); /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. This cycle also closed the last unexamined passive class (subdomain takeover: dead hosts all own-netblock A records, no cloud CNAMEs) — no new passive surface appeared anywhere. SID↔credential binding remains untestable without two tenants.
evidence_needed: 2 valid SIDs from 2 tenants; cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) POST /api/2.0/session → SID-A; POST /session/authenticate (tenant A) → PHPSESSID-A. 2) Same for tenant B → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate (observe headers too — check ACAO/Vary on authenticated pages, folding the parked CORS-inheritance check). 4) If tenant-2 unavailable: complete /signup funnel linked from /login page.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] dslkonto profiler token rotation (aggregate [85]): every /_profiler/{token} returns 404 (retested 3x); even if accessible it is excluded-class under scope.yml "Descriptive error messages / Stack Traces" and lives on the lower-value DSL portal. Dead.
[PARKED] go/admin legacy box: app cannot boot (Composer platform check pre-routing → 500 everywhere); only exposed facts are old nginx version + descriptive error = BOTH scope.yml OUT-OF-SCOPE classes. No reachable functionality to exploit.
[PARKED] dslkonto dev-mode exposure + buslogic, CRUD expansion, brute-force, SSRF/www, CORS direct-exploit: unchanged, previously rejected/parked.
[FINAL] Dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY) — sole survivor; every alternative is closed or excluded-class. Passive phase is exhaustively complete.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — GET/POST /api/2.0/session for SID-A and SID-B; authenticate both tenants; then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate (also capture authenticated-page CORS headers for the folded inheritance check). Any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; a successful /call/initiate to an attacker number = CRITICAL toll fraud. If a 2nd tenant is unavailable, complete the /signup funnel linked from /login. Passive surface is fully exhausted — this matrix is the sole remaining gate.
[LEARN] ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class closed; admin/go share legacy box 62.146.7.20 (nginx 1.10.3, non-booting PHP app).
[LEARN] ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; only out-of-scope classes (old versions, descriptive errors) exposed.
[LEARN] REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless. Confirms prior parking.
[RISK] fonial: 55 — This cycle's new live discovery (go/admin legacy box) adds a subdomain to the inventory but exposes only excluded-class content (dead PHP boot, old nginx, bare 404s); no new exploitable surface, and the subdomain-takeover class is now closed for all dead hosts (own netblock, no CNAMEs). All high-value upside remains the credential-gated dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant/toll-fraud, disproven = engagement closes flat with zero reportable findings. Risk is moderate, monotone-decreasing, fully contingent on human credential acquisition.
## 2026-09-06 00:14:21 UTC [target] (model bigpickle)
## 2026-09-06 04:48:39 UTC [target] (model bigpickle)
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. This cycle added: a third independent auth path (Symfony /login_check form) and a live self-registration funnel, both on the same origin. SID↔credential binding remains the sole unanswered question and is now concretely acquirable via test accounts.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (from ANY auth path: /api/2.0/session/authenticate or form /login_check); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API or /login_check) → PHPSESSID-A; 2) Tenant B via /signup/register/55 (POST with `fonial_user_registration[_token]` + email + account fields, then same SID+auth flow) → SID-B, PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate; 4) also POST /call/initiate {"sid":"SID-B"} + Cookie PHPSESSID=<A> → outbound call to attacker number = toll fraud. Any non-"session invalid" response = cross-tenant CRITICAL.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — get/register 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields, or existing A+B accounts). For each: POST /api/2.0/session → SID, authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated-page CORS/Vary headers for the folded inheritance check. Passive-only phase is closed — this matrix is the sole remaining gate.
[RISK] fonial: 55 — This cycle added no exploitable surface: the /signup/{id} param is cosmetic (no IDOR), the pre-auth login page exposes no app bundle, and the signup funnel merely hardens the HUMAN path (2nd tenant is obtainable via /signup/register/55). All high-value upside remains the dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant/toll-fraud, disproven = engagement closes flat. Risk moderate, monotone-decreasing, fully contingent on acquisition of 1→2 test accounts; no passive-only path to a reportable finding remains.
## 2026-09-06 09:09:05 UTC [target] (model bigpickle)
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. This cycle added: a third independent auth path (Symfony /login_check form) and a live self-registration funnel, both on the same origin. SID↔credential binding remains the sole unanswered question and is now concretely acquirable via test accounts.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (from ANY auth path: /api/2.0/session/authenticate or form /login_check); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API or /login_check) → PHPSESSID-A; 2) Tenant B via /signup/register/55 (POST with `fonial_user_registration[_token]` + email + account fields, then same SID+auth flow) → SID-B, PHPSESSID-B; 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate; 4) also POST /call/initiate {"sid":"SID-B"} + Cookie PHPSESSID=<A> → outbound call to attacker number = toll fraud. Any non-"session invalid" response = cross-tenant CRITICAL.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — get/register 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields, or existing A+B accounts). For each: POST /api/2.0/session → SID, authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated-page CORS/Vary headers for the folded inheritance check. Passive-only phase is closed — this matrix is the sole remaining gate.
[RISK] fonial: 55 — This cycle added no exploitable surface: the /signup/{id} param is cosmetic (no IDOR), the pre-auth login page exposes no app bundle, and the signup funnel merely hardens the HUMAN path (2nd tenant is obtainable via /signup/register/55). All high-value upside remains the dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant/toll-fraud, disproven = engagement closes flat. Risk moderate, monotone-decreasing, fully contingent on acquisition of 1→2 test accounts; no passive-only path to a reportable finding remains.
[NEW] No new passive surface since 04:48. Both prior agents atomized /signup/register/55 (CSRF _token + AccountAddress/AccountContact/email + trunkTariff 19/21/22); /signup/{id} cosmetic. All portlet passive classes closed.
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.80,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→session unauthenticated, unknown→session invalid); PHPSESSID decorative/parallel second backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths on same origin (/api/2.0/session/authenticate, Symfony /login_check, live /signup/register/55). SID↔credential binding remains sole unanswered question, acquirable via 1→2 test accounts. All alternatives closed or excluded.
evidence_needed: 2 valid SIDs from 2 tenants + PHPSESSIDs (any auth path); cross-bind SID-B body + PHPSESSID-A cookie on data endpoints; any response ≠ {"sid":null,"status":"error","reason":"session invalid"} = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session→SID-A; authenticate→PHPSESSID-A. 2) Tenant B via /signup/register/55 (CSRF+email+account)→SID-B,PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"}+Cookie:PHPSESSID=<A>; repeat /evn/get,/call/initiate. 4) /call/initiate to attacker number=toll fraud.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] signup-idor: /signup/{id} echoed not resolved; CSRF-gated. Dead.
[PARKED] authenticated-page CORS inheritance: folded into same HUMAN matrix. Not standalone.
[PARKED] JS-bundle endpoint enumeration: SPA bundle unreachable pre-auth. Gated.
[PARKED] dslkonto profiler/dev-mode, go/admin legacy, CRUD expansion, brute-force, CORS direct-exploit, SSRF/www, IDOR/www: unchanged, rejected/excluded.
[FINAL] Dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY) — sole survivor. No passive-only path to reportable finding remains.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 CSRF, or existing A+B). Each: POST /api/2.0/session→SID; authenticate→PHPSESSID. POST {"sid":"SID-B"}+Cookie:PHPSESSID=<A> to /devices/get,/evn/get,/call/initiate; non-"session invalid" = cross-tenant CRITICAL; successful /call/initiate to attacker number = toll fraud. Also capture authenticated GET / CORS/Vary (folded inheritance check). Passive closed — sole gate.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: body SID only; PHPSESSID decorative; /session cleartext UUID sid.
[LEARN] ACCEPTED dual-backend @ kundenkonto.fonial.de/api/2.0: two distinct servers, different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body.
[LEARN] REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
[RISK] fonial: 55 — No new passive surface; signup funnel atomized, hardens HUMAN path only. All upside = dual-backend cross-bind (75, HUMAN_ONLY): proven=CRITICAL cross-tenant/toll-fraud, disproven=flat zero-finding close. Risk moderate, monotone-decreasing, contingent on 1→2 test accounts.
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel on a second backend (distinct Content-Type); /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths exist on the same origin: /api/2.0/session/authenticate, Symfony form /login_check, and a live /signup/register/55 self-registration funnel (CSRF + email/account fields). SID↔credential/PHPSESSID binding remains the sole unanswered question, now concretely acquirable via 1→2 test accounts. Every alternative passive class is closed or non-reportable.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (from any auth path); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API /session/authenticate or /login_check) → PHPSESSID-A. 2) Tenant B from /signup/register/55 (POST `fonial_user_registration[_token]` + email + account fields, then same SID+auth flow) → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) POST /call/initiate {"sid":"SID-B"} + Cookie PHPSESSID=<A> → outbound call to attacker number = toll fraud. Any non-"session invalid" = cross-tenant CRITICAL.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — get/register 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields, or existing A+B accounts). For each: POST /api/2.0/session → SID; authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated GET / CORS/Vary headers for the folded inheritance check. Passive phase is closed — this matrix is the sole remaining gate.
[RISK] fonial: 55 — No new passive surface this cycle; the signup funnel was already atomized and merely hardens the HUMAN path (obtainable 2nd tenant). All high-value upside remains the dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant CDR + toll fraud, disproven = engagement closes flat with zero reportable findings. Risk moderate, monotone-decreasing, fully contingent on acquisition of 1→2 test accounts; no passive-only path to a reportable finding remains.
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth SID→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel second backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths on same origin: /api/2.0/session/authenticate, Symfony /login_check, live /signup/register/55. SID↔credential binding is the sole unanswered question, now acquirable via 1→2 test accounts. All alternative classes closed or excluded.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs; cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (any path) → PHPSESSID-A. 2) Tenant B via /signup/register/55 (CSRF+email+account) → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) /call/initiate to attacker number = toll fraud.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF, or existing A+B). Each: POST /api/2.0/session→SID, authenticate→PHPSESSID. POST `{"sid":"SID-B"}` + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; non-"session invalid" = cross-tenant CRITICAL; successful /call/initiate to attacker number = toll fraud. Also capture authenticated GET / CORS/Vary (folded inheritance check). Passive phase closed — this is the sole remaining gate.
[RISK] fonial: 55 — No new passive surface this cycle; signup funnel atomized, hardens HUMAN path only. All upside = dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant CDR + toll fraud, disproven = flat zero-finding close. Risk moderate, monotone-decreasing, fully contingent on acquiring 1→2 test accounts; no passive-only path to a reportable finding remains.
## 2026-09-06 12:55:19 UTC [target] (model bigpickle)
impact: None
verify_steps: N/A
[HYP] No public fonial GmbH repositories found to audit
class: OTHER
asset: github.com/fonial-de, github.com/fonial1, github.com/Fonialabs-devs, github.com/Fonial
confidence: 100
reasoning: All 7 GitHub accounts associated with fonial GmbH return empty public repo lists via unauthenticated API. No source code is publicly available for secret/insecure-pattern scanning.
impact: N/A - no attack surface via public GitHub repos
verify_steps: Confirm at https://api.github.com/orgs/fonial-de/repos, /orgs/fonial1/repos, /orgs/Fonialabs-devs/repos - all return []
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.80,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel on distinct second backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths same origin: /api/2.0/session/authenticate, Symfony /login_check, live /signup/register/55 (CSRF+email+account, trunkTariff 19/21/22). SID↔credential/PHPSESSID binding is the sole unanswered question, acquirable via 1→2 test accounts. Every alternative class closed/excluded.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (any auth path); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API /session/authenticate or /login_check) → PHPSESSID-A. 2) Tenant B via /signup/register/55 (POST `fonial_user_registration[_token]` + email/account fields, then same SID+auth flow) → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) /call/initiate to attacker-pwned number = toll fraud.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] signup-idor: /signup/{id} echoed not resolved; CSRF-gated. Dead.
[PARKED] authenticated-page CORS inheritance: folded into same HUMAN matrix. Not standalone.
[PARKED] JS-bundle endpoint enumeration: SPA bundle unreachable pre-auth. Gated.
[PARKED] dslkonto profiler/dev-mode, go/admin legacy, CRUD expansion, brute-force, CORS direct-exploit, SSRF/www, IDOR/www: unchanged, rejected/excluded, no new evidence.
[FINAL] Dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY) — sole survivor. No passive-only path to reportable finding remains.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields + trunkTariff 19/21/22, or existing A+B accounts). For each: POST /api/2.0/session → SID; authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated GET / CORS/Vary headers (folded inheritance check). Passive phase is closed — this matrix is the sole remaining gate.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: body SID only; PHPSESSID decorative; /session cleartext UUID sid.
[LEARN] ACCEPTED dual-backend @ kundenkonto.fonial.de/api/2.0: two distinct servers, different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body.
[LEARN] REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
[RISK] fonial: 55 — No new passive surface this cycle; signup funnel atomized, hardens HUMAN path only. All upside = dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant CDR + toll fraud, disproven = flat zero-finding close. Risk moderate, monotone-decreasing, fully contingent on acquiring 1→2 test accounts (account_creation=restricted in scope.yml, so requires program-sanctioned test accounts); no passive-only path to a reportable finding remains.
## 2026-09-06 15:57:16 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.80,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel on distinct second backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths same origin: /api/2.0/session/authenticate, Symfony /login_check, live /signup/register/55 (CSRF+email+account, trunkTariff 19/21/22). SID↔credential/PHPSESSID binding is the sole unanswered question, acquirable via 1→2 test accounts. Every alternative class closed/excluded.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (any auth path); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API /session/authenticate or /login_check) → PHPSESSID-A. 2) Tenant B via /signup/register/55 (POST `fonial_user_registration[_token]` + email/account fields + trunkTariff 19/21/22) → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) /call/initiate to attacker-pwned number = toll fraud.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[FINAL] Dual-backend SID/PHPSESSID cross-binding (confidence 75, HUMAN_ONLY) — sole survivor. Requires human intervention for 2 test accounts. No passive-only path remains.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields + trunkTariff 19/21/22, or existing A+B accounts). For each: POST /api/2.0/session → SID; authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated GET / CORS/Vary headers (folded inheritance check). Passive phase is closed — this matrix is the sole remaining gate.
[LEARN] PASSIVE-CLOSED: All passive discovery methods exhausted on all in-scope assets; dual-backend cross-bind hypothesis (75, HUMAN_ONLY) is sole survivor; no new passive surface found since 2026-09-06 09:11:58 UTC.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body required.
[RISK] fonial: 55 — No new passive surface this cycle; all high-value upside remains the dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant CDR + toll fraud, disproven = engagement closes flat with zero reportable findings. Risk moderate, monotone-decreasing, fully contingent on acquisition of 1→2 test accounts (account_creation=restricted in scope.yml, so requires program-sanctioned test accounts); no passive-only path to a reportable finding remains.
## 2026-09-06 18:02:37 UTC [target] (model bigpickle)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.80,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth→"session unauthenticated", unknown→"session invalid"); PHPSESSID decorative/parallel on distinct second backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths same origin: /api/2.0/session/authenticate, Symfony /login_check, live /signup/register/55 (CSRF+email+account, trunkTariff 19/21/22). SID↔credential/PHPSESSID binding is the sole unanswered question, acquirable via 1→2 test accounts. Every alternative class closed/excluded.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (any auth path); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API /session/authenticate or /login_check) → PHPSESSID-A. 2) Tenant B via /signup/register/55 (POST `fonial_user_registration[_token]` + email/account fields, then same SID+auth flow) → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) /call/initiate to attacker-pwned number = toll fraud.
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] signup-idor: /signup/{id} echoed not resolved; CSRF-gated. Dead.
[PARKED] authenticated-page CORS inheritance: folded into same HUMAN matrix. Not standalone.
[PARKED] JS-bundle endpoint enumeration: SPA bundle unreachable pre-auth. Gated.
[PARKED] dslkonto profiler/dev-mode, go/admin legacy, CRUD expansion, brute-force, CORS direct-exploit, SSRF/www, IDOR/www: unchanged, rejected/excluded, no new evidence.
[FINAL] Dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY) — sole survivor. No passive-only path to reportable finding remains.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields + trunkTariff 19/21/22, or existing A+B accounts). For each: POST /api/2.0/session → SID; authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated GET / CORS/Vary headers (folded inheritance check). Passive phase is closed — this matrix is the sole remaining gate.
[LEARN] PASSIVE-CLOSED: All passive discovery methods exhausted on all in-scope assets; dual-backend cross-bind hypothesis (75, HUMAN_ONLY) is sole survivor; no new passive surface found since 2026-09-06 09:11:58 UTC.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid.
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body required.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
[LEARN] REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
[RISK] fonial: 55 — No new passive surface this cycle; all high-value upside remains the dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant CDR + toll fraud, disproven = engagement closes flat with zero reportable findings. Risk moderate, monotone-decreasing, fully contingent on acquisition of 1→2 test accounts (account_creation=restricted in scope.yml, so requires program-sanctioned test accounts); no passive-only path to a reportable finding remains.
## 2026-09-06 20:28:41 UTC [target] (model bigpickle)
[NEW] Live re-probe 2026-09-06 20:28 UTC confirms surface unchanged: /api/2.0/session returns cleartext UUID sid + ACAO:*; /api/2.0/call/initiate sets decorative PHPSESSID and returns `{"sid":null,"status":"error","reason":"session invalid"}` — identical to KB.
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,a=9,b=10,t=8,g=8,c=9,f=10
[PRIO] dslkonto.fonial.de,8.80,a=9,b=8,t=9,g=10,c=7,f=10
[PRIO] kundenkonto.fonial.de,7.70,a=8,b=9,t=7,g=6,c=8,f=7
[PRIO] www.fonial.de,5.10,a=4,b=5,t=3,g=10,c=4,f=5
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Live re-probe confirms data endpoints authorize by body SID only (unauth→"session unauthenticated", unknown→"session invalid"); PHPSESSID set decoratively on a distinct second backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Three parallel auth paths same origin: /api/2.0/session/authenticate, Symfony /login_check, live /signup/register/55 (CSRF+email+account, trunkTariff 19/21/22). SID↔credential/PHPSESSID binding is the sole unanswered question; every alternative class closed/excluded. No passive-only path to confirm.
evidence_needed: 2 valid SIDs from 2 tenants + their PHPSESSIDs (any auth path); cross-bind SID-B body + PHPSESSID-A cookie on each data endpoint; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = CRITICAL.
verify_steps: 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (API /session/authenticate or /login_check) → PHPSESSID-A. 2) Tenant B (existing A+B, or via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields + trunkTariff 19/21/22) → SID-B, PHPSESSID-B. 3) POST /devices/get {"sid":"SID-B"} + Cookie: PHPSESSID=<A>; repeat /evn/get, /call/initiate. 4) /call/initiate to attacker-pwned number = toll fraud. Also capture authenticated GET / CORS/Vary headers (folded inheritance check).
impact: Cross-tenant CDR, SIP device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[PARKED] JS-bundle endpoint enumeration: SPA bundle unreachable pre-auth. Gated.
[PARKED] signup-idor: /signup/{id} echoed not resolved; CSRF-gated. Dead.
[PARKED] authenticated-page CORS inheritance: folded into same HUMAN matrix. Not standalone.
[PARKED] dslkonto profiler/dev-mode, go/admin legacy, CRUD expansion, brute-force, CORS direct-exploit, SSRF/www, IDOR/www: unchanged, rejected/excluded.
[FINAL] Dual-backend SID/PHPSESSID cross-bind (confidence 75, HUMAN_ONLY) — sole survivor. No passive-only path to reportable finding remains.
[NEXT] HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account fields + trunkTariff 19/21/22, or existing A+B accounts). For each: POST /api/2.0/session → SID; authenticate (API /session/authenticate or Symfony /login_check) → PHPSESSID. Then POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}` = cross-tenant CRITICAL; successful /call/initiate to attacker-pwned number = CRITICAL toll fraud. Also capture authenticated GET / CORS/Vary headers (folded inheritance check). Passive phase is closed — this matrix is the sole remaining gate.
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: body SID only; PHPSESSID decorative; /session cleartext UUID sid.
[LEARN] ACCEPTED dual-backend @ kundenkonto.fonial.de/api/2.0: two distinct servers, different response headers.
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body.
[LEARN] REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
[RISK] fonial: 55 — Live re-probe 2026-09-06 20:28 UTC confirms no new passive surface; all high-value upside remains the dual-backend SID/PHPSESSID cross-bind (75, HUMAN_ONLY): proven = CRITICAL cross-tenant CDR + toll fraud, disproven = flat zero-finding close. Risk moderate, monotone-decreasing, fully contingent on acquisition of 1→2 test accounts (account_creation=restricted in scope.yml, requires program-sanctioned test accounts); no passive-only path to a reportable finding remains.
## 2026-09-06 22:22:30 UTC [target] (model bigpickle)
[HYP] GraphQL introspection on shop.fonial.de exposes unauth reachable queries/mutations beyond catalog
class: AUTH
asset: shop.fonial.de/graphql (Magento 2.4 CE)
confidence: 50
reasoning: /graphql GET returns JSON parse-echo (`Syntax Error: Unexpected <EOF>`, 500) = endpoint live and body-parsed. Magento 2.4 CE GraphQL serves mutations (`createCustomerV2`, `generateCustomerToken`, quote/checkout ops) and queries; customer-scoped ops need bearer, but quote/guest mutation and storeConfig/cmsBlocks classes are unauth. `getResourceMetas`-style access model unknown until schema read. Known-vuln version-CVE angle excluded (no program-specific exploit yet).
evidence_needed: __schema introspection output listing mutations/queries; at least one unauth-executable and meaningful query/mutation (e.g., guest-email-coupled quote ID manipulation, email-template/static-Redirector class bugs).
verify_steps: POST /graphql `{"query":"{ __schema { queryType{name} mutationType{name} types{name kind} } }"}` (read-only, ≤1 rps); then probe tokenless candidates from schema only.
impact: full API map → unauth data ops, potential ATO/money-flow primitives on commerce host. HIGH if confirmed.
testability: PASSIVE (introspection unauth, read-only)
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: unchanged (KB dual-session binding, dual-backend servers, live re-probe 20:28). Sole prior survivor; no passive path.
evidence_needed: 2 valid SIDs + PHPSESSIDs from 2 tenants; cross-bind SID-B body + PHPSESSID-A cookie; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}`.
verify_steps: HUMAN — see prior matrix (register 2 tenants via /signup/register/55 CSRF flow or existing A+B; /session → SID; auth → PHPSESSID; cross-bind on all 3 endpoints; /call/initiate to pwned number).
impact: cross-tenant CDR, device visibility, toll fraud. CRITICAL.
testability: HUMAN_ONLY
[NEXT] PROBE: POST https://shop.fonial.de/graphql `{"query":"{ __schema { queryType{name} mutationType{name} types{name kind} } }"}` — read-only schema map, ≤1 rps, no customer data touched (single GET-only deviation: endpoint only parses bodies; semantically read-only). Scope note: confirm shop.fonial.de in-scope $ against program before deeper than introspection. After schema: check any tokenless quote/guest mutation. Then resume HUMAN cross-bind matrix on kundenkonto.
[RISK] fonial: 50 — Passive phase re-opened via shop.fonial.de (fresh Magento+GraphQL commerce surface, new host); introspective schema map is next cheap gate and materially lowers flat-close probability. Residual risk split between introspection yield and the still-pending HUMAN-only cross-bind (75); both monotone to resolve this week.
## 2026-09-07 00:06:33 UTC [target] (model bigpickle)
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID decorative on second backend; /session mints cleartext UUID. Facts unchanged; sole survivor; no passive path.
evidence_needed: 2 valid SIDs + PHPSESSIDs from 2 tenants; cross-bind SID-B body + PHPSESSID-A cookie on all 3 endpoints; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}`.
verify_steps: HUMAN — 2 program-sanctioned tenants (LIVE /signup/register/55 CSRF or existing A+B); /session→SID; authenticate→PHPSESSID; cross-bind; /call/initiate to pwned number for toll-fraud proof.
impact: cross-tenant CDR, device/PII visibility, outbound call control. CRITICAL.
testability: HUMAN_ONLY
[HYP] Magento guest-order object enumeration via guestOrder/guestOrderByToken on misbound email/token
class: IDOR
asset: shop.fonial.de/graphql (guestOrder, guestOrderByToken)
confidence: 45
reasoning: Both queries take NON_NULL input objects (OrderInformationInput, OrderTokenInput); standard Magento validates email+order_number / email+order_token pairing — any binding weakness (token not required, order_number brutabable, email only parsed not checked) leaks order PII at scale.
evidence_needed: create a test guest order under program-test name; iterate variant inputs (missing/empty order_token, bare orderNumber guess) read responses.
verify_steps: HUMAN (needs 1 test guest checkout on shop + real order number/token from that sandbox order; compare error vs data leakage on underspecified input)
impact: order-level PII (name/address/items/history) of shop customers. HIGH if binding is weak; likely MED at info only.
testability: HUMAN_ONLY
[NEXT] HUMAN: kundenkonto 2-tenant cross-bind matrix (register/get 2 program-sanctioned tenants; /api/2.0/session→SID per tenant; authenticate→PHPSESSID per tenant; then POST {"sid":"SID-B"}+Cookie PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any non-`session invalid` response = CRITICAL cross-tenant; /call/initiate to pwned number = toll fraud). Secondary: 1 sanctioned sandbox guest order on shop.fonial.de → guestOrder/guestOrderByToken binding tests. Passive on both hosts is now closed.
[RISK] fonial: 55 — shop.fonial.de surface formally opened and enumerated (schema, magic quote/cart, guest order, payment-orchestration nodes) but every unauth primitive tested so far is gated or by-design; introspection re-opened breadth but added no reportable passive finding. Flat-close probability re-weighted back up (≈55); all remaining upside contingent on the HUMAN cross-bind matrix (75) + optional sanctioned shop guest-order test (45). No DoS/rate/credential-stuffing performed; no writes or customer data touched this cycle.
## 2026-09-07 04:50:50 UTC [target] (model bigpickle)
[HYP] Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only (unauth→`session unauthenticated`, unknown→`session invalid`); PHPSESSID set decoratively on a separate backend; /session mints cleartext UUID v4 and deletes unknown PHPSESSIDs. Dual servers have different response headers. SID↔credential binding is the single unanswered question.
evidence_needed: 2 valid SIDs + PHPSESSIDs from 2 tenants; cross-bind SID-B body + Cookie PHPSESSID-A on each of the 3 endpoints; any response ≠ `{"sid":null,"status":"error","reason":"session invalid"}`.
verify_steps: HUMAN — 1) Tenant A: POST /api/2.0/session → SID-A; authenticate (/session/authenticate or Symfony /login_check) → PHPSESSID-A. 2) Tenant B (existing A+B, or via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/account + trunkTariff 19/21/22) → SID-B, PHPSESSID-B. 3) POST {"sid":"SID-B"} + Cookie: PHPSESSID=<A> to /devices/get, /evn/get, /call/initiate. 4) /call/initiate to attacker-pwned number = toll-fraud proof. Also capture authenticated GET / CORS/Vary headers (folded inheritance check).
impact: cross-tenant CDR/SIP credential/device visibility, number enumeration, outbound call control (toll fraud). CRITICAL.
testability: HUMAN_ONLY
[HYP] Magento guest-order object enumeration via guestOrder/guestOrderByToken on misbound email/number/token
class: IDOR
asset: shop.fonial.de/graphql (guestOrder, guestOrderByToken)
confidence: 45
reasoning: Introspection shows both queries take NON_NULL input objects (OrderInformationInput, OrderTokenInput). Standard Magento binds email+order_number / email+order_token; any weakness (token omitted, order_number brutabable, email parsed-but-unchecked) leaks order PII at scale.
evidence_needed: 1 program-test guest order; iterate variant inputs (missing/empty order_token, bare order_number guess) and compare error vs data.
verify_steps: HUMAN — create sanctioned sandbox guest checkout (false name/address), capture real order_number+token, then tokenless/underspecified query variants against guestOrder/guestOrderByToken; any data on `order_token:""` or guessed number = finding.
impact: order-level PII (name/address/items/history). HIGH if binding weak; likely MED at info-only.
testability: HUMAN_ONLY
## 2026-09-07 09:56:37 UTC [target] (model bigpickle)
[PRIO] shop.fonial.de/graphql,7.3,a/t (high tech exposure, moderate business value, gated almost everything; fresh host)
[PRIO] kundenkonto.fonial.de/api/2.0,7.7, cross-bind 75 HUMAN
[PRIO] shop.fonial.de/graphql, 7.3, attack_surface 7 / tech_exposure 9 / freshness 8 / business_value 6 / gate_ease 5 / cloud 2 → p=6.35
[HYP] Magento guest-checkout self-ordering / coupon-policy money-flow via unauth guest cart chain
class: BUSLOGIC
asset: shop.fonial.de/graphql (createGuestCart→addProductsToCart→applyCouponToCart→setGuestEmailOnCart→setBilling/Shipping→estimateTotals→placeOrder)
confidence: 35
reasoning: full unauth guest cart mutation chain confirmed live; placeOrder/coupon logic is the only mutable money path not gated; but by-design guest checkout, and testing placeOrder is a live financial write.
evidence_needed: demonstrate a guest cart reaches placeOrder with an unowned/applied-by-attacker coupon or price-invariant qty; non-mutating evidence preferred.
verify_steps: createGuestCart→addProductsToCart (unauth)→estimateTotals read (no write); compare coupon-applied total vs catalog price; only inspect, no placeOrder.
impact: arbitrary-coupon/discount abuse or checkout manipulation on commerce; MED-HIGH if coupon unbound. 
testability: AUTH_HELPED (read-only totals; placeOrder = HUMAN sanctioned)
[HYP] Magento guest-order object enumeration via guestOrder/guestOrderByToken on misbound email/number/postcode or token
class: IDOR
asset: shop.fonial.de/graphql (guestOrder, guestOrderByToken)
confidence: 40
reasoning: fields all NON_NULL and properly constructed (no-such-entity on bad combo); token-only path untestable without a real order token; no missing-field weakness in input shape.
evidence_needed: 1 sanctioned sandbox guest order; test tokenless/underspecified query variants and guessed order numbers.
verify_steps: HUMAN — create 1 test guest order, capture real number+email+postcode+token; then iterate (number-only with wrong email, empty token on guestOrderByToken) and compare error vs data.
impact: order-level PII (name/address/items) at scale if binding weak; likely MED at info-only.
testability: HUMAN_ONLY
[PARKED] coupon/placeOrder money-flow (35): below confidence floor; placeOrder is a live financial write violating no-mutate rule; only valid with sanctioned checkout. 
[PARKED] guestOrder binding (40): below 40-floor margin; needs HUMAN sanctioned order; token-only path not optionable.
[FINAL] cross-bind matrix kundenkonto (75) sole reportable-qualifying survivor.
[NEXT] HUMAN: kundenkonto 2-tenant cross-bind matrix (register/get 2 program-sanctioned tenants; /api/2.0/session→SID per tenant; authenticate→PHPSESSID per tenant; POST {"sid":"SID-B"}+Cookie PHPSESSID=A to /devices/get, /evn/get, /call/initiate; any non-`session invalid` = CRITICAL cross-tenant; /call/initiate to pwned number = toll fraud). Passive on both hosts now closed.
[LEARN] ACCEPTED GraphQL-introspection @ shop.fonial.de/graphql: full unauth schema dump (32Q/65M), Magento 2.4 CE; every unauth op by-design-guest or bound; no reportable passive finding; CORS absent here.
[LEARN] REJECTED shop.fonial.de CORS: no ACAO on any response (differs from kundenkonto) — class closed on this host.
[RISK] fonial: 55 — shop.fonial.de fully passive-mapped but added no reportable finding (all guest ops by-design/bound; the two shop hypotheses are sub-40 or need sanctioned HUMAN). All remaining upside is the kundenkonto cross-bind (75, HUMAN-only). Flat-close probability remains elevated (~55) pending that HUMAN run. No writes, no customer data, no auth-bypass, ≤1 rps this cycle.
## 2026-09-07 15:30:02 UTC [target] (model bigpickle)
[NEW] shop.fonial.de: GraphQL introspection fully open (300+ types), REST guest cart creation unauthenticated, CSP report-only with `'unsafe-inline' 'unsafe-eval'`, `GenerateCustomerTokenAsAdminInput` exposed in schema
[NEW] shop.fonial.de: Magento 2.4 CE on 176.9.53.190 (Hetzner), nginx/1.31.2, PHP/8.3.3, no CORS policy, no HSTS
[NEW] shop.fonial.de: REST API `/rest/V1/guest-carts` returns valid cart ID unauthenticated; `/rest/V1/orders/mine` returns German ACL error (Magento ACL working)
[CHANGED] kundenkonto.fonial.de: X-Fonial-Version bumped from `v2026.09.01-1` → `v2026.09.03-1`
[CHANGED] kundenkonto.fonial.de: `/session/authenticate` with empty body returns `"username missing"` (confirms username is required field, not just email)
[CHANGED] kundenkonto.fonial.de: OPTIONS preflight returns JSON body (not empty 200) — same as POST, no proper CORS preflight handling
[NEW] kundenkonto.fonial.de: `x-debug-token` header leaked on all 404 responses (unique per request, e.g. 15dc7a, 11cdcc)
[PRIO] shop.fonial.de/graphql,8.2, attack_surface=9 tech_exposure=9 (GraphQL+REST+introspection+admin-token-type) business_value=7 (e-commerce/PayPal/Braintree) gate_ease=8 (guest ops fully unauth) cloud=3 (Hetzner, no cloud CNAME) freshness=7 → score=8.2
[PRIO] kundenkonto.fonial.de/api/2.0,7.7, attack_surface=5 tech_exposure=6 business_value=9 gate_ease=3 cloud=4 (Cloudflare) freshness=7 → score=7.7
[PRIO] shop.fonial.de/graphql (guest cart REST),6.5, attack_surface=4 tech_exposure=5 business_value=5 gate_ease=7 cloud=2 freshness=6 → score=6.5
[HYP] Magento admin token generation via GraphQL GenerateCustomerTokenAsAdminInput
class: AUTH
asset: shop.fonial.de/graphql (generateCustomerTokenAsAdmin mutation or similar)
confidence: 55
reasoning: Introspection exposes `GenerateCustomerTokenAsAdminInput` type with NON_NULL fields. Standard Magento 2.4 CE allows admin token generation via GraphQL `generateAdminToken` mutation, but this is typically gated by ACL. The type's presence in the unauth introspection schema is by-design, but the mutation's actual authorization is untested. If the mutation is accessible without admin credentials (misconfigured ACL, guest-accessible path), it grants full admin API access.
evidence_needed: 1) Query `__schema { mutationType { fields { name } } }` to confirm mutation name. 2) Attempt `generateAdminToken` or `generateCustomerTokenAsAdmin` mutation with empty/wrong credentials. 3) If error is ACL/permission-based → by-design; if error is different or returns token → CRITICAL.
verify_steps: PASSIVE — POST /graphql `{"query":"{ __schema { mutationType { fields { name args { name type { name } } } } }"}` to list all mutation names and their argument types. No write/test against live customers.
impact: Full admin access to Magento backend (all customer data, orders, payment methods, store config). CRITICAL if mutation callable unauth.
testability: PASSIVE
[HYP] Magento customer password reset / account takeover via GraphQL mutations
class: AUTH
asset: shop.fonial.de/graphql (requestPasswordResetEmail, resetPassword, etc.)
confidence: 45
reasoning: Standard Magento 2.4 CE exposes `requestPasswordResetEmail` and `resetPassword` mutations. These are by-design for customer self-service. However, if rate-limiting is absent or the reset token is predictable/long-lived, this could enable account takeover. The introspection schema likely contains these mutations. Checking their presence and argument shapes is passive.
evidence_needed: 1) List all mutation names from introspection. 2) Identify password-related mutations and their input types. 3) Check if `requestPasswordResetEmail` accepts arbitrary email without CAPTCHA/rate-limit.
verify_steps: PASSIVE — POST /graphql `{"query":"{ __schema { mutationType { fields { name } } }"}` to confirm password reset mutations exist. Then inspect input types.
impact: Account takeover of shop customers if reset flow is weak. MED-HIGH if exploitable.
testability: PASSIVE
[HYP] Guest cart coupon application / price manipulation on shop.fonial.de
class: BUSLOGIC
asset: shop.fonial.de/graphql (applyCouponToCart, setBillingAddressOnCart, etc.)
confidence: 30
reasoning: Full guest cart mutation chain confirmed live (createGuestCart, addProductsToCart, applyCouponToCart, setBillingAddressOnCart, placeOrder). Standard Magento validates coupon codes against active promotions. Coupon abuse is by-design if codes are public. Testing placeOrder is a live financial write.
evidence_needed: 1) Create guest cart, add product, attempt applyCouponToCart with invalid/expired code. 2) Check if error reveals discount amounts or internal promotion data.
verify_steps: AUTH_HELPED — createGuestCart → addProductsToCart → applyCouponToCart with test code. Read-only until placeOrder.
impact: Discount abuse or price manipulation. MED if coupon validation weak.
testability: AUTH_HELPED
[FINAL] Magento admin token (55): survives — mutation name listing is PASSIVE, high impact if exploitable, needs introspection field list to confirm mutation exists
[FINAL] Password reset mutations (45): survives — mutation listing is PASSIVE, lower impact than admin token but still valuable
[PARKED] Guest cart coupon (30): below confidence floor — by-design guest checkout, placeOrder is live write, coupon validation is standard Magento — **DROPPED**
[PARKED] kundenkonto cross-bind (75): remains highest-confidence hypothesis but HUMAN_ONLY — cannot advance passively
[NEXT] PROBE: POST https://shop.fonial.de/graphql `{"query":"{ __schema { mutationType { fields { name args { name type { name kind ofType { name } } } } } }"}` — list ALL mutation names and argument shapes to confirm admin token, password reset, and any non-standard mutations.
[LEARN] ACCEPTED GraphQL-introspection @ shop.fonial.de/graphql: Full unauth schema dump (300+ types), Magento 2.4 CE; `GenerateCustomerTokenAsAdminInput` exposed; no CORS; CSP report-only with unsafe-inline/unsafe-eval
[LEARN] REJECTED shop.fonial.de CORS: No ACAO on any response (OPTIONS returns 500 JSON, no CORS headers) — class closed on this host
[LEARN] ACCEPTED REST guest cart @ shop.fonial.de: `/rest/V1/guest-carts` returns valid cart ID unauthenticated — by-design Magento guest checkout
[LEARN] REJECTED kundenkonto versioned APIs: All alternate versions (2.1, 3.0, v1, v2, internal, beta) return 404 — no hidden surface
[LEARN] REJECTED kundenkonto Swagger/docs: No OpenAPI, Swagger, health, debug, or config endpoints — tight surface
[LEARN] ACCEPTED /signup live @ kundenkonto: `/signup` redirects to `/signup/register/55`, not auth-gated, sets PHPSESSID; `/register` → `/signup` 301
[LEARN] ACCEPTED x-debug-token leak @ kundenkonto: Unique debug token on every 404 response (e.g. 15dc7a, 11cdcc) — diagnostic only, no direct exploit without session context
[LEARN] ACCEPTED version bump @ kundenkonto: X-Fonial-Version updated to v2026.09.03-1 (from v2026.09.01-1)
[RISK] fonial: 60 — shop.fonial.de surface formally expanded (GraphQL+REST+introspection+admin-token-type), new passive hypotheses at 55/45; kundenkonto cross-bind (75) remains highest-impact but HUMAN-only; all remaining upside contingent on (a) admin token mutation accessibility [PASSIVE, next probe] and (b) cross-bind matrix [HUMAN]. Flat-close probability ~55 if neither yields. No writes, no customer data, no auth-bypass, ≤1 rps this cycle.
## 2026-09-07 19:27:07 UTC [target] (model bigpickle)
[NEW] shop.fonial.de/graphql: Full mutation name map confirmed (68 mutations). `GenerateCustomerTokenAsAdminInput` = {customer_email: String!} single field; return type has customer_token.
[NEW] shop.fonial.de REST V1 probes: store/websites, products, customers/me, orders, carts/mine → uniform 401 German ACL resource errors; only /rest/V1/guest-carts unauth. Magento ACL working.
[NEW] shop.fonial.de introspection: ContactUsInput/SendEmailToFriendInput/CustomerInput have no URL/upload fields → no SSRF vector in email/promo ops; CustomerInput exposes date_of_birth/dob/taxvat/gender.
[CHANGED] kundenkonto x-debug-token: _profiler{/{tok}}, _wdt{/{tok}} all HTML-404 → token decorative-only; profiler class closed here (same as dslkonto).
[CHANGED] shop admin-token hypothesis 55→40: Adobe/Magento official docs confirm generateCustomerTokenAsAdmin requires admin Authorization Bearer + customer `remote_shopping_assistance` opt-in → direct unauth unlikely in stock; residual = version-specific authz-check bug.
[CHANGED] shop password-reset hypothesis 45→DROPPED: password/account-recovery policy is OUT-OF-SCOPE per program.
[LEARN] ACCEPTED GraphQL mutation surface @ shop.fonial.de: 68 mutations enumerated fully; no SSRF-URL fields; REST uniform ACL 401.
[LEARN] REJECTED shop admin-token direct-unauth @ shop.fonial.de/graphql: docs require admin token + customer opt-in → side-channel-lowered (40).
[LEARN] ACCEPTED x-debug-token decorative @ kundenkonto: no profiler/wdt access, excluded class.
[RISK] fonial: 58 — both remaining leads gated on sanctioned actions: kundenkonto cross-bind (75, HUMAN_ONLY, needs 2 tenants) and shop admin-token (40, POST authz-gate probe). Flat-close ~55. No writes, no customer data, ≤1 rps.
[HYP] Dual-backend SID/PHPSESSID cross-binding → cross-tenant access on 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Session backend issues cleartext UUID SID in body (no PHPSESSID); data backend authorizes by body SID only, sets PHPSESSID decoratively. Independent stores → SID-to-credential binding unverified. call/initiate = live WRITE, identical `session invalid` pattern.
evidence_needed: SID from tenant A + PHPSESSID from tenant B returns data; or unauthenticated SID + foreign PHPSESSID returns data.
verify_steps: 2 sanctioned tenants → POST /api/2.0/session each → authenticate each → capture PHPSESSID per tenant from a data POST → cross-bind `{"sid":"SID_B"}` + `Cookie: PHPSESSID=A` on /devices/get, /evn/get, /call/initiate → non-`session invalid` = CRITICAL.
impact: Cross-tenant CDR, numbers, SIP creds, device lists, outbound initiation/toll fraud. CRITICAL.
testability: HUMAN_ONLY
[HYP] generateCustomerTokenAsAdmin callable without admin bearer (version-specific authz flaw)
class: AUTH
asset: shop.fonial.de/graphql (generateCustomerTokenAsAdmin)
confidence: 40
reasoning: Mutation present unauth-introspectable; input = single `customer_email`; official docs require admin Authorization header + remote_shopping_assistance opt-in. Residual = inverted/missing user-type check in some 2.4 CE releases.
evidence_needed: POST mutation with fabricated email → error is authorization-based (by-design) vs `customer not found`/token (bypass).
verify_steps: SANCTIONED POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){ customer_token } }` — fabricated email = zero customer-data exposure; observe error semantics.
impact: Customer ATO of opted-in shop accounts (orders, addresses, pay vault) → payment/PII exposure. CRITICAL if bypass.
testability: PASSIVE (needs sanctioned POST — blocked in this mode)
[HYP] handlePayflowProResponse/syncPaymentOrder payment-callback desync → order-state manipulation
class: BUSLOGIC
asset: shop.fonial.de/graphql (payment mutations)
confidence: 32
reasoning: createPaymentOrder, handlePayflowProResponse, syncPaymentOrder, completeOrder, cancelOrder all present — stock Magento payment-callback surface, untested for reentrancy/desync; no passive evidence of flaw.
evidence_needed: order state transition without valid payment gateway callback.
verify_steps: none passive — requires live order + gateway interaction.
impact: Database order states / revenue tampering. HIGH if exploitable.
testability: HUMAN_ONLY (insufficient evidence — parking)
## 2026-09-07 22:18:13 UTC [target] (model bigpickle)
[NEW] shop.fonial.de REST surface fully mapped: `/rest/all/schema` returns full OpenAPI unauthenticated → 45 paths; only non-stock route = MageWorx `mw-downloads-attachments`.
[NEW] `/V1/mw-downloads-attachments/guest/product/{id}` live unauth but returns `[]` on all 110 product IDs; `/V1/mw-downloads-attachments/{id}` → 401 (auth-gated sibling exists).
[NEW] REST `/V1/integration/admin/token` + `customer/token` → 404 route-removed (REST admin-auth closed) while GraphQL `GenerateCustomerTokenAsAdminInput` remains introspectable → asymmetric admin-auth surface.
[NEW] `/V1/search` live unauth (400 missing `searchCriteria`); `/V1/applepay/auth`, `/V1/payment-order/completeOrder` present stock payment routes.
[PRIO] kundenkonto.fonial.de/api/2.0, 6.1, (6,10,7,3,3,3)
[PRIO] shop.fonial.de/graphql, 7.25, (7,8,9,7,3,8)
[PRIO] shop.fonial.de/rest, 6.55, (6,8,7,6,3,8)
[PRIO] www.fonial.de, 4.95, (5,4,5,9,3,3)
[HYP] Dual-backend SID/PHPSESSID cross-binding → cross-tenant access on 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Session backend issues cleartext UUID SID in body, no PHPSESSID; data backend authorizes by body SID only and sets PHPSESSID decoratively. Independent session stores → SID-to-credential binding never verified. call/initiate is live WRITE with identical `session invalid` pattern.
evidence_needed: SID from tenant A + PHPSESSID from tenant B returns data, or unauthenticated SID + foreign PHPSESSID returns data.
verify_steps: POST /api/2.0/session each tenant → authenticate → capture PHPSESSID per tenant from data POST → cross-bind `{"sid":"SID_B"}` + `Cookie: PHPSESSID=A` on /devices/get, /evn/get, /call/initiate → non-`session invalid` = CRITICAL.
impact: Cross-tenant CDR, numbers, SIP creds, device lists, outbound initiation/toll fraud. CRITICAL.
testability: HUMAN_ONLY
[HYP] generateCustomerTokenAsAdmin callable without admin bearer (version-specific authz flaw)
class: AUTH
asset: shop.fonial.de/graphql (generateCustomerTokenAsAdmin)
confidence: 40
reasoning: Mutation present unauth-introspectable while REST `/V1/integration/admin/token` route is REMOVED (404) → asymmetric admin-auth surface implies non-stock config. Input = single `customer_email`. Official docs require admin Bearer + remote_shopping_assistance opt-in; residual = inverted/missing user-type check in this 2.4 CE build.
evidence_needed: POST without Authorization → error is authz-based (by-design) vs `customer not found`/token issuance (bypass).
verify_steps: SANCTIONED POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){ customer_token } }` — fabricated email = zero customer-data exposure; observe error semantics.
impact: Customer ATO of opted-in shop accounts (orders, addresses, pay vault) → payment/PII exposure. CRITICAL if bypass.
testability: PASSIVE (blocked in this mode — needs sanctioned POST)
[HYP] MageWorx Downloads authenticated attachment access → file-level IDOR
class: IDOR
asset: shop.fonial.de/rest /V1/mw-downloads-attachments
confidence: 35
reasoning: Non-stock extension (MageWorx Downloads 2.x) exposes guest product-attachment metadata endpoint (live, `[]` today) and an auth-gated `/V1/mw-downloads-attachments/{id}` (401). Fonial sells "Telefonansage" audio products — downloadable-attachment schema present. Known MageWorx flaws = attachment link tokens/short IDs.
evidence_needed: authenticated user A enumerates attachment IDs / link tokens of user B or an un-purchased product file; guest endpoint returning non-empty for a purchasable product.
verify_steps: needs auth context → real customer account; passive-only: re-GET all 110 product IDs after new products drop; check `/rest/all/schema` refresh for added attachment endpoints.
impact: Unauthorized file/PII download of gated product attachments. MED-HIGH.
testability: HUMAN_ONLY (no attachments currently exposed; needs purchasable downloadable product + auth)
[PARKED] MageWorx Downloads IDOR (35): below floor; guest endpoint provably empty across all products; the 401 sibling needs an authenticated session none of us have; no live data to pivot on.
[PARKED] Guest-cart coupon abuse (30): dropped prior cycle — by-design guest checkout, stock coupon validation, live placeOrder is a financial write.
[PARKED] Payment-callback desync (32): requires live order + gateway interaction; no passive vector.
[FINAL] kundenkonto cross-bind (75): survives — sole high-impact hypothesis, but HUMAN_ONLY (2 sanctioned tenants). Not resolvable passively.
[FINAL] shop admin-token (40): survives — REST route removed while GraphQL mutation present is a real anomaly, but official-doc authz gate keeps it low; only resolvable via one sanctioned POST.
[NEXT] PROBE: POST https://shop.fonial.de/graphql `{"query":"mutation{ generateCustomerTokenAsAdmin(input:{customer_email:\"authz-probe-<unix-ts>@example.invalid\"}){ customer_token } }"}` — fabricated email, zero real-data exposure; observe `errors` (by-design admin-authz) vs `customer not found`/token (bypass). Requires sanctioned mutating-test switch; blocked in current passive mode.
[LEARN] ACCEPTED GraphQL mutation surface @ shop.fonial.de: 68 mutations enumerated fully (prior); full REST map now complementary — 45 paths via `/rest/all/schema`; only non-stock = MageWorx Downloads; REST passive discovery now closed (mirrors GraphQL closure).
[LEARN] REJECTED MageWorx guest-downloads IDOR @ shop.fonial.de/rest: endpoint live but `[]` across all 110 product IDs — no data to access; `/V1/mw-downloads-attachments/{id}` 401-gated; class parked.
[LEARN] ACCEPTED asymmetric admin-auth @ shop.fonial.de: REST `/V1/integration/admin/token` → 404 (route removed) but GraphQL `generateCustomerTokenAsAdmin` introspectable → non-uniform admin-auth surface; supports (does not prove) admin-token hypothesis.
[LEARN] REJECTED REST-schema exposure @ shop.fonial.de/rest/all/schema: stock Magento OpenAPI, no secrets, surface already known via GraphQL introspection — informational only, not reportable.
[RISK] fonial: 56 — both surviving leads gated: cross-bind (75, HUMAN_ONLY, 2 tenants) and admin-token (40, one sanctioned POST). REST+GraphQL surface now fully mapped both sides → REST admin-token route-removal is the only meaningful new asymmetry, no impact. MageWorx dead (empty data). Flat-close probability ~55%: high-value resolution requires sanctioned interactivity this mode cannot take. No writes, no customer data, ≤1 rps.
## 2026-09-08 00:32:46 UTC [target] (model bigpickle)
## 2026-09-08 05:24:04 UTC [target] (model bigpickle)
impact: Database order states / revenue tampering. HIGH if exploitable.
testability: HUMAN_ONLY (insufficient evidence — parking)
[NEW] shop.fonial.de REST surface fully mapped: `/rest/all/schema` returns full OpenAPI unauthenticated → 45 paths; only non-stock route = MageWorx `mw-downloads-attachments`.
[NEW] `/V1/mw-downloads-attachments/guest/product/{id}` live unauth but returns `[]` on all 110 product IDs; `/V1/mw-downloads-attachments/{id}` → 401 (auth-gated sibling exists).
[NEW] REST `/V1/integration/admin/token` + `customer/token` → 404 route-removed (REST admin-auth closed) while GraphQL `GenerateCustomerTokenAsAdminInput` remains introspectable → asymmetric admin-auth surface.
[NEW] `/V1/search` live unauth (400 missing `searchCriteria`); `/V1/applepay/auth`, `/V1/payment-order/completeOrder` present stock payment routes.
[PRIO] kundenkonto.fonial.de/api/2.0, 6.1, (6,10,7,3,3,3)
[PRIO] shop.fonial.de/graphql, 7.25, (7,8,9,7,3,8)
[PRIO] shop.fonial.de/rest, 6.55, (6,8,7,6,3,8)
[PRIO] www.fonial.de, 4.95, (5,4,5,9,3,3)
[HYP] Dual-backend SID/PHPSESSID cross-binding → cross-tenant access on 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Session backend issues cleartext UUID SID in body, no PHPSESSID; data backend authorizes by body SID only and sets PHPSESSID decoratively. Independent session stores → SID-to-credential binding never verified. call/initiate is live WRITE with identical `session invalid` pattern.
evidence_needed: SID from tenant A + PHPSESSID from tenant B returns data, or unauthenticated SID + foreign PHPSESSID returns data.
verify_steps: POST /api/2.0/session each tenant → authenticate → capture PHPSESSID per tenant from data POST → cross-bind `{"sid":"SID_B"}` + `Cookie: PHPSESSID=A` on /devices/get, /evn/get, /call/initiate → non-`session invalid` = CRITICAL.
impact: Cross-tenant CDR, numbers, SIP creds, device lists, outbound initiation/toll fraud. CRITICAL.
testability: HUMAN_ONLY
[HYP] generateCustomerTokenAsAdmin callable without admin bearer (version-specific authz flaw)
class: AUTH
asset: shop.fonial.de/graphql (generateCustomerTokenAsAdmin)
confidence: 40
reasoning: Mutation present unauth-introspectable while REST `/V1/integration/admin/token` route is REMOVED (404) → asymmetric admin-auth surface implies non-stock config. Input = single `customer_email`. Official docs require admin Bearer + remote_shopping_assistance opt-in; residual = inverted/missing user-type check in this 2.4 CE build.
evidence_needed: POST without Authorization → error is authz-based (by-design) vs `customer not found`/token issuance (bypass).
verify_steps: SANCTIONED POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){ customer_token } }` — fabricated email = zero customer-data exposure; observe error semantics.
impact: Customer ATO of opted-in shop accounts (orders, addresses, pay vault) → payment/PII exposure. CRITICAL if bypass.
testability: PASSIVE (blocked in this mode — needs sanctioned POST)
[HYP] MageWorx Downloads authenticated attachment access → file-level IDOR
class: IDOR
asset: shop.fonial.de/rest /V1/mw-downloads-attachments
confidence: 35
reasoning: Non-stock extension (MageWorx Downloads 2.x) exposes guest product-attachment metadata endpoint (live, `[]` today) and an auth-gated `/V1/mw-downloads-attachments/{id}` (401). Fonial sells "Telefonansage" audio products — downloadable-attachment schema present. Known MageWorx flaws = attachment link tokens/short IDs.
evidence_needed: authenticated user A enumerates attachment IDs / link tokens of user B or an un-purchased product file; guest endpoint returning non-empty for a purchasable product.
verify_steps: needs auth context → real customer account; passive-only: re-GET all 110 product IDs after new products drop; check `/rest/all/schema` refresh for added attachment endpoints.
impact: Unauthorized file/PII download of gated product attachments. MED-HIGH.
testability: HUMAN_ONLY (no attachments currently exposed; needs purchasable downloadable product + auth)
[PARKED] MageWorx Downloads IDOR (35): below floor; guest endpoint provably empty across all products; the 401 sibling needs an authenticated session none of us have; no live data to pivot on.
[PARKED] Guest-cart coupon abuse (30): dropped prior cycle — by-design guest checkout, stock coupon validation, live placeOrder is a financial write.
[PARKED] Payment-callback desync (32): requires live order + gateway interaction; no passive vector.
[FINAL] kundenkonto cross-bind (75): survives — sole high-impact hypothesis, but HUMAN_ONLY (2 sanctioned tenants). Not resolvable passively.
[FINAL] shop admin-token (40): survives — REST route removed while GraphQL mutation present is a real anomaly, but official-doc authz gate keeps it low; only resolvable via one sanctioned POST.
[NEXT] PROBE: POST https://shop.fonial.de/graphql `{"query":"mutation{ generateCustomerTokenAsAdmin(input:{customer_email:\"authz-probe-<unix-ts>@example.invalid\"}){ customer_token } }"}` — fabricated email, zero real-data exposure; observe `errors` (by-design admin-authz) vs `customer not found`/token (bypass). Requires sanctioned mutating-test switch; blocked in current passive mode.
[LEARN] ACCEPTED GraphQL mutation surface @ shop.fonial.de: 68 mutations enumerated fully (prior); full REST map now complementary — 45 paths via `/rest/all/schema`; only non-stock = MageWorx Downloads; REST passive discovery now closed (mirrors GraphQL closure).
[LEARN] REJECTED MageWorx guest-downloads IDOR @ shop.fonial.de/rest: endpoint live but `[]` across all 110 product IDs — no data to access; `/V1/mw-downloads-attachments/{id}` 401-gated; class parked.
[LEARN] ACCEPTED asymmetric admin-auth @ shop.fonial.de: REST `/V1/integration/admin/token` → 404 (route removed) but GraphQL `generateCustomerTokenAsAdmin` introspectable → non-uniform admin-auth surface; supports (does not prove) admin-token hypothesis.
[LEARN] REJECTED REST-schema exposure @ shop.fonial.de/rest/all/schema: stock Magento OpenAPI, no secrets, surface already known via GraphQL introspection — informational only, not reportable.
[RISK] fonial: 56 — both surviving leads gated: cross-bind (75, HUMAN_ONLY, 2 tenants) and admin-token (40, one sanctioned POST). REST+GraphQL surface now fully mapped both sides → REST admin-token route-removal is the only meaningful new asymmetry, no impact. MageWorx dead (empty data). Flat-close probability ~55%: high-value resolution requires sanctioned interactivity this mode cannot take. No writes, no customer data, ≤1 rps.
[HYP] customers.fonial.de = unenumerated second customer portal on dedicated non-CDN IP
class: AUTH
asset: customers.fonial.de (92.197.176.46)
confidence: 40
reasoning: New host, no Cloudflare/62.146.7.2x; name implies customer-facing app; silent on :80/:443 from this egress (host-filter/geo/down) — surface completely unprobed.
evidence_needed: any live service (TLS SNI banner, HTTP, alt port) → app identity/login/API to compare with kundenkonto patterns.
verify_steps: passive: TLS-SNI + port banner via openssl s_client (exceeds pure-HTTP bounds → sanctioned); if live, GET / and kundenkonto-style paths.
impact: Potential parallel customer portal → AUTH/IDOR surface. MED-HIGH if live.
testability: HUMAN_ONLY (no HTTP responder observed)
[HYP] sbc/ldap/devops public-IP group = internet-reachable telephony/identity/devops infra
class: MISCONFIG
asset: sbc.fonial.de (62.146.28.124), ldap.fonial.de (62.146.7.24), 62.146.7.22, 62.146.28.126
confidence: 45
reasoning: CT+certs put an SBC (SIP), an LDAP dir, RabbitMQ/Sentry app servers on public IPs; web layer on 62.146.7.22 returns 403/404 proxy-deny (rabbitmq/penpot/waiting-queue) while sentry serves a live org login — non-HTTP ports (5060/5061 SIP, 389/636 LDAP, 15672 RMQ) never probed, host-level reachability unknown.
evidence_needed: single service banner per host (SIP OPTIONS / LDAP anonymous bind / RMQ mgmt port) — banner only, zero auth attempts.
verify_steps: sanctioned single-packet service classification on sbc:5060/5061, ldap:389/636, 62.146.7.22:15672, 62.146.28.126:9000.
impact: Toll fraud / internal identity enumeration / error-tracking PII if exposed. HIGH if reachable+misconf by banner.
testability: HUMAN_ONLY (non-HTTP; exceeds passive bounds)
[HYP] red/mm/sentry old-build fingerprint → patchable auth-bypass
class: AUTH
asset: red.fonial.de (Redmine), mm.fonial.de (Mattermost), sentry.fonial.dev
confidence: 45
reasoning: red=Redmine +Agile/Questions on Passenger 5.1.1/nginx 1.10.2 (pre-2020 era), mm=old Mattermost (v4-only API), sentry=self-hosted (`version:"0"` reply) with org slug `fonial-de`. All three are internal-collaboration tools exposed on public DNS+certs; historical auth-by-CVE sets exist for each in these build eras. Version not extractable passively (login-only surfaces).
evidence_needed: exact build → version-exact CVE matching deployed code path (program-specific exploit, not just version listing).
verify_steps: passive: pull webroot JS bundles on red/mm for version strings; monitor deploy cabtimestamps; no live exploit without exact-build match.
impact: Internal chat/issues/error-data breach → MED-HIGH only if version-exact bypass confirmed; login-panel+known-vuln classes otherwise.
testability: PASSIVE (fingerprint) / HUMAN_ONLY (exploit)
[NEXT] HUMAN: sanctioned single-packet service classification of the 4 new public infra targets — customers.fonial.de (92.197.176.46, TLS-SNI+banner), sbc.fonial.de (62.146.28.124, SIP OPTIONS 5060/5061), ldap.fonial.de (62.146.7.24, LDAP bind banner 389/636), 62.146.7.22:15672 + 62.146.28.126 (RMQ/Sentry ports); banner-only, zero auth attempts; passive-HTTP pool is exhausted (every reachable new host is auth-gated).
[RISK] fonial: 58 — CT-delta expands inventory to ~90 hostnames across 2 domains and reveals a publicly reachable self-hosted Sentry (org slug leaked) plus 4 public-IP infra hosts, but every reachable new web surface is auth-gated/proxy-denied (login-panel + known-vuln-without-exploit classes only). No new confirmed finding; the two decisive leads stay sanctioned-gated (cross-bind 75 = 2 tenants; admin-token 40 = 1 POST). Flat-close probability now ~50%: genuine new attack surface identified, but resolution of all three surviving classes (cross-bind, customers liveness, infra banners) requires sanctioned interactivity this mode cannot take.
## 2026-09-08 09:48:01 UTC [target] (model bigpickle)
[HYP] customers.fonial.de = unenumerated second customer portal on dedicated non-CDN IP
reasoning: New host, no Cloudflare/62.146.7.2x; name implies customer-facing app; silent on :80/:443 from this egress (host-filter/geo/down) — surface completely unprobed.
[HYP] sbc/ldap/devops public-IP group = internet-reachable telephony/identity/devops infra
[HYP] red/mm/sentry old-build fingerprint → patchable auth-bypass
[NEXT] HUMAN: sanctioned single-packet service classification of the 4 new public infra targets — customers.fonial.de (92.197.176.46, TLS-SNI+banner), sbc.fonial.de (62.146.28.124, SIP OPTIONS 5060/5061), ldap.fonial.de (62.146.7.24, LDAP bind banner 389/636), 62.146.7.22:15672 + 62.146.28.126 (RMQ/Sentry ports); banner-only, zero auth attempts; passive-HTTP pool is exhausted (every reachable new host is auth-gated).
[RISK] fonial: 58 — CT-delta expands inventory to ~90 hostnames across 2 domains and reveals a publicly reachable self-hosted Sentry (org slug leaked) plus 4 public-IP infra hosts, but every reachable new web surface is auth-gated/proxy-denied (login-panel + known-vuln-without-exploit classes only). No new confirmed finding; the two decisive leads stay sanctioned-gated (cross-bind 75 = 2 tenants; admin-token 40 = 1 POST). Flat-close probability now ~50%: genuine new attack surface identified, but resolution of all three surviving classes (cross-bind, customers liveness, infra banners) requires sanctioned interactivity this mode cannot take.
[HYP] generateCustomerTokenAsAdmin callable without admin bearer
class: AUTH
asset: shop.fonial.de/graphql (generateCustomerTokenAsAdmin)
confidence: 40
reasoning: Mutation present unauth-introspectable while REST `/V1/integration/admin/token` route REMOVED (404) → asymmetric admin-auth surface implies non-stock config. Input = single `customer_email`. Official docs require admin Bearer + remote_shopping_assistance opt-in; residual = inverted/missing user-type check in this 2.4 CE build. (Live-verified intact this cycle.)
evidence_needed: POST without Authorization → authz-error (by-design) vs `customer not found`/token issuance (bypass).
verify_steps: SANCTIONED POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<unix-ts>@example.invalid"}){ customer_token } }` — fabricated email, zero data exposure.
impact: Customer ATO of opted-in shop accounts (orders, addresses, pay vault) → payment/PII exposure. CRITICAL if bypass.
testability: HUMAN_ONLY (needs sanctioned POST)
[HYP] sbc/ldap/devops public-IP group = reachable telephony/identity infra
class: MISCONFIG
asset: sbc.fonial.de (62.146.28.124), ldap.fonial.de (62.146.7.24), 62.146.7.22:15672, 62.146.28.126
confidence: 45
reasoning: CT+certs place SBC/SIP, LDAP dir, RabbitMQ/Sentry on public IPs; web layer proxy-deny (403/404) but non-HTTP ports (5060/5061 SIP, 389/636 LDAP, 15672 RMQ) never probed; host-level reachability unknown.
evidence_needed: single banner per host — SIP OPTIONS / LDAP anonymous bind / RMQ mgmt banner, zero auth.
verify_steps: sanctioned single-packet classification; banner-only, no binds/auth attempts.
impact: Toll fraud / internal identity enumeration / error-tracking PII if reachable+misconf. HIGH if banner confirms.
testability: HUMAN_ONLY (non-HTTP)
[HYP] customers.fonial.de = unenumerated second customer portal
class: AUTH
asset: customers.fonial.de (92.197.176.46)
confidence: 40
reasoning: New host, dedicated non-CDN IP, name implies customer-facing app; silent on :80/:443 from this egress (host-filter/geo/down) — surface completely unprobed.
evidence_needed: any live service (banner, HTTP, alt port) → app identity/login/API comparable to kundenkonto.
verify_steps: HUMAN: TLS-SNI + banner via openssl s_client; if live, GET / and kundenkonto-style paths.
impact: Potential parallel customer portal → AUTH/IDOR surface. MED-HIGH if live.
testability: HUMAN_ONLY (no HTTP responder observed)
[NEXT] HUMAN: sanctioned single-packet service classification of the 4 public-IP infra targets — customers.fonial.de (92.197.176.46, TLS-SNI + banner), sbc.fonial.de (62.146.28.124, SIP OPTIONS 5060/5061), ldap.fonial.de (62.146.7.24, LDAP banner 389/636), 62.146.7.22:15672 + 62.146.28.126 (RMQ/Sentry ports); banner-only, zero auth attempts; the passive-HTTP pool is fully exhausted — every reachable new host is auth-gated/proxy-denied and live re-probes confirm no drift.
[RISK] fonial: 58 — no drift in 4.5h across all assets; live re-probes confirm every decisive lead is sanctioned-gated (cross-bind 75 = 2 tenants; admin-token 40 = 1 fabricated-email POST; customers/infra = single-packet banner). Zero new passive surface; all 5 API endpoints, GraphQL schema, and CT-derived hostnames reconfirm prior closures. Flat-close probability holds ~50%: meaningful reportable impact requires any one authorized interactivity switch this mode cannot take. No writes, no customer data, ≤1 rps.
## 2026-09-08 14:08:14 UTC [target] (model bigpickle)
[HYP] Data-layer SID authz not credential-bound → cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only, PHPSESSID decorative/parallel; /session issues cleartext UUID sid; every unauth probe returns identical `session invalid`/`session unauthenticated` JSON. The sole open question is whether authenticated sid→account binding is enforced. Frontend uses separate Symfony /login_check; API credential path independent → double session store.
evidence_needed: two valid accounts; confirm SID_B + PHPSESSID_A accepted (rejected = credential-bound) on /evn/get; or unauthenticated SID + valid PHPSESSID returns data.
verify_steps: (HUMAN, sanctioned, 2 test tenants) 1) /session→SID_A; 2) /session/authenticate A; 3) /devices/get SID_A→PHPSESSID_A; 4) /session→SID_B; 5) /evn/get {sid:SID_B} + Cookie PHPSESSID_A; observe data vs error.
impact: cross-tenant CDR, phone/SIP creds, device lists, unauthorized outbound call initiation (call/initiate). CRITICAL.
testability: HUMAN_ONLY
[HYP] generateCustomerTokenAsAdmin callable without admin bearer
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: Mutation unauth-introspectable; REST `/V1/integration/admin/token` route REMOVED (404) → asymmetric admin-auth implies non-stock config; input = single customer_email; stock docs require admin Bearer + customer remote_shopping_assistance opt-in.
evidence_needed: POST without Authorization → authz-error (by-design) vs token issuance (bypass).
verify_steps: SANCTIONED POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<unix-ts>@example.invalid"}){ customer_token } }` — fabricated email, zero data exposure.
impact: customer ATO of opted-in accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[HYP] customers/sbc/ldap/devops public-IP group = reachable telephony/identity/devops infra
class: MISCONFIG
asset: customers.fonial.de (92.197.176.46), sbc.fonial.de (62.146.28.124:5060/5061), ldap.fonial.de (62.146.7.24:389/636), 62.146.7.22:15672, 62.146.28.126
confidence: 45
reasoning: CT+certs place SBC/SIP, LDAP dir, RMQ/Sentry on public IPs; web layer proxy-deny (403/404) but non-HTTP ports never probed; host-level reachability unknown.
evidence_needed: single service banner per host (SIP OPTIONS / LDAP bind banner / RMQ mgmt port) — banner only, zero auth.
verify_steps: sanctioned single-packet classification, no binds/auth.
impact: toll fraud / internal identity enumeration / error-tracking PII. HIGH if reachable+misconf by banner.
testability: HUMAN_ONLY
[NEXT] HUMAN: sanctioned single-packet service classification of 4 public infra targets — customers.fonial.de (92.197.176.46, TLS-SNI+banner via openssl s_client), sbc.fonial.de (62.146.28.124, SIP OPTIONS 5060/5061), ldap.fonial.de (62.146.7.24, LDAP banner 389/636), 62.146.7.22:15672 + 62.146.28.126 (RMQ/Sentry ports); banner-only, zero auth; passive-HTTP pool remains fully exhausted (live re-probes confirm no drift).
[RISK] fonial: 58 — Live re-probes (14:05 UTC) confirm zero drift on all assets; every decisive lead (cross-bind 75=2 tenants, admin-token 40=1 fabricated-email POST, customers/sbc/ldap=single-packet banner) is sanctioned-gated, none passively resolvable. Genuine high-value surface (double session store, asymmetric admin-auth, public telephony/identity infra) confirmed via CT+certs, but resolution requires authorized interactivity this mode cannot take. Flat-close ~50%. Zero writes, no real customer data, ≤1 rps.
## 2026-09-08 18:00:24 UTC [target] (model bigpickle)
[HYP] Data-layer SID authz not credential-bound → cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid; /session/authenticate is an independent credential path; every unauth probe returns identical `session invalid`/`session unauthenticated` JSON. Sole open question: is authenticated sid→account binding enforced? Re-probe 17:59 UTC confirms store dynamics unchanged.
evidence_needed: SID_B body + PHPSESSID_A cookie accepted on /evn/get (rejected = credential-bound); or unauthenticated SID + valid PHPSESSID returns data.
verify_steps: (HUMAN, sanctioned, 2 test tenants) 1) GET /api/2.0/session→SID_A; 2) POST /session/authenticate A; 3) POST /devices/get {sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session→SID_B; 5) POST /evn/get {sid:SID_B} + Cookie:PHPSESSID_A; observe data vs error.
impact: cross-tenant CDR, phone/SIP creds, device lists, unauthorized outbound calls via call/initiate. CRITICAL.
testability: HUMAN_ONLY
[HYP] generateCustomerTokenAsAdmin callable without admin bearer
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: Mutation unauth-introspectable while REST `/V1/integration/admin/token` is 404 (route removed) → asymmetric admin-auth surface implies non-stock config; input = single `customer_email`; stock docs require admin Bearer + remote_shopping_assistance opt-in; residual = inverted/missing user-type check in this 2.4 CE build.
evidence_needed: POST without Authorization → authz-error (by-design) vs token issuance (bypass).
verify_steps: SANCTIONED POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<unix-ts>@example.invalid"}){ customer_token } }` — fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[HYP] sbc/ldap/devops public-IP group = reachable telephony/identity infra
class: MISCONFIG
asset: sbc.fonial.de (62.146.28.124:5060/5061), ldap.fonial.de (62.146.7.24:389/636), 62.146.7.22:15672, 62.146.28.126
confidence: 45
reasoning: CT+certs place SBC/SIP, LDAP dir, RabbitMQ/Sentry on public IPs; HTTP layer proxy-deny (403/404); non-HTTP ports never probed; host-level reachability unknown.
evidence_needed: single banner per host (SIP OPTIONS / LDAP bind banner / RMQ mgmt port) — banner only, zero auth.
verify_steps: sanctioned single-packet classification per host; no binds/auth attempts.
impact: toll fraud / internal identity enumeration / error-tracking PII. HIGH if reachable+misconf by banner.
testability: HUMAN_ONLY
## 2026-09-08 20:48:40 UTC [target] (model bigpickle)
[HYP] Data-layer SID authz not credential-bound → cross-tenant access on all 3 data endpoints
class: AUTH
asset: kundenkonto.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid; /session/authenticate is an independent credential path; every unauth probe returns identical `session invalid`/`session unauthenticated` JSON. 20:47 probe reconfirms store unchanged. Sole open question: is authenticated sid→account binding enforced?
evidence_needed: SID_B body + PHPSESSID_A cookie accepted on /evn/get (rejected = credential-bound); or unauthenticated SID + valid PHPSESSID returns data.
verify_steps: (HUMAN, program-approved tenants T_A/T_B — account_creation restricted, do NOT self-create live accounts) 1) GET /api/2.0/session→SID_A; 2) POST /session/authenticate{...T_A}→authenticated; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh jar)→SID_B; 5) POST /evn/get{sid:SID_B}+Cookie:PHPSESSID_A → data vs error. 1 request, ≤1rps.
impact: cross-tenant CDR, phone/SIP creds, device lists, unauthorized outbound calls (write endpoint). CRITICAL.
testability: HUMAN_ONLY
[HYP] generateCustomerTokenAsAdmin callable without admin bearer
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: Mutation unauth-introspectable; REST `/V1/integration/admin/token` → 404 (route removed) → asymmetric admin-auth; input = single `customer_email`; stock docs require admin Bearer + `remote_shopping_assistance` opt-in. Residual = inverted/missing user-type check in this 2.4 CE build. No new evidence since 55→40 downgrade / prior REJECTED.
evidence_needed: POST without Authorization → authz-error (by-design) vs token issuance (bypass).
verify_steps: SANCTIONED single POST /graphql `mutation{ generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){ customer_token } }` — fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[HYP] sbc/ldap/RMQ/Sentry public-IP group = reachable telephony/identity/devops infra
class: MISCONFIG
asset: sbc.fonial.de (62.146.28.124:5060/5061), ldap.fonial.de (62.146.7.24:389/636), 62.146.7.22:15672, 62.146.28.126, customers.fonial.de (92.197.176.46)
confidence: 45
reasoning: CT+certs place SBC/SIP, LDAP, RabbitMQ/Sentry on public IPs; HTTP layer proxy-deny; non-HTTP ports never probed; host-level reachability unknown.
evidence_needed: single banner per host — banner-only, zero auth.
verify_steps: sanctioned single-packet classification (SIP OPTIONS / LDAP banner / RMQ mgmt port reachability).
impact: toll fraud / identity enumeration / error-tracking PII exposure. HIGH if reachable+misconf.
testability: HUMAN_ONLY
[NEXT] HUMAN: obtain program-approved test tenants T_A/T_B, then run the 5-request cross-bind sequence (s3): `POST /evn/get {"sid":SID_B}` + `Cookie: PHPSESSID_A` on kundenkonto.fonial.de/api/2.0 — data ⇒ BOLA confirmed (CRITICAL); error ⇒ sid credential-bound, hypothesis closed, flat-close.
[RISK] fonial: 58 — 20:47 probe confirms zero drift across all assets; passive pool exhausted; sole [FINAL] survivor (75) needs program-approved tenants, admin-token and infra suppressed by REJECTED/OOS rulings → reportable-outcome probability remains ~50% flat-close. Zero writes, no customer data touched, ≤1rps.
## 2026-09-08 23:13:04 UTC [target] (model bigpickle)
[HYP] Cross-tenant BOLA via dual-backend SID/PHPSESSID on unfronted prov.fonial.de
class: AUTH
asset: prov.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: prov is a byte-behavior duplicate of kundenkonto (same X-Fonial-Version v2026.09.03-1, same SID issuance, same session-invalid JSON, same PHPSESSID decoration, same ACAO *) but served directly on nginx/1.10.3. Same dual-backend bind: data endpoints authorize by body SID only. Cross-bind test now possible on a lower-impact, non-Cloudflare deployment.
evidence_needed: 2 program-approved tenants on prov; SID_B + PHPSESSID_A accepted on /evn/get (BOLA confirmed) vs `session invalid` (credential-bound).
verify_steps: (HUMAN, sanctioned) 1) GET /api/2.0/session→SID_A; 2) POST /session/authenticate{T_A}; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh)→SID_B; 5) POST /evn/get{sid:SID_B}+Cookie:PHPSESSID_A. 1 req, ≤1rps.
impact: cross-tenant CDR, device lists, SIP creds, outbound call control on the portal API. CRITICAL.
testability: HUMAN_ONLY
[HYP] Open self-signup on internal Mattermost 3.7 enables outsider account creation in company chat
class: BUSLOGIC
asset: mm.fonial.de
confidence: 55
reasoning: `/signup/email` returns 200 (no redirect to disabled-signup), `/api/v4/users/ping` 401 (SFTP session expiry, normal for logged-out), X-Version-Id 3.7.3..., v4 config route absent (legacy). Default vendor behavior: email+password signup w/o admin approval when EnableSignUpWithEmail=true; guest/SSO unknown. Signup = mutating → program-approved.
evidence_needed: POST /users/create with fabricated email observed to issue an account token vs adminApproval/disabled error.
verify_steps: (HUMAN, sanctioned) single GET /signup/email → form fields; single POST /api/v4/users/create {email:<sanitised>,username:"ot-<ts>",password:…}; revert/delete if created.
impact: outsider foothold in company chat (internal directory, channels, webhooks) + legacy-version CVEs prior. HIGH if open.
testability: HUMAN_ONLY
[HYP] Shop admin-token asymmetric surface
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: unchanged from prior - REST /V1/integration/admin/token 404 but GraphQL generateCustomerTokenAsAdmin introspectable. Carried for completeness.
impact: ATO of opted-in shop accounts. CRITICAL if bypass.
testability: HUMAN_ONLY
## 2026-09-09 01:18:20 UTC [target] (model bigpickle)
[NEW] prov.fonial.de/api/2.0 — byte-behavior duplicate of kundenkonto (X-Fonial-Version v2026.09.03-1, cleartext UUID SID, PHPSESSID decoration, ACAO *) served directly on nginx/1.10.3 without Cloudflare → cross-bind testable on lower-impact unfronted deployment.
[NEW] mm.fonial.de — internal Mattermost 3.7.x; `/signup/email` 200, `/api/v4/users/ping` 401, v4 config absent → default vendor self-signup untested.
[CHANGED] kundenkonto + shop surfaces unchanged (probe 2026-09-08 23:13:15: call/initiate 200, graphql 500); no drift in X-Fonial-Version v2026.09.03-1.
[PRIO] prov.fonial.de/api/2.0, 8.7, a=8 b=10 t=7 g=9 c=8 f=10 (unfronted → highest gate_ease of the three portal API deployments)
[PRIO] kundenkonto.fonial.de/api/2.0, 7.8, a=8 b=10 t=7 g=6 c=8 f=5
[PRIO] mm.fonial.de, 5.7, a=5 b=6 t=6 g=5 c=4 f=9
[PRIO] shop.fonial.de/graphql, 6.7, a=7 b=8 t=8 g=5 c=6 f=4
[HYP] Cross-tenant BOLA via dual-backend SID/PHPSESSID on unfronted prov.fonial.de
class: AUTH
asset: prov.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: prov is a byte-behavior duplicate of kundenkonto (X-Fonial-Version v2026.09.03-1, SID issuance, session-invalid JSON, PHPSESSID decoration, ACAO *) on direct nginx/1.10.3, no Cloudflare. Data endpoints authorize by body SID only; session endpoint issues cleartext UUID sid; every unauth probe returns identical session-invalid JSON. Cross-binding sid↔account is the sole open question and now testable on a lower-impact deployment.
evidence_needed: program-approved tenants T_A/T_B on prov; `POST /evn/get {"sid":SID_B}` + `Cookie: PHPSESSID_A` → data (BOLA) vs `session invalid` (credential-bound).
verify_steps: (HUMAN, sanctioned) 1) GET /api/2.0/session→SID_A; 2) POST /session/authenticate{T_A}; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh jar)→SID_B; 5) POST /evn/get{sid:SID_B}+Cookie:PHPSESSID_A. 1 request each, ≤1rps, zero writes.
impact: cross-tenant CDR, device/SIP creds, unauthorized outbound call control on portal API. CRITICAL.
testability: HUMAN_ONLY
[HYP] Open self-signup on internal Mattermost 3.7 enables outsider account entry in company chat
class: BUSLOGIC
asset: mm.fonial.de
confidence: 55
reasoning: `/signup/email` returns 200 with no disabled-signup redirect; `/api/v4/users/ping` 401 (normal logged-out); X-Version-Id 3.7.3...; v4 config route absent (legacy build). Vendor default: email+password self-signup active when EnableSignUpWithEmail=true; real signup path never POSTed. Signup is mutating → program-approved only.
evidence_needed: POST /api/v4/users/create with fabricated email issues account token vs adminApproval/disabled error.
verify_steps: (HUMAN, sanctioned) single POST /api/v4/users/create {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>}; on success immediately delete/disable account; single GET /signup/email to confirm form fields first. ≤1rps.
impact: outsider foothold in company chat (internal directory, channels, webhooks) plus legacy-3.7 CVE surface. HIGH if open.
testability: HUMAN_ONLY
[HYP] Shop admin-token asymmetric surface
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: REST `/V1/integration/admin/token` → 404 (route removed) while GraphQL `generateCustomerTokenAsAdmin` is unauth-introspectable (input = single `customer_email`); Adobe docs require admin Bearer + `remote_shopping_assistance` opt-in. Residual = inverted/missing user-type check in this 2.4 CE build. No new evidence since downgrade.
evidence_needed: POST /graphql `mutation{generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){customer_token}}` without Authorization → authz-error vs token issuance.
verify_steps: (HUMAN, sanctioned) single POST, fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[PARKED] shop admin-token (40): no new evidence since 55→40; relegate to chain-only, not primary.
[PARKED] prov v1/v2 version-diff probe: alternate API versions already 404 on kundenkonto; no evidence prov differs — needs a read-only drift check before crediting any divergence.
[PARKED] sbc/LDAP/RMQ/Sentry public-IP banner group (45): single-packet classification is HUMAN-only; no HTTP reachability evidence; unchanged.
[FINAL] prov.fonial.de cross-tenant BOLA (75) — top survivor, gated on TWO approved tenants.
[FINAL] mm.fonial.de open self-signup (55) — gated on ONE approved mutating POST, no tenant prerequisite.
[NEXT] HUMAN: request program approval for ONE sanctioned mutating POST, then run `POST https://mm.fonial.de/api/v4/users/create` {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>} after `GET /signup/email` to confirm form fields; capture token-issuance vs adminApproval/disabled error, delete account immediately if created. This is the only active hypothesis whose blocker is a single approval rather than two real test tenants; the 75 BOLA cross-bind remains queued behind T_A/T_B provisioning.
[LEARN] ACCEPTED prov.fonial.de duplicate @ prov.fonial.de/api/2.0: unfronted byte-behavior clone of kundenkonto API (same version, session semantics, PHPSESSID decoration, ACAO *) → lowest-gate deployment to test dual-backend cross-bind.
[LEARN] REJECTED shop admin-token standalone @ shop.fonial.de/graphql: confidence 40 at critique floor, no new evidence; Drop as primary, retain only as chain component.
[RISK] fonial: 58 — passive pool still exhausted; top survivor (75 BOLA) gated on two program-approved tenants, Mattermost (55) on one approved POST; no writes, no customer data touched since 23:13 probe reconfirms zero drift. Reportable-outcome probability remains ~50% flat-close if outside approvals stall.
## 2026-09-09 06:03:15 UTC [target] (model bigpickle)
[PRIO] prov.fonial.de/api/2.0,8.7
[PRIO] mm.fonial.de,5.7
[PRIO] kundenkonto.fonial.de/api/2.0,7.8
[PRIO] shop.fonial.de/graphql,6.7
[HYP] Cross-tenant BOLA via dual-backend SID/PHPSESSID on unfronted prov.fonial.de
class: AUTH
asset: prov.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: prov is byte-behavior duplicate of kundenkonto (same version, session semantics, PHPSESSID decoration, ACAO *) on direct nginx/1.10.3 without Cloudflare. Data endpoints authorize by body SID only; session endpoint issues cleartext UUID sid; every unauth probe returns identical session-invalid JSON. Cross-binding sid↔account is the sole open question and testable on lower-impact unfronted deployment.
evidence_needed: program-approved tenants T_A/T_B on prov; POST /evn/get {"sid":SID_B} + Cookie: PHPSESSID_A → data (BOLA) vs session invalid (credential-bound).
verify_steps: (HUMAN, sanctioned) 1) GET /api/2.0/session→SID_A; 2) POST /session/authenticate{T_A}; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh jar)→SID_B; 5) POST /evn/get{sid:SID_B}+Cookie:PHPSESSID_A. 1 request each, ≤1rps, zero writes.
impact: cross-tenant CDR, device/SIP creds, unauthorized outbound call control on portal API. CRITICAL.
testability: HUMAN_ONLY
[HYP] Open self-signup on internal Mattermost 3.7 enables outsider account entry in company chat
class: BUSLOGIC
asset: mm.fonial.de
confidence: 55
reasoning: /signup/email returns 200 with no disabled-signup redirect; /api/v4/users/ping 401 (normal logged-out); X-Version-Id 3.7.3...; v4 config route absent (legacy build). Vendor default: email+password self-signup active when EnableSignUpWithEmail=true; real signup path never POSTed. Signup is mutating → program-approved only.
evidence_needed: POST /api/v4/users/create with fabricated email issues account token vs adminApproval/disabled error.
verify_steps: (HUMAN, sanctioned) single POST /api/v4/users/create {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>}; on success immediately delete/disable account; single GET /signup/email to confirm form fields first. ≤1rps.
impact: outsider foothold in company chat (internal directory, channels, webhooks) plus legacy-3.7 CVE surface. HIGH if open.
testability: HUMAN_ONLY
[HYP] Shop admin-token asymmetric surface
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: REST /V1/integration/admin/token → 404 (route removed) while GraphQL generateCustomerTokenAsAdmin is unauth-introspectable (input = single customer_email); Adobe docs require admin Bearer + customer opt-in. Residual = inverted/missing user-type check in this 2.4 CE build. No new evidence since downgrade.
evidence_needed: POST /graphql mutation{generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){customer_token}} without Authorization → authz-error vs token issuance.
verify_steps: (HUMAN, sanctioned) single POST, fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[PARKED] shop admin-token standalone (40): below report threshold as independent finding; retain only as chain component if BOLA/sidleak found.
[PARKED] prov v1/v2 version-diff: alternate API versions already 404 on kundenkonto; no evidence prov differs — speculative.
[PARKED] sbc/LDAP/RMQ/Sentry public-IP banner group (45): single-packet classification HUMAN-only; no HTTP reachability evidence.
[FINAL] prov.fonial.de cross-tenant BOLA (75) — top survivor, gated on TWO approved tenants.
[FINAL] mm.fonial.de open self-signup (55) — gated on ONE approved mutating POST, no tenant prerequisite.
[NEXT] HUMAN: request program approval for ONE sanctioned mutating POST, then run `POST https://mm.fonial.de/api/v4/users/create` {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>} after confirming form fields via GET /signup/email. Capture token-issuance vs adminApproval/disabled error; delete account immediately if created. This is the only hypothesis whose blocker is a single approval rather than two real test tenants; the 75 BOLA cross-bind remains queued behind T_A/T_B provisioning.
[LEARN] ACCEPTED prov.fonial.de duplicate @ prov.fonial.de/api/2.0: unfronted byte-behavior clone of kundenkonto API (same version, session semantics, PHPSESSID decoration, ACAO *) → lowest-gate deployment to test dual-backend cross-bind.
[LEARN] REJECTED shop admin-token standalone @ shop.fonial.de/graphql: confidence 40 at critique floor, no new evidence; drop as primary, retain only as chain component.
[RISK] fonial: 58 — passive pool exhausted; top survivor (75 BOLA) gated on two program-approved tenants, Mattermost (55) on one approved POST; no writes, no customer data touched since 23:13 probe reconfirms zero drift. Reportable-outcome probability ~50% flat-close if outside approvals stall.
## 2026-09-09 11:30:46 UTC [target] (model bigpickle)
[PRIO] prov.fonial.de/api/2.0, 8.7
[PRIO] mm.fonial.de, 5.7
[PRIO] kundenkonto.fonial.de/api/2.0, 7.8
[PRIO] shop.fonial.de/graphql, 6.7
[HYP] Cross-tenant BOLA via dual-backend SID/PHPSESSID on unfronted prov.fonial.de
class: AUTH
asset: prov.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: prov is byte-behavior duplicate of kundenkonto (same version, session semantics, PHPSESSID decoration, ACAO *) on direct nginx/1.10.3 without Cloudflare. Data endpoints authorize by body SID only; session endpoint issues cleartext UUID sid; every unauth probe returns identical session-invalid JSON. Cross-binding sid↔account is the sole open question and testable on lower-impact unfronted deployment.
evidence_needed: program-approved tenants T_A/T_B on prov; POST /evn/get {"sid":SID_B} + Cookie: PHPSESSID_A → data (BOLA) vs session invalid (credential-bound).
verify_steps: (HUMAN, sanctioned) 1) GET /api/2.0/session→SID_A; 2) POST /session/authenticate{T_A}; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh jar)→SID_B; 5) POST /evn/get{sid:SID_B}+Cookie:PHPSESSID_A. 1 request each, ≤1rps, zero writes.
impact: cross-tenant CDR, device/SIP creds, unauthorized outbound call control on portal API. CRITICAL.
testability: HUMAN_ONLY
[HYP] Open self-signup on internal Mattermost 3.7 enables outsider account entry in company chat
class: BUSLOGIC
asset: mm.fonial.de
confidence: 55
reasoning: /signup/email returns 200 with no disabled-signup redirect; /api/v4/users/ping 401 (normal logged-out); X-Version-Id 3.7.3...; v4 config route absent (legacy build). Vendor default: email+password self-signup active when EnableSignUpWithEmail=true; real signup path never POSTed. Signup is mutating → program-approved only.
evidence_needed: POST /api/v4/users/create with fabricated email issues account token vs adminApproval/disabled error.
verify_steps: (HUMAN, sanctioned) single POST /api/v4/users/create {email:ot-<ts>@example.invalid, username:ot-<ts, password:<random>}; on success immediately delete/disable account; single GET /signup/email to confirm form fields first. ≤1rps.
impact: outsider foothold in company chat (internal directory, channels, webhooks) plus legacy-3.7 CVE surface. HIGH if open.
testability: HUMAN_ONLY
[HYP] Shop admin-token asymmetric surface
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: REST /V1/integration/admin/token → 404 (route removed) while GraphQL generateCustomerTokenAsAdmin is unauth-introspectable (input = single customer_email); Adobe docs require admin Bearer + customer opt-in. Residual = inverted/missing user-type check in this 2.4 CE build. No new evidence since downgrade.
evidence_needed: POST /graphql mutation{generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){customer_token}} without Authorization → authz-error vs token issuance.
verify_steps: (HUMAN, sanctioned) single POST, fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[PARKED] shop admin-token standalone (40): no new evidence since 55→40; relegate to chain-only, not primary.
[PARKED] prov v1/v2 version-diff probe: alternate API versions already 404 on kundenkonto; no evidence prov differs — needs a read-only drift check before crediting any divergence.
[PARKED] sbc/LDAP/RMQ/Sentry public-IP banner group (45): single-packet classification is HUMAN-only; no HTTP reachability evidence; unchanged.
[FINAL] prov.fonial.de cross-tenant BOLA (75) — top survivor, gated on TWO approved tenants.
[FINAL] mm.fonial.de open self-signup (55) — gated on ONE approved mutating POST, no tenant prerequisite.
[NEXT] HUMAN: request program approval for ONE sanctioned mutating POST, then run `POST https://mm.fonial.de/api/v4/users/create` {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>} after `GET /signup/email` to confirm form fields; capture token-issuance vs adminApproval/disabled error, delete account immediately if created. This is the only active hypothesis whose blocker is a single approval rather than two real test tenants; the 75 BOLA cross-bind remains queued behind T_A/T_B provisioning.
[LEARN] ACCEPTED prov.fonial.de duplicate @ prov.fonial.de/api/2.0: unfronted byte-behavior clone of kundenkonto API (same version, session semantics, PHPSESSID decoration, ACAO *) → lowest-gate deployment to test dual-backend cross-bind.
[LEARN] REJECTED shop admin-token standalone @ shop.fonial.de/graphql: confidence 40 at critique floor, no new evidence; Drop as primary, retain only as chain component.
[RISK] fonial: 58 — passive pool still exhausted; top survivor (75 BOLA) gated on two program-approved tenants, Mattermost (55) on one approved POST; no writes, no customer data touched since 23:13 probe reconfirms zero drift. Reportable-outcome probability remains ~50% flat-close if outside approvals stall.
## 2026-09-09 15:23:19 UTC [target] (model bigpickle)
[PRIO] prov.fonial.de/api/2.0, 8.7, a=9,b=10,t=8,g=9,c=9,f=10
[PRIO] mm.fonial.de, 5.7, a=6,b=7,t=6,g=8,c=5,f=7
[PRIO] kundenkonto.fonial.de/api/2.0, 7.8, a=5,b=10,t=6,g=3,c=8,f=5
[PRIO] shop.fonial.de/graphql, 6.7, a=7,b=7,t=9,g=7,c=3,f=6
[HYP] Cross-tenant BOLA via dual-backend SID/PHPSESSID cross-bind on unfronted prov.fonial.de
class: AUTH
asset: prov.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: prov.fonial.de is a byte-behavior duplicate of kundenkonto.fonial.de API. Same X-Fonial-Version (v2026.09.03-1), same session semantics (cleartext UUID SID in body), same PHPSESSID decoration pattern, same ACAO * on all endpoints. Served directly on nginx/1.10.3 without Cloudflare — the lowest-gate deployment to test the cross-bind hypothesis. Data endpoints authorize by body SID only (`session unauthenticated` vs `session invalid`); session endpoint issues cleartext UUID sid; PHPSESSID is set decoratively. Cross-binding SID↔credential is the sole open question and is the highest-impact hypothesis in the entire engagement.
evidence_needed: program-approved tenants T_A/T_B on prov; POST /evn/get {"sid":SID_B} + Cookie: PHPSESSID_A → data (BOLA) vs session invalid (credential-bound).
verify_steps: (HUMAN, sanctioned) 1) GET /api/2.0/session → SID_A; 2) POST /session/authenticate{T_A}; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh jar) → SID_B; 5) POST /evn/get{sid:SID_B} + Cookie:PHPSESSID_A. 1 request each, ≤1rps, zero writes.
impact: cross-tenant CDR, device/SIP creds, unauthorized outbound call control on portal API. CRITICAL.
testability: HUMAN_ONLY
[HYP] Open self-signup on internal Mattermost 3.7 enables outsider account entry in company chat
class: BUSLOGIC
asset: mm.fonial.de
confidence: 55
reasoning: /signup/email returns 200 with no disabled-signup redirect; /api/v4/users/ping 401 (normal logged-out); X-Version-Id 3.7.3...; v4 config route absent (legacy build). Vendor default: email+password self-signup active when EnableSignUpWithEmail=true; real signup path never POSTed. Signup is mutating → program-approved only.
evidence_needed: POST /api/v4/users/create with fabricated email issues account token vs adminApproval/disabled error.
verify_steps: (HUMAN, sanctioned) single POST /api/v4/users/create {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>}; on success immediately delete/disable account; single GET /signup/email to confirm form fields first. ≤1rps.
impact: outsider foothold in company chat (internal directory, channels, webhooks) plus legacy-3.7 CVE surface. HIGH if open.
testability: HUMAN_ONLY
[HYP] Shop admin-token asymmetric surface
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: REST /V1/integration/admin/token → 404 (route removed) while GraphQL generateCustomerTokenAsAdmin is unauth-introspectable (input = single customer_email); Adobe docs require admin Bearer + customer opt-in. Residual = inverted/missing user-type check in this 2.4 CE build. No new evidence since downgrade.
evidence_needed: POST /graphql mutation{generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){customer_token}} without Authorization → authz-error vs token issuance.
verify_steps: (HUMAN, sanctioned) single POST, fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[PARKED] shop admin-token standalone (40): no new evidence since 55→40; relegate to chain-only, not primary.
[PARKED] prov v1/v2 version-diff probe: alternate API versions already 404 on kundenkonto; no evidence prov differs — needs a read-only drift check before crediting any divergence.
[PARKED] sbc/LDAP/RMQ/Sentry public-IP banner group (45): single-packet classification is HUMAN-only; no HTTP reachability evidence; unchanged.
[FINAL] prov.fonial.de cross-tenant BOLA (75) — top survivor, gated on TWO approved tenants.
[FINAL] mm.fonial.de open self-signup (55) — gated on ONE approved mutating POST, no tenant prerequisite.
[NEXT] HUMAN: request program approval for ONE sanctioned mutating POST, then run `POST https://mm.fonial.de/api/v4/users/create` {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>} after confirming form fields via GET /signup/email. Capture token-issuance vs adminApproval/disabled error; delete account immediately if created. This is the only hypothesis whose blocker is a single approval rather than two real test tenants; the 75 BOLA cross-bind remains queued behind T_A/T_B provisioning.
[LEARN] ACCEPTED prov.fonial.de duplicate @ prov.fonial.de/api/2.0: unfronted byte-behavior clone of kundenkonto API (same version, session semantics, PHPSESSID decoration, ACAO *) → lowest-gate deployment to test dual-backend cross-bind.
[LEARN] REJECTED shop admin-token standalone @ shop.fonial.de/graphql: confidence 40 at critique floor, no new evidence; drop as primary, retain only as chain component.
[RISK] fonial: 58 — passive pool exhausted; top survivor (75 BOLA) gated on two program-approved tenants, Mattermost (55) on one approved POST; no writes, no customer data touched since 23:13 probe reconfirms zero drift. Reportable-outcome probability ~50% flat-close if outside approvals stall.
## 2026-09-09 18:41:05 UTC [target] (model bigpickle)
[NEW] prov.fonial.de/api/2.0 accepted as unfronted byte-behavior duplicate of kundenkonto (nginx/1.10.3 direct, no Cloudflare) — lowest-gate deployment to test cross-bind hypothesis
[NEW] mm.fonial.de Mattermost 3.7.x with /signup/email 200, /api/v4/users/ping 401 — open self-signup untested (mutating → program-approved only)
[CHANGED] shop admin-token standalone confidence dropped to 40 (at critique floor) — relegate to chain-only, not primary
[PRIO] prov.fonial.de/api/2.0, 8.7, a=9,b=10,t=8,g=9,c=9,f=10
[PRIO] mm.fonial.de, 5.7, a=6,b=7,t=6,g=8,c=5,f=7
[PRIO] kundenkonto.fonial.de/api/2.0, 7.8, a=5,b=10,t=6,g=3,c=8,f=5
[PRIO] shop.fonial.de/graphql, 6.7, a=7,b=7,t=9,g=7,c=3,f=6
[HYP] Cross-tenant BOLA via dual-backend SID/PHPSESSID cross-bind on unfronted prov.fonial.de
class: AUTH
asset: prov.fonial.de/api/2.0 (devices/get, evn/get, call/initiate)
confidence: 75
reasoning: prov.fonial.de is a byte-behavior duplicate of kundenkonto.fonial.de API. Same X-Fonial-Version (v2026.09.03-1), same session semantics (cleartext UUID SID in body), same PHPSESSID decoration pattern, same ACAO * on all endpoints. Served directly on nginx/1.10.3 without Cloudflare — the lowest-gate deployment to test the cross-bind hypothesis. Data endpoints authorize by body SID only (`session unauthenticated` vs `session invalid`); session endpoint issues cleartext UUID sid; PHPSESSID is set decoratively. Cross-binding SID↔credential is the sole open question and is the highest-impact hypothesis in the entire engagement.
evidence_needed: program-approved tenants T_A/T_B on prov; POST /evn/get {"sid":SID_B} + Cookie: PHPSESSID_A → data (BOLA) vs session invalid (credential-bound).
verify_steps: (HUMAN, sanctioned) 1) GET /api/2.0/session → SID_A; 2) POST /session/authenticate{T_A}; 3) POST /devices/get{sid:SID_A} capture PHPSESSID_A; 4) GET /api/2.0/session (fresh jar) → SID_B; 5) POST /evn/get{sid:SID_B} + Cookie:PHPSESSID_A. 1 request each, ≤1rps, zero writes.
impact: cross-tenant CDR, device/SIP creds, unauthorized outbound call control on portal API. CRITICAL.
testability: HUMAN_ONLY
[HYP] Open self-signup on internal Mattermost 3.7 enables outsider account entry in company chat
class: BUSLOGIC
asset: mm.fonial.de
confidence: 55
reasoning: /signup/email returns 200 with no disabled-signup redirect; /api/v4/users/ping 401 (normal logged-out); X-Version-Id 3.7.3...; v4 config route absent (legacy build). Vendor default: email+password self-signup active when EnableSignUpWithEmail=true; real signup path never POSTed. Signup is mutating → program-approved only.
evidence_needed: POST /api/v4/users/create with fabricated email issues account token vs adminApproval/disabled error.
verify_steps: (HUMAN, sanctioned) single POST /api/v4/users/create {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>}; on success immediately delete/disable account; single GET /signup/email to confirm form fields first. ≤1rps.
impact: outsider foothold in company chat (internal directory, channels, webhooks) plus legacy-3.7 CVE surface. HIGH if open.
testability: HUMAN_ONLY
[HYP] Shop admin-token asymmetric surface
class: AUTH
asset: shop.fonial.de/graphql
confidence: 40
reasoning: REST /V1/integration/admin/token → 404 (route removed) while GraphQL generateCustomerTokenAsAdmin is unauth-introspectable (input = single customer_email); Adobe docs require admin Bearer + customer opt-in. Residual = inverted/missing user-type check in this 2.4 CE build. No new evidence since downgrade.
evidence_needed: POST /graphql mutation{generateCustomerTokenAsAdmin(input:{customer_email:"authz-probe-<ts>@example.invalid"}){customer_token}} without Authorization → authz-error vs token issuance.
verify_steps: (HUMAN, sanctioned) single POST, fabricated email, zero data exposure.
impact: ATO of opted-in shop accounts (orders, addresses, pay vault). CRITICAL if bypass.
testability: HUMAN_ONLY
[PARKED] shop admin-token standalone (40): no new evidence since 55→40; relegate to chain-only, not primary.
[PARKED] prov v1/v2 version-diff probe: alternate API versions already 404 on kundenkonto; no evidence prov differs — needs a read-only drift check before crediting any divergence.
[PARKED] sbc/LDAP/RMQ/Sentry public-IP banner group (45): single-packet classification is HUMAN-only; no HTTP reachability evidence; unchanged.
[FINAL] prov.fonial.de cross-tenant BOLA (75) — top survivor, gated on TWO approved tenants.
[FINAL] mm.fonial.de open self-signup (55) — gated on ONE approved mutating POST, no tenant prerequisite.
[NEXT] HUMAN: request program approval for ONE sanctioned mutating POST, then run `POST https://mm.fonial.de/api/v4/users/create` {email:ot-<ts>@example.invalid, username:ot-<ts>, password:<random>} after `GET /signup/email` to confirm form fields; capture token-issuance vs adminApproval/disabled error, delete account immediately if created. This is the only active hypothesis whose blocker is a single approval rather than two real test tenants; the 75 BOLA cross-bind remains queued behind T_A/T_B provisioning.
[LEARN] ACCEPTED prov.fonial.de duplicate @ prov.fonial.de/api/2.0: unfronted byte-behavior clone of kundenkonto API (same version, session semantics, PHPSESSID decoration, ACAO *) → lowest-gate deployment to test dual-backend cross-bind.
[LEARN] REJECTED shop admin-token standalone @ shop.fonial.de/graphql: confidence 40 at critique floor, no new evidence; Drop as primary, retain only as chain component.
[RISK] fonial: 58 — passive pool still exhausted; top survivor (75 BOLA) gated on two program-approved tenants, Mattermost (55) on one approved POST; no writes, no customer data touched since 23:13 probe reconfirms zero drift. Reportable-outcome probability remains ~50% flat-close if outside approvals stall.
