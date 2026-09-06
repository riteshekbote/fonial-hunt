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

## RANKED HYPOTHESES 2026-09-05 09:55:04 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- [75] dslkonto.fonial.de: DSL portal default-product misconfiguration blocks registration — public self-signup on dslkonto dead, product-param logic may mis-handle attacker-supplied product selection elsewhere (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/ (200 dev boot, confirm no IP block), then GET https://dslkonto.fonial.de/app_dev.php/forgot/request and https
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

## RANKED HYPOTHESES 2026-09-05 13:20:03 UTC
- [85] dslkonto.fonial.de: Symfony dev-mode profiler token exposure enables route/config enumeration on dslkonto (from art/lead_nemotron3.txt)
- [25] kundenkonto.fonial.de/wizard/,: Frontend wizard/settings routes expose unauthenticated actions (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: obtain 2 valid test accounts; execute 2-account cross-bind matrix: for each of /devices/get, /evn/get, /call/initiate POST SID-B body + Cookie PHPSESSID=
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/0951e9 to confirm profiler UI access and enumerate debug routes
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow
- LEARN: REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/set
- LEARN: ACCEPTED narrow-API @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; this closes passive discovery on the API (split-frontend + SPA routes enumerated).
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names) only, no env/param/session dump
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e
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

## RANKED HYPOTHESES 2026-09-05 16:25:11 UTC
- [85] dslkonto.fonial.de: Symfony dev-mode profiler token rotation enables route/config enumeration on dslkonto (from art/lead_nemotron3.txt)
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — GET /api/2.0/session twice (SID-A, SID-B), authenticate both, then POST {"sid":"SID-B"} with Cookie: PHPSESSID=
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken. Dead.
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints. Passive discovery closed.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e

## RANKED HYPOTHESES 2026-09-05 18:29:23 UTC
- [85] dslkonto.fonial.de: Symfony dev-mode profiler token rotation enables route/config enumeration on dslkonto (from art/lead_nemotron3.txt)
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — register or obtain 2 valid fonial accounts (A + B), then: (1) POST /api/2.0/session → SID-A; POST /api/2.0/sess
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints. All passive discovery methods exhausted (~50 guessed names returned HTML-404).
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint confirmed; returns identical session-invalid JSON pattern as read endpoints; same SI
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no allow-credentials, browser won't send cookies cross-origin 
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope per program (rate-limit/lockout policy).
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits.
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon.
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references.
- LEARN: REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/set
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e

