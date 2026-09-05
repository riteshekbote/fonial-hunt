# Hypotheses (ranked)

## RANKED HYPOTHESES 2026-09-02 21:58:51 UTC

## RANKED HYPOTHESES 2026-09-02 23:49:27 UTC

## RANKED HYPOTHESES 2026-09-03 02:50:19 UTC

## RANKED HYPOTHESES 2026-09-03 07:36:10 UTC

## RANKED HYPOTHESES 2026-09-03 12:17:36 UTC

## RANKED HYPOTHESES 2026-09-03 16:47:39 UTC
- [85] kundenkonto.fonial.de: Overly permissive CORS on authenticated customer portal (from art/lead_nemotron3.txt)
- [72] kundenkonto.fonial.de/api/2.0/session/authenticate: API session auth lacks brute-force protection enabling credential stuffing (from art/lead_bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ (with cookie from login redirect) to check CORS headers on authenticated landing page and enumerate any JSON API endpo
- NEXT(hypotheses-bigpickle.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session/authenticate — send 20 rapid requests with different passwords to the same SID to confirm no rate limi
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism

## RANKED HYPOTHESES 2026-09-03 19:31:01 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [72] kundenkonto.fonial.de/api/2.0/session/authenticate: API session auth lacks brute-force protection enabling credential stuffing (from art/lead_bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID, then POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test creden
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://kundenkonto.fonial.de/api/2.0/ to check for error messages or documentation links.
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism

## RANKED HYPOTHESES 2026-09-03 21:56:14 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [72] kundenkonto.fonial.de/api/2.0/session/authenticate: API session auth lacks brute-force protection enabling credential stuffing (from art/lead_bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID, then POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test creden
- NEXT(hypotheses-bigpickle.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session/authenticate — send 20 rapid requests with different passwords to the same SID to confirm no rate limi
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: data endpoints authorize by body SID only (`session unauthenticated` vs `session invalid`), PHPSE
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: out of scope (rate-limit/lockout policy) per program.
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues clearte
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: out of scope (rate-limit/lockout policy).

## RANKED HYPOTHESES 2026-09-03 23:48:44 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [55] kundenkonto.fonial.de/api/2.0: Data-layer SID authz bound to body SID, PHPSESSID parallel + not cross-checked → cross-tenant potential (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: obtain two valid test accounts on kundenkonto.fonial.de sandbox to execute the cross-binding test (sid-vs-PHPSESSID) on /api/2.0/evn/get. Passive automat
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues clearte
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: out of scope (rate-limit/lockout policy).
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues clearte
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues clearte
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program

## RANKED HYPOTHESES 2026-09-04 02:39:13 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [55] kundenkonto.fonial.de/api/2.0: Data-layer SID authz cross-tenant via session confusion (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://kundenkonto.fonial.de/api/2.0/devices/add — check 405 vs 404. If 405, expand to GET /api/2.0/extension/get, /user/get, /contact/get, /account
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
- LEARN: ACCEPTED CRUD pattern: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings.
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID parallel/decorative; /session issues clearte
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program

## RANKED HYPOTHESES 2026-09-04 07:33:52 UTC
- [70] kundenkonto.fonial.de/api/2.0: CRUD write endpoints bypass SID authorization (from art/lead_nemotron3.txt)
- [55] kundenkonto.fonial.de/api/2.0: Data-layer SID authz cross-tenant via session confusion (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://kundenkonto.fonial.de/api/2.0/devices/add — check 405 vs 404. If 405, expand to GET /api/2.0/extension/get, /user/get, /contact/get, /account
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
- LEARN: ACCEPTED CRUD pattern: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings.
- LEARN: ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- LEARN: ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references

## RANKED HYPOTHESES 2026-09-04 12:20:45 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [55] kundenkonto.fonial.de/api/2.0: Data-layer SID authz cross-tenant via session confusion (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://kundenkonto.fonial.de/api/2.0/devices/add — check 405 vs 404. If 405, expand to GET /api/2.0/extension/get, /user/get, /contact/get, /account
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
- LEARN: ACCEPTED CRUD pattern: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings.
- LEARN: ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references

## RANKED HYPOTHESES 2026-09-04 16:35:45 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [55] kundenkonto.fonial.de/api/2.0/call/initiate: call/initiate write endpoint bypasses SID authz or leaks cross-tenant call data (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: obtain 2 valid test accounts on kundenkonto.fonial.de to execute cross-binding test: (1) authenticate Account A → SID A → devices/get → PHPSESSID A; (2) 
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 gu
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz su
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits.
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy).
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative.
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist; ~50 guessed names all HTML-404.
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; same session-invalid JSON pattern.
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming.
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope.
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative.
- LEARN: ACCEPTED CRUD pattern @ kundenkonto.fonial.de/api/2.0: /{resource}/{action} confirmed by /devices/get, /evn/get; ~30 likely siblings
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanism
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing @ /api/2.0/session/authenticate: Out of scope (rate-limit/lockout policy) per program
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references

## RANKED HYPOTHESES 2026-09-04 19:22:01 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 gu
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz su
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits

## RANKED HYPOTHESES 2026-09-04 21:36:27 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend session confusion enables cross-tenant data access (from art/lead_nemotron3.txt)
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: obtain 2 valid test accounts; execute 2-account cross-bind matrix: for each of /devices/get, /evn/get, /call/initiate POST SID-B body + Cookie PHPSESSID=
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow
- LEARN: REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/set
- LEARN: ACCEPTED narrow-API @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; this closes passive discovery on the API (split-frontend + SPA routes enumerated).
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 gu
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz su
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references

## RANKED HYPOTHESES 2026-09-04 23:17:56 UTC
- [80] kundenkonto.fonial.de: Authenticated landing page inherits wildcard CORS without Vary: Origin (from art/lead_nemotron3.txt)
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://kundenkonto.fonial.de/ → follow to /login HTML → extract <script src> list → GET each JS bundle → grep 'api/2.0/' and {sid,number,caller,call
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 gu
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz su
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no allow-credentials, browser won't send cookies cross-origin 
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references

## RANKED HYPOTHESES 2026-09-05 01:10:40 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- [70] dslkonto.fonial.de: app_dev.php Symfony dev-mode debug exposure on DSL customer portal leaks internal paths and may expose further debug routes (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/ (200 dev boot, confirm no IP block), then GET https://dslkonto.fonial.de/app_dev.php/forgot/request and https
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://kundenkonto.fonial.de/ with Cookie: PHPSESSID=<from_login_redirect> to check CORS headers on authenticated landing page and enumerate any JSO
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 gu
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz su
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references

## RANKED HYPOTHESES 2026-09-05 05:52:24 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), then POST {"sid":"SID-B"} with Cookie PHPSESSID=A to /devices/get, /
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler to confirm Symfony profiler exposure and enumerate debug routes
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: Only 5 endpoints exist (session, session/authenticate, devices/get, evn/get, call/initiate); ~50 gu
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; returns same session-invalid JSON pattern as read endpoints; same SID-only authz su
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origi
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope (rate-limit/lockout policy)
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
