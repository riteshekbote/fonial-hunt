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
