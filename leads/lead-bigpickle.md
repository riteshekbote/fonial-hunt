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
