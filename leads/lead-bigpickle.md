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