## RANKED HYPOTHESES 2026-09-05 20:46:45 UTC
- [85] dslkonto.fonial.de: Symfony dev-mode profiler token rotation enables route/config enumeration on dslkonto (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints. All passive discovery methods exhausted (~50 guessed names returned HTML-404).
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint confirmed; returns identical session-invalid JSON pattern as read endpoints; same SI
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard (ACAO *, ACAM GET/POST/OPTIONS) consistent on all 5 API endpoints + both landing/login pages; no allow
- LEARN: REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no allow-credentials, browser won't send cookies cross-origin 
- LEARN: REJECTED brute-force/credential-stuffing: Out of scope per program (rate-limit/lockout policy).
- LEARN: REJECTED CRUD expansion ~30 siblings: PROVEN WRONG — API uses non-standard naming; guessed {resource}/{action} pattern yields 0 new hits.
- LEARN: REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon.
- LEARN: REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references.
- LEARN: REJECTED frontend-unauth @ kundenkonto.fonial.de: /wizard/, /settings/menu/toggle, /settings/ticket/ all 302 → /login; frontend auth-gated; no unauth config/set
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e

## RANKED HYPOTHESES 2026-09-05 22:40:15 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — GET/POST /api/2.0/session for SID-A and SID-B; authenticate both tenants; then POST {"sid":"SID-B"} + Cookie: P
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless. Co
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e

## RANKED HYPOTHESES 2026-09-06 00:18:57 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 04:48:50 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — get/register 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/a
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 09:11:58 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_bigpickle.txt)
- [70] kundenkonto.fonial.de: Authenticated landing page inherits wildcard CORS without Vary: Origin (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — get/register 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/a
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://dslkonto.fonial.de/app_dev.php/_profiler/031fa8 to confirm profiler UI access and enumerate debug routes
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: body SID only; PHPSESSID decorative; /session cleartext UUID sid.
- LEARN: ACCEPTED dual-backend @ kundenkonto.fonial.de/api/2.0: two distinct servers, different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body.
- LEARN: REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths (deploy tag, bundle names, exception semantics) only, no e
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 12:57:34 UTC
- [100] github.com/fonial-de,: No public fonial GmbH repositories found to audit (from art/lead_bigpickle.txt)
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/a
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: body SID only; PHPSESSID decorative; /session cleartext UUID sid.
- LEARN: ACCEPTED dual-backend @ kundenkonto.fonial.de/api/2.0: two distinct servers, different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body.
- LEARN: REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 16:13:10 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/a
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
- LEARN: PASSIVE-CLOSED: All passive discovery methods exhausted on all in-scope assets; dual-backend cross-bind hypothesis (75, HUMAN_ONLY) is sole survivor; no new pas
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body required.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 18:10:57 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/a
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
- LEARN: PASSIVE-CLOSED: All passive discovery methods exhausted on all in-scope assets; dual-backend cross-bind hypothesis (75, HUMAN_ONLY) is sole survivor; no new pas
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: Data endpoints authorize by body SID only; PHPSESSID decorative; /session issues cleartext UUID s
- LEARN: ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body required.
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
- LEARN: REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 20:31:22 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): HUMAN: kundenkonto 2-account cross-bind matrix — register/get 2 tenants (2nd via LIVE /signup/register/55 with CSRF `fonial_user_registration[_token]` + email/a
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
- LEARN: ACCEPTED narrow API surface @ kundenkonto.fonial.de/api/2.0: exactly 5 endpoints; ~50 guesses HTML-404. Passive discovery closed.
- LEARN: ACCEPTED call/initiate @ kundenkonto.fonial.de/api/2.0: Live WRITE endpoint; identical session-invalid pattern; SID-only authz.
- LEARN: ACCEPTED dual-session binding @ kundenkonto.fonial.de/api/2.0: body SID only; PHPSESSID decorative; /session cleartext UUID sid.
- LEARN: ACCEPTED dual-backend @ kundenkonto.fonial.de/api/2.0: two distinct servers, different response headers.
- LEARN: ACCEPTED MISCONFIG @ kundenkonto.fonial.de: CORS wildcard on all 5 API endpoints + landing/login; no allow-credentials → no cookie exfil; SID-in-body.
- LEARN: REJECTED subdomain-takeover/dev-mode-exposure/profiler/buslogic/CRUD/brute-force/SSRF/www/IDOR/www/CORS-direct-exploit: unchanged, closed/excluded.
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.

## RANKED HYPOTHESES 2026-09-06 22:22:42 UTC
- [75] kundenkonto.fonial.de/api/2.0: Dual-backend SID/PHPSESSID cross-binding enables cross-tenant access on all 3 data endpoints (from art/lead_nemotron3.txt)
- [50] shop.fonial.de/graphql: GraphQL introspection on shop.fonial.de exposes unauth reachable queries/mutations beyond catalog (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: POST https://shop.fonial.de/graphql `{"query":"{ __schema { queryType{name} mutationType{name} types{name kind} } }"}` — read-only schema map, ≤1 rps, no
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://kundenkonto.fonial.de/api/2.0/session → get SID1; POST https://kundenkonto.fonial.de/api/2.0/session/authenticate with valid test credential
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
- LEARN: REJECTED dev-mode-exposure @ dslkonto.fonial.de/app_dev.php: leaked content = stack traces + fs paths only; scope.yml excludes "Descriptive error messages or he
- LEARN: REJECTED dslkonto buslogic: no attacker-controllable params; registration broken on prod/dev. Dead.
- LEARN: ACCEPTED subdomain-takeover @ fonial.de dead hosts: app/admin/staging all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs → takeover class clo
- LEARN: ACCEPTED legacy-box @ go.fonial.de: Composer platform check (requires PHP >7.2.5) aborts all routing → HTTP 500 on every path; /app_dev.php→301. Non-bootable; o
- LEARN: REJECTED profiler-access @ dslkonto.fonial.de/app_dev.php/_profiler/{token}: 404 on all rotating tokens incl. 031fa8; class excluded by scope.yml regardless.
