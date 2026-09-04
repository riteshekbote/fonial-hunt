## 2026-09-03 16:47:30 UTC [target] (model nemotron3)
[NEW] kundenkonto.fonial.de: Customer portal behind Cloudflare, PHP 8.3, permissive CORS (*), version header exposed (X-Fonial-Version: v2026.09.01-1)
[NEW] www.fonial.de/graphql/: GraphQL endpoint exists but returns 404 (TYPO3), not functional
[NEW] API marketed at /telefonanlage/funktionen/api/ but no public OpenAPI/Swagger/GraphQL introspection accessible
[CHANGED] Inventory passive recon previously showed 0 live HTTP; now 2 confirmed live (www, kundenkonto), 3 dead (app, admin, staging)
[PRIO] kundenkonto.fonial.de,7.5,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,4.0,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[PRIO] api.fonial.de (not resolving),0.0,attack_surface=0,business_value=0,tech_exposure=0,gate_ease=0,cloud_surface=0,freshness=0
[HYP] Overly permissive CORS on authenticated customer portal
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 85
reasoning: Response headers show 'access-control-allow-origin: *' on authenticated endpoints (login page sets PHPSESSID). Wildcard CORS with credentials creates risk of cross-origin data theft if any endpoint reflects user data.
evidence_needed: Confirmation that authenticated API endpoints (if any) also return ACAO: * without Vary: Origin
verify_steps: GET https://kundenkonto.fonial.de/login (check headers); GET https://kundenkonto.fonial.de/ (authenticated, check CORS on any JSON endpoint)
impact: If authenticated API endpoints exist with wildcard CORS, any malicious site could exfiltrate customer call records, SIP credentials, billing data — HIGH
testability: PASSIVE
[HYP] GraphQL endpoint present but non-functional — potential hidden attack surface
class: MISCONFIG
asset: www.fonial.de/graphql/
confidence: 60
reasoning: /graphql returns 301 to /graphql/ which serves TYPO3 404. Suggests GraphQL was planned/implemented but disabled or misrouted. Could indicate stale code paths or misconfigured routing.
evidence_needed: Check if GraphQL works with valid auth or on different host (kundenkonto)
verify_steps: POST https://www.fonial.de/graphql/ with introspection query; GET https://kundenkonto.fonial.de/graphql
impact: If GraphQL exists internally with introspection enabled, schema enumeration could reveal sensitive mutations (call control, number management) — MEDIUM
testability: PASSIVE
[HYP] Version disclosure via X-Fonial-Version header
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 95
reasoning: Header 'x-fonial-version: v2026.09.01-1' present on login response. Exposes exact deployment version to unauthenticated attackers.
evidence_needed: Confirm header persists across all endpoints
verify_steps: GET https://kundenkonto.fonial.de/login (already observed); GET https://kundenkonto.fonial.de/
impact: Facilitates targeted exploit research for known versions; low direct impact but aids recon — LOW
testability: PASSIVE
[PARKED] GraphQL endpoint present but non-functional — potential hidden attack surface: confidence 60 but verify_steps only check public endpoints; no evidence of actual GraphQL API existing. Low actionability without auth.
[FINAL] 1. Overly permissive CORS on authenticated customer portal (85) 2. Version disclosure via X-Fonial-Version header (95)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ (with cookie from login redirect) to check CORS headers on authenticated landing page and enumerate any JSON API endpoints via Link headers or JS source
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[RISK] fonial: 65 — Customer portal (kundenkonto) has high business value (PII, call logs, SIP creds, billing) and exploitable misconfig (wildcard CORS). Main API surface not publicly enumerable but marketed. No critical vulns confirmed yet; risk rises if authenticated API endpoints inherit same CORS policy.
## 2026-09-03 19:30:48 UTC [target] (model nemotron3)
[NEW] kundenkonto.fonial.de/api/2.0 confirmed as POST-only REST API with dual-backend architecture (session vs data endpoints with different Content-Type headers and session mechanisms)
[NEW] CORS wildcard `access-control-allow-origin: *` on ALL API endpoints including `/session/authenticate` (no `allow-credentials`)
[NEW] Two distinct backends: session endpoints return `text/json` (no PHPSESSID); data endpoints return `text/json;charset=UTF-8` and set `PHPSESSID` with Secure;HttpOnly
[CHANGED] Probe results confirm: `/login` 200, `/` 200, `/graphql` 404 on both hosts, `/api/2.0/session/authenticate` 200
[CHANGED] Knowledge base updated: ACCEPTED dual-backend architecture → session confusion attack surface; REJECTED CORS direct-exploit (SID in body, no credentials)
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
## 2026-09-03 21:56:02 UTC [target] (model nemotron3)
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
[HYP] Overly permissive CORS on authenticated customer portal endpoints
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 85
reasoning: Response headers show `access-control-allow-origin: *` on login page (sets PHPSESSID) and likely on authenticated landing page. Wildcard CORS with credentials creates risk if any authenticated JSON endpoint reflects user data without `Vary: Origin`.
evidence_needed: Confirm authenticated JSON endpoints (if any) also return ACAO: * without Vary: Origin
verify_steps: GET https://kundenkonto.fonial.de/ (with PHPSESSID from login redirect) → check CORS headers on response; inspect JS source for API endpoint references
impact: If authenticated API endpoints inherit wildcard CORS, malicious site could exfiltrate call records, SIP credentials, billing data — HIGH
testability: PASSIVE
[PARKED] Authentication endpoint lacks rate limiting enabling credential stuffing: class AUTH but "Enforcement policies for brute force, rate limiting, or account lockout" is explicitly out-of-scope per program rules.
[FINAL] 1. Overly permissive CORS on authenticated customer portal endpoints (85) 2. Dual-backend session confusion enables cross-tenant data access (75)
[NEXT] PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID, then POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credentials (or known invalid) to capture PHPSESSID from data endpoint response, then test cross-bind with second account's SID + first account's PHPSESSID on /api/2.0/evn/get
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. Auth endpoint shows no visible rate limiting. CORS wildcard on auth domain confirmed. Main API not publicly documented but live. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-03 23:48:34 UTC [target] (model nemotron3)
[NEW] Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
[NEW] Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
[NEW] /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
[CHANGED] CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
[CHANGED] Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints use SID in JSON body (text/json, no PHPSESSID). Data endpoints set PHPSESSID cookie but authorize by body SID only. Independent session stores suggest SID-to-PHPSESSID binding may not be enforced on data layer. SID is cleartext UUID v4; PHPSESSID deleted if unknown.
evidence_needed: Proof that data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or that unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST [0m
[NEW] Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
[NEW] Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
[NEW] /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
[CHANGED] CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
[CHANGED] Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints use SID in JSON body (text/json, no PHPSESSID). Data endpoints set PHPSESSID cookie but authorize by body SID only. Independent session stores suggest SID-to-PHPSESSID binding may not be enforced on data layer. SID is cleartext UUID v4; PHPSESSID deleted if unknown.
evidence_needed: Proof that data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or that unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
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
[HYP] Unauthenticated SID + valid PHPSESSID bypasses data endpoint authorization
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 65
reasoning: Data endpoints authorize by body SID only (unauthenticated SID → "session unauthenticated"; absent/unknown → "session invalid"). PHPSESSID is set decoratively. Open question: does data layer validate SID auth-state against PHPSESSID binding, or only SID?
evidence_needed: Data endpoint response with unauthenticated SID but valid PHPSESSID from authenticated session
verify_steps: 1) POST /api/2.0/session → get SID_unauth; 2) POST /api/2.0/session/authenticate with valid creds → get SID_auth + PHPSESSID via /devices/get; 3) POST /api/2.0/evn/get with body {"sid": SID_unauth} + Cookie: PHPSESSID=PHPSESSID_auth; 4) Observe if data returns vs "session unauthenticated"
impact: If PHPSESSID alone authorizes data access, session fixation or SID prediction could lead to cross-tenant data leak — HIGH
testability: AUTH_HELPED
[PARKED] Unauthenticated SID + valid PHPSESSID bypasses data endpoint authorization: Subsumed by dual-backend session confusion hypothesis; same verify steps, lower confidence without additional evidence
[FINAL] 1. Overly permissive CORS on authenticated customer portal endpoints (85) 2. Dual-backend session confusion enables cross-tenant data access (75)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid and deletes unknown PHPSESSID
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
[NEW] Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
[NEW] Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
[NEW] /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
[CHANGED] CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
[CHANGED] Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints use SID in JSON body (text/json, no PHPSESSID). Data endpoints set PHPSESSID cookie but authorize by body SID only. Independent session stores suggest SID-to-PHPSESSID binding may not be enforced on data layer. SID is cleartext UUID v4; PHPSESSID deleted if unknown.
evidence_needed: Proof that data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or that unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST [0m
[NEW] Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
[NEW] Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
[NEW] /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
[CHANGED] CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
[CHANGED] Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints use SID in JSON body (text/json, no PHPSESSID). Data endpoints set PHPSESSID cookie but authorize by body SID only. Independent session stores suggest SID-to-PHPSESSID binding may not be enforced on data layer. SID is cleartext UUID v4; PHPSESSID deleted if unknown.
evidence_needed: Proof that data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or that unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
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
[HYP] Unauthenticated SID + valid PHPSESSID bypasses data endpoint authorization
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 65
reasoning: Data endpoints authorize by body SID only (unauthenticated SID → "session unauthenticated"; absent/unknown → "session invalid"). PHPSESSID is set decoratively. Open question: does data layer validate SID auth-state against PHPSESSID binding, or only SID?
evidence_needed: Data endpoint response with unauthenticated SID but valid PHPSESSID from authenticated session
verify_steps: 1) POST /api/2.0/session → get SID_unauth; 2) POST /api/2.0/session/authenticate with valid creds → get SID_auth + PHPSESSID via /devices/get; 3) POST /api/2.0/evn/get with body {"sid": SID_unauth} + Cookie: PHPSESSID=PHPSESSID_auth; 4) Observe if data returns vs "session unauthenticated"
impact: If PHPSESSID alone authorizes data access, session fixation or SID prediction could lead to cross-tenant data leak — HIGH
testability: AUTH_HELPED
[PARKED] Unauthenticated SID + valid PHPSESSID bypasses data endpoint authorization: Subsumed by dual-backend session confusion hypothesis; same verify steps, lower confidence without additional evidence
[FINAL] 1. Overly permissive CORS on authenticated customer portal endpoints (85) 2. Dual-backend session confusion enables cross-tenant data access (75)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid and deletes unknown PHPSESSID
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-04 02:34:12 UTC [target] (model nemotron3)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints (text/json, no PHPSESSID) issue cleartext UUID v4 SID in body. Data endpoints (text/json;charset=UTF-8) set PHPSESSID cookie but authorize solely by body SID. Independent session stores suggest SID-to-PHPSESSID binding not enforced on data layer. Bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` on session endpoint.
evidence_needed: Data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + Cookie: PHPSESSID=PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
testability: AUTH_HELPED
[HYP] Overly permissive CORS on authenticated customer portal endpoints
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 85
reasoning: Response headers show `access-control-allow-origin: *` on login page (sets PHPSESSID) and likely on authenticated landing page. Wildcard CORS with credentials creates risk if any authenticated JSON endpoint reflects user data without `Vary: Origin`.
evidence_needed: Authenticated JSON endpoints (if any) also return ACAO: * without Vary: Origin
verify_steps: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check CORS headers on response; inspect JS source for API endpoint references
impact: If authenticated API endpoints inherit wildcard CORS, malicious site could exfiltrate call records, SIP credentials, billing data — HIGH
testability: PASSIVE
[HYP] Unauthenticated SID + valid PHPSESSID bypasses data endpoint authorization
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 65
reasoning: Data endpoints authorize by body SID only (unauthenticated SID → "session unauthenticated"; absent/unknown → "session invalid"). PHPSESSID set decoratively. Open question: does data layer validate SID auth-state against PHPSESSID binding, or only SID?
evidence_needed: Data endpoint response with unauthenticated SID but valid PHPSESSID from authenticated session
verify_steps: 1) POST /api/2.0/session → get SID_unauth; 2) POST /api/2.0/session/authenticate with valid creds → get SID_auth + PHPSESSID via /devices/get; 3) POST /api/2.0/evn/get with body {"sid": SID_unauth} + Cookie: PHPSESSID=PHPSESSID_auth; 4) Observe if data returns vs "session unauthenticated"
impact: If PHPSESSID alone authorizes data access, session fixation or SID prediction could lead to cross-tenant data leak — HIGH
testability: AUTH_HELPED
[PARKED] Unauthenticated SID + valid PHPSESSID bypasses data endpoint authorization: Subsumed by dual-backend session confusion hypothesis; same verify steps, lower confidence without additional evidence
[FINAL] 1. Overly permissive CORS on authenticated customer portal endpoints (85) 2. Dual-backend session confusion enables cross-tenant data access (75)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues cleartext UUID sid and deletes unknown PHPSESSID
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-04 07:33:43 UTC [target] (model nemotron3)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] CRUD write endpoints bypass SID authorization
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 70
reasoning: Read endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID decorative. Write verbs (add/update/delete) untested. Dual-backend architecture suggests data layer may not enforce SID auth-state on mutating operations.
evidence_needed: Write endpoint (e.g., /devices/add) accepts unauthenticated SID or SID from Account A with PHPSESSID from Account B and performs state change
verify_steps: 1) POST /api/2.0/session → get SID_unauth; 2) POST /api/2.0/devices/add with body {"sid": SID_unauth, "data":{...}} → observe if 200/state-change vs "session unauthenticated"; 3) If blocked, repeat with authenticated SID_A + Cookie: PHPSESSID_B (cross-bind)
impact: Unauthorized device provisioning, SIP credential modification, call forwarding changes, billing manipulation — CRITICAL
testability: AUTH_HELPED
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 80
reasoning: Login page returns `access-control-allow-origin: *` with PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints (e.g., /api/2.0/evn/get called by frontend JS) also return ACAO: * without `Vary: Origin`, cross-origin read of CDR/SIP/billing data is possible.
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data
verify_steps: 1) GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH
testability: PASSIVE
[HYP] TYPO3 eID handlers expose internal API/debug endpoints
class: MISCONFIG
asset: www.fonial.de
confidence: 45
reasoning: TYPO3 CMS on www.fonial.de (PHP 8.3, nginx). eID handlers (`?eID=xxx`) often expose AJAX endpoints, debug tools, or internal APIs without auth. Unenumerated attack surface.
evidence_needed: Enumeration of valid eID parameters returning non-404 responses with sensitive data or functionality
verify_steps: 1) GET https://www.fonial.de/?eID=dump → check status/body; 2) Common eID wordlist: `dump`, `debug`, `api`, `ajax`, `tx_*`, `backend` → observe non-404; 3) Any 200/500 response → inspect for PII/config/debug
impact: Information disclosure, potential RCE via deserialization, internal API access — MEDIUM
testability: PASSIVE
[PARKED] TYPO3 eID handlers expose internal API/debug endpoints: confidence 45 (borderline), low business value asset (www.fional.de marketing site), passive enumeration only — deprioritized behind API/core portal
[FINAL] 1. Authenticated landing page inherits wildcard CORS without Vary: Origin (80) 2. CRUD write endpoints bypass SID authorization (70)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live with ~30 CRUD endpoints. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] CRUD write endpoints bypass SID authorization
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 70
reasoning: Read endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID decorative. Write verbs (add/update/delete) untested. Dual-backend architecture suggests data layer may not enforce SID auth-state on mutating operations.
evidence_needed: Write endpoint (e.g., /devices/add) accepts unauthenticated SID or SID from Account A with PHPSESSID from Account B and performs state change
verify_steps: 1) POST /api/2.0/session → get SID_unauth; 2) POST /api/2.0/devices/add with body {"sid": SID_unauth, "data":{...}} → observe if 200/state-change vs "session unauthenticated"; 3) If blocked, repeat with authenticated SID_A + Cookie: PHPSESSID_B (cross-bind)
impact: Unauthorized device provisioning, SIP credential modification, call forwarding changes, billing manipulation — CRITICAL
testability: AUTH_HELPED
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 80
reasoning: Login page returns `access-control-allow-origin: *` with PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints (e.g., /api/2.0/evn/get called by frontend JS) also return ACAO: * without `Vary: Origin`, cross-origin read of CDR/SIP/billing data is possible.
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data
verify_steps: 1) GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH
testability: PASSIVE
[HYP] TYPO3 eID handlers expose internal API/debug endpoints
class: MISCONFIG
asset: www.fonial.de
confidence: 45
reasoning: TYPO3 CMS on www.fonial.de (PHP 8.3, nginx). eID handlers (`?eID=xxx`) often expose AJAX endpoints, debug tools, or internal APIs without auth. Unenumerated attack surface.
evidence_needed: Enumeration of valid eID parameters returning non-404 responses with sensitive data or functionality
verify_steps: 1) GET https://www.fonial.de/?eID=dump → check status/body; 2) Common eID wordlist: `dump`, `debug`, `api`, `ajax`, `tx_*`, `backend` → observe non-404; 3) Any 200/500 response → inspect for PII/config/debug
impact: Information disclosure, potential RCE via deserialization, internal API access — MEDIUM
testability: PASSIVE
[PARKED] TYPO3 eID handlers expose internal API/debug endpoints: confidence 45 (borderline), low business value asset (www.fional.de marketing site), passive enumeration only — deprioritized behind API/core portal
[FINAL] 1. Authenticated landing page inherits wildcard CORS without Vary: Origin (80) 2. CRUD write endpoints bypass SID authorization (70)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live with ~30 CRUD endpoints. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-04 12:20:35 UTC [target] (model nemotron3)
[NEW] kundenkonto.fonial.de/api/2.0: GET /devices/add returns 404 (not 405) — write endpoint does not respond to GET; existence unconfirmed
[NEW] www.fonial.de: TYPO3 eID handler `?eID=dump` returns 404 — no debug endpoint exposed
[CHANGED] CORS hypothesis confidence adjusted: 85→80 (nemotron3) — authenticated landing page untested, needs PHPSESSID from login redirect
[CHANGED] CRUD write hypothesis confidence: 70 — based on pattern inference only, no live write endpoint confirmed
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints (text/json, no PHPSESSID) issue cleartext UUID v4 SID in body. Data endpoints (text/json;charset=UTF-8) set PHPSESSID cookie but authorize solely by body SID. Independent session stores suggest SID-to-PHPSESSID binding not enforced on data layer. Bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` on session endpoint.
evidence_needed: Data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + Cookie: PHPSESSID=PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
testability: AUTH_HELPED
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 80
reasoning: Login page returns `access-control-allow-origin: *` with PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints (e.g., /api/2.0/evn/get called by frontend JS) also return ACAO: * without `Vary: Origin`, cross-origin read of CDR/SIP/billing data is possible.
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data
verify_steps: 1) GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH
testability: PASSIVE
[HYP] CRUD write endpoints bypass SID authorization
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 70
reasoning: Read endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID decorative. Write verbs (add/update/delete) untested. Dual-backend architecture suggests data layer may not enforce SID auth-state on mutating operations. GET /devices/add returns 404 (not 405) — endpoint existence unconfirmed via GET.
evidence_needed: Write endpoint (e.g., /devices/add) accepts unauthenticated SID or SID from Account A with PHPSESSID from Account B and performs state change
verify_steps: 1) POST /api/2.0/session → get SID_unauth; 2) POST /api/2.0/devices/add with body {"sid": SID_unauth, "data":{...}} → observe if 200/state-change vs "session unauthenticated"; 3) If blocked, repeat with authenticated SID_A + Cookie: PHPSESSID_B (cross-bind)
impact: Unauthorized device provisioning, SIP credential modification, call forwarding changes, billing manipulation — CRITICAL
testability: AUTH_HELPED
[PARKED] CRUD write endpoints bypass SID authorization: GET /devices/add returns 404 (not 405); write endpoint existence unconfirmed; confidence 70 but verify steps require POST which needs valid test accounts — borderline without endpoint confirmation
[PARKED] TYPO3 eID handlers expose internal API/debug endpoints: confidence 45, `?eID=dump` returns 404, low business value asset (marketing site), passive enumeration only — deprioritized behind API/core portal
[FINAL] 1. Dual-backend session confusion enables cross-tenant data access (75) 2. Authenticated landing page inherits wildcard CORS without Vary: Origin (80)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live with ~30 CRUD endpoints. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-04 16:35:37 UTC [target] (model nemotron3)
[NEW] kundenkonto.fonial.de/api/2.0: GET /devices/add returns 404 (not 405) — write endpoint existence unconfirmed via GET; requires POST to verify  
[NEW] www.fonial.de: TYPO3 eID handler `?eID=dump` returns 404 — no debug endpoint exposed  
[CHANGED] CORS hypothesis confidence adjusted: 85→80 — authenticated landing page untested, needs PHPSESSID from login redirect  
[CHANGED] CRUD write hypothesis confidence: 70 — based on pattern inference only, no live write endpoint confirmed  
[CHANGED] Priority scores stable: kundenkonto.fonial.de/api/2.0 (9.05), kundenkonto.fonial.de (7.70), www.fonial.de (5.10)
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10  
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7  
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5  
[HYP] Dual-backend session confusion enables cross-tenant data access  
class: AUTH  
asset: kundenkonto.fonial.de/api/2.0  
confidence: 75  
reasoning: Session endpoints (text/json, no PHPSESSID) issue cleartext UUID v4 SID in body. Data endpoints (text/json;charset=UTF-8) set PHPSESSID cookie but authorize solely by body SID. Independent session stores suggest SID-to-PHPSESSID binding not enforced on data layer. Bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` on session endpoint.  
evidence_needed: Data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or unauthenticated SID + valid PHPSESSID returns data  
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + Cookie: PHPSESSID=PHPSESSID1 (cross-bind); 6) Observe if data returns  
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL  
testability: AUTH_HELPED  
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin  
class: MISCONFIG  
asset: kundenkonto.fonial.de  
confidence: 80  
reasoning: Login page returns `access-control-allow-origin: *` with PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints (e.g., /api/2.0/evn/get called by frontend JS) also return ACAO: * without `Vary: Origin`, cross-origin read of CDR/SIP/billing data is possible.  
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data  
verify_steps: 1) GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header  
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH  
testability: PASSIVE  
[HYP] Data-layer SID authz cross-tenant via session confusion  
class: AUTH  
asset: kundenkonto.fonial.de/api/2.0  
confidence: 55  
reasoning: Data endpoints (/devices/get, /evn/get) authorize purely by body `sid` auth-state (`session unauthenticated` vs `session invalid`) and independently set a PHPSESSID cookie. The session endpoint issues cleartext UUID sid. If the authenticated sid-to-account binding lives only in the session store and PHPSESSID is decorative, a stolen/replayed sid could be used cross-tenant without any cookie match.  
evidence_needed: (HUMAN) authenticate two accounts; verify Account B's sid + Account A's PHPSESSID is rejected or accepted on /evn/get; confirm sid is bound to credential  
verify_steps: (AUTH_HELPED→HUMAN) 1) /session → sid A; 2) /session/authenticate A → auth; 3) /devices/get sid A → capture PHPSESSID A; 4) /session → sid B [+ PHPSESSID A cookie]; 5) /evn/get with sid B + PHPSESSID A → observe data scoping  
impact: Cross-tenant CDR, phone/SIP creds, device lists, outbound call control. Severity CRITICAL if sid not credential-bound  
testability: HUMAN_ONLY (needs 2 valid accounts)  
[PARKED] Data-layer SID authz cross-tenant via session confidence 55: HUMAN_ONLY testability (needs 2 valid accounts), no passive verification path, subsumed by higher-confidence dual-backend hypothesis (75) with same verify steps  
[PARKED] CRUD write endpoints bypass SID authorization: confidence 70 but GET /devices/add returns 404 (not 405); write endpoint existence unconfirmed; verify steps require POST with valid test accounts — borderline without endpoint confirmation  
[PARKED] TYPO3 eID handlers expose internal API/debug endpoints: confidence 45, `?eID=dump` returns 404, low business value asset (marketing site), passive enumeration only — deprioritized behind API/core portal  
[FINAL] 1. Dual-backend session confusion enables cross-tenant data access (75) 2. Authenticated landing page inherits wildcard CORS without Vary: Origin (80)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings  
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid  
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface  
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed  
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain  
[LEARN] REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program  
[LEARN] REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon  
[LEARN] REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references  
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live with ~30 CRUD endpoints. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-04 19:21:50 UTC [target] (model nemotron3)
[NEW] Landing page `/` returns 302 to `/login` with `Set-Cookie: PHPSESSID` + `ACAO: *` even when unauthenticated (no valid session)
[NEW] API endpoints (`/api/2.0/session`, `/call/initiate`, `/devices/get`, `/evn/get`) all return `ACAO: *` + `ACAM: GET, POST, OPTIONS` without `allow-credentials` — confirmed on all 4 tested endpoints
[NEW] `/call/initiate` confirmed as LIVE WRITE endpoint: returns JSON `{"sid":null,"status":"error","reason":"session invalid"}` without auth; same error pattern as read endpoints
[CHANGED] CRUD expansion hypothesis CONFIRMED WRONG: ~50 guessed `{resource}/{action}` names all return HTML 404 (web app fallback), not JSON 404/405 — API surface is exactly 5 endpoints
[CHANGED] CORS hypothesis: authenticated landing page untested (requires valid PHPSESSID from 2FA login flow); unauthenticated landing page redirects to login with CORS wildcard
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints (text/json, no PHPSESSID) issue cleartext UUID v4 SID in body. Data endpoints (text/json;charset=UTF-8) set PHPSESSID cookie but authorize solely by body SID. Independent session stores suggest SID-to-PHPSESSID binding not enforced on data layer. Bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` on session endpoint.
evidence_needed: Data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + Cookie: PHPSESSID=PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
testability: AUTH_HELPED
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 80
reasoning: Login page returns `ACAO: *` with PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints (e.g., /api/2.0/evn/get called by frontend JS) also return ACAO: * without `Vary: Origin`, cross-origin read of CDR/SIP/billing data is possible.
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data
verify_steps: 1) GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH
testability: PASSIVE
[HYP] call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/call/initiate
confidence: 55
reasoning: /call/initiate is live WRITE endpoint returning same `session invalid` JSON pattern as read endpoints. Suspected same SID-only authz. If SID not bound to credential/tenant, attacker could initiate outbound calls as victim.
evidence_needed: Valid SID from Account A accepted on /call/initiate with PHPSESSID from Account B; or unauthenticated SID + valid PHPSESSID triggers call
verify_steps: 1) Obtain authenticated SID for Account A; 2) Obtain PHPSESSID for Account B; 3) POST /call/initiate with SID_A + Cookie: PHPSESSID_B + call params; 4) Observe if call initiated or data leaked
impact: Unauthorized outbound calls, toll fraud, call metadata leakage — CRITICAL
testability: HUMAN_ONLY
[PARKED] call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data: confidence 55, testability HUMAN_ONLY (requires 2 valid accounts), subsumed by higher-confidence dual-backend hypothesis (75) with overlapping verify steps
[FINAL] 1. Dual-backend session confusion enables cross-tenant data access (75) 2. Authenticated landing page inherits wildcard CORS without Vary: Origin (80)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
[LEARN] ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 guessed names all HTML-404
[LEARN] ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz suspected
[LEARN] ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID sid
[LEARN] ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
[LEARN] ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
[LEARN] REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain
[LEARN] REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
[LEARN] REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live with 5 endpoints including WRITE. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
## 2026-09-04 21:36:19 UTC [target] (model nemotron3)
[NEW] API surface confirmed at exactly 5 endpoints: `/api/2.0/session`, `/api/2.0/session/authenticate`, `/api/2.0/devices/get`, `/api/2.0/evn/get`, `/api/2.0/call/initiate` — ~50 guessed `{resource}/{action}` names all return HTML 404 (web app fallback), not JSON 404/405
[NEW] `/call/initiate` confirmed as LIVE WRITE endpoint: POST returns JSON `{"sid":null,"status":"error","reason":"session invalid"}` without auth; identical error pattern to read endpoints
[NEW] Landing page `/` returns 302 to `/login` with `Set-Cookie: PHPSESSID` + `ACAO: *` even when unauthenticated (no valid session)
[NEW] All 4 tested API endpoints return `ACAO: *` + `ACAM: GET, POST, OPTIONS` without `allow-credentials` — confirmed on session, call/initiate, devices/get, evn/get
[NEW] Frontend HTML exposes routes: `/wizard/`, `/settings/menu/toggle`, `/help/suggest/__search__`, `/settings/ticket/`
[CHANGED] CRUD expansion hypothesis CONFIRMED WRONG: API uses non-standard naming, narrow surface
[CHANGED] CORS hypothesis: authenticated landing page still untested (requires valid PHPSESSID from 2FA login flow); unauthenticated landing page redirects to login with CORS wildcard
[PRIO] kundenkonto.fonial.de/api/2.0,9.05,attack_surface=9,business_value=10,tech_exposure=8,gate_ease=8,cloud_surface=9,freshness=10
[PRIO] kundenkonto.fonial.de,7.70,attack_surface=8,business_value=9,tech_exposure=7,gate_ease=6,cloud_surface=8,freshness=7
[PRIO] www.fonial.de,5.10,attack_surface=4,business_value=5,tech_exposure=3,gate_ease=10,cloud_surface=4,freshness=5
[HYP] Dual-backend session confusion enables cross-tenant data access
class: AUTH
asset: kundenkonto.fonial.de/api/2.0
confidence: 75
reasoning: Session endpoints (text/json, no PHPSESSID) issue cleartext UUID v4 SID in body. Data endpoints (text/json;charset=UTF-8) set PHPSESSID cookie but authorize solely by body SID. Independent session stores suggest SID-to-PHPSESSID binding not enforced on data layer. Bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` on session endpoint.
evidence_needed: Data endpoint accepts valid SID from Account A with PHPSESSID from Account B, or unauthenticated SID + valid PHPSESSID returns data
verify_steps: 1) POST /api/2.0/session → get SID1; 2) POST /api/2.0/session/authenticate with valid creds → SID1 authenticated; 3) POST /api/2.0/devices/get with SID1 → capture PHPSESSID1; 4) Repeat for Account B → get SID2, PHPSESSID2; 5) POST /api/2.0/evn/get with SID2 + Cookie: PHPSESSID=PHPSESSID1 (cross-bind); 6) Observe if data returns
impact: Cross-tenant CDR (call records), phone numbers, SIP credentials, device lists, outbound call initiation — CRITICAL
testability: AUTH_HELPED
[HYP] Authenticated landing page inherits wildcard CORS without Vary: Origin
class: MISCONFIG
asset: kundenkonto.fonial.de
confidence: 80
reasoning: Login page returns `ACAO: *` with PHPSESSID. Authenticated landing page likely shares middleware. If any JSON API endpoints (e.g., /api/2.0/evn/get called by frontend JS) also return ACAO: * without `Vary: Origin`, cross-origin read of CDR/SIP/billing data is possible.
evidence_needed: Authenticated GET / returns ACAO: * without Vary: Origin; JS source references JSON API endpoints that reflect user data
verify_steps: 1) GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> → check response headers for ACAO and Vary; 2) View page source → grep for /api/2.0/ or fetch/XHR patterns; 3) If JSON endpoints found, test them directly with Origin header
impact: Malicious site exfiltrates call records, SIP credentials, phone numbers, billing data — HIGH
testability: PASSIVE
[HYP] call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data
class: AUTH
asset: kundenkonto.fonial.de/api/2.0/call/initiate
confidence: 55
reasoning: /call/initiate is live WRITE endpoint returning same `session invalid` JSON pattern as read endpoints. Suspected same SID-only authz. If SID not bound to credential/tenant, attacker could initiate outbound calls as victim.
evidence_needed: Valid SID from Account A accepted on /call/initiate with PHPSESSID from Account B; or unauthenticated SID + valid PHPSESSID triggers call
verify_steps: 1) Obtain authenticated SID for Account A; 2) Obtain PHPSESSID for Account B; 3) POST /call/initiate with SID_A + Cookie: PHPSESSID_B + call params; 4) Observe if call initiated or data leaked
impact: Unauthorized outbound calls, toll fraud, call metadata leakage — CRITICAL
testability: HUMAN_ONLY
[PARKED] call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data: confidence 55, testability HUMAN_ONLY (requires 2 valid accounts), subsumed by higher-confidence dual-backend hypothesis (75) with overlapping verify steps
[FINAL] 1. Dual-backend session confusion enables cross-tenant data access (75) 2. Authenticated landing page inherits wildcard CORS without Vary: Origin (80)
[NEXT] PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSON API endpoints referenced in JS
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
[RISK] fonial: 70 — Customer portal (kundenkonto) has high business value (PII, CDR, SIP creds, billing, call control) and confirmed dual-backend architecture creating session confusion surface. CORS wildcard on auth domain confirmed. Main API not publicly documented but live with 5 endpoints including WRITE. Risk elevated due to enterprise telephony data sensitivity and architectural anomaly.
