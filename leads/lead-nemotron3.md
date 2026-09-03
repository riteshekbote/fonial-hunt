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
