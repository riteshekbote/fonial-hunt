# fonial GmbH inventory (discovery seed 2026-09-02)
# NOTE: hosts below are discovery candidates from passive DNS/CT; confirm in-scope vs program scope before active testing.
admin.fonial.de
app.fonial.de
fonial.de
staging.fonial.de
www.fonial.de

## PASSIVE RECON 2026-09-02 (read-only, non-intrusive)

> Recon observations only. These are NOT confirmed vulnerabilities; ownership/in-scope of each host must be confirmed against the program scope before any active testing. Hosts resolve + serve HTTP — investigation requires scoped authorization.

**Probed:** 5 hosts | **Live HTTP:** 0

| Host | Status | Server/Tech |
|---|---|---|

## 2026-09-02 21:58:51 UTC

## 2026-09-02 23:49:27 UTC

## 2026-09-03 02:50:19 UTC

## 2026-09-03 07:36:10 UTC

## 2026-09-03 12:17:36 UTC

## 2026-09-03 16:47:39 UTC
- NEW kundenkonto.fonial.de: Customer portal behind Cloudflare, PHP 8.3, permissive CORS (*), version header exposed (X-Fonial-Version: v2026.09.01-1)
- NEW www.fonial.de/graphql/: GraphQL endpoint exists but returns 404 (TYPO3), not functional
- NEW API marketed at /telefonanlage/funktionen/api/ but no public OpenAPI/Swagger/GraphQL introspection accessible
- CHANGED Inventory passive recon previously showed 0 live HTTP; now 2 confirmed live (www, kundenkonto), 3 dead (app, admin, staging)
- NEW kundenkonto.fonial.de — Customer portal + API host (Cloudflare-fronted, version v2026.09.01-1)
- NEW kundenkonto.fonial.de/api/2.0 — Live REST API (POST-only, JSON body, session-based auth)
- NEW CORS wildcard `access-control-allow-origin: *` on ALL API endpoints including /session/authenticate
- NEW Two backend systems: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID)
- NEW www.fonial.de/shop/ — Hardware e-commerce (Magento-like, redirects from fonial.de/shop)
- NEW www.fonial.de/hilfe/ — Help center (separate PHP app)
- NEW www.fonial.de — TYPO3 CMS, PHP/8.3.3, nginx/1.31.2
- CHANGED fonial.de → 301 to www.fonial.de (was previously unresolved; nginx/1.31.2 confirmed)
- CHANGED app.fonial.de — Transport error (DNS/SSL unreachable)
- CHANGED admin.fonial.de — Transport error (DNS/SSL unreachable)
- CHANGED staging.fonial.de — Timeout (unreachable)

## 2026-09-03 19:31:01 UTC
- NEW kundenkonto.fonial.de/api/2.0 confirmed as POST-only REST API with dual-backend architecture (session vs data endpoints with different Content-Type headers and session mechanisms)
- NEW CORS wildcard `access-control-allow-origin: *` on ALL API endpoints including `/session/authenticate` (no `allow-credentials`)
- NEW Two distinct backends: session endpoints return `text/json` (no PHPSESSID); data endpoints return `text/json;charset=UTF-8` and set `PHPSESSID` with Secure;HttpOnly
- CHANGED Probe results confirm: `/login` 200, `/` 200, `/graphql` 404 on both hosts, `/api/2.0/session/authenticate` 200
- CHANGED Knowledge base updated: ACCEPTED dual-backend architecture → session confusion attack surface; REJECTED CORS direct-exploit (SID in body, no credentials)

## 2026-09-03 21:56:14 UTC
- NEW Two backend systems: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID)
- NEW www.fonial.de/shop/ — Hardware e-commerce (Magento-like, redirects from fonial.de/shop)
- NEW www.fonial.de/hilfe/ — Help center (separate PHP app)
- NEW www.fonial.de — TYPO3 CMS, PHP/8.3.3, nginx/1.31.2
- CHANGED fonial.de → 301 to www.fonial.de (was previously unresolved; nginx/1.31.2 confirmed)
- CHANGED app.fonial.de — Transport error (DNS/SSL unreachable)
- CHANGED admin.fonial.de — Transport error (DNS/SSL unreachable)
- CHANGED staging.fonial.de — Timeout (unreachable)
- NEW API dual-session binding confirmed: data endpoints (/devices/get, /evn/get) set PHPSESSID AND read authz SID from body — unauthenticated SID -> `"reason":"session unauthenticated"`; absent/unknown SID
- NEW /api/2.0/session without cookie returns `{"status":"ok","sid":"<uuid4>"}` and sets NO PHPSESSID; presenting a bogus PHPSESSID header causes it to reply with `PHPSESSID=deleted; Max-Age=0`.
- NEW Confirmed SID is cleartext UUID v4 returned in body; data authz is bound to body SID auth-state, NOT to PHPSESSID cookie (parallel/session-confusion surface).
- NEW Data endpoints (/devices/get, /evn/get) authorize by body SID only: unauthenticated SID -> `"reason":"session unauthenticated"`; absent/unknown -> `"reason":"session invalid"`. PHPSESSID set decorativ
- NEW /api/2.0/session (no cookie) -> `{"status":"ok","sid":"<uuid4>"}` no PHPSESSID; bogus PHPSESSID header -> reply `PHPSESSID=deleted; Max-Age=0`. SID is cleartext UUID in body.

## 2026-09-03 23:48:44 UTC
- NEW Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
- NEW Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
- NEW /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
- CHANGED CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
- CHANGED Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
- NEW Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
- NEW Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
- NEW /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
- CHANGED CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
- CHANGED Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
- NEW Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
- NEW Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
- NEW /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
- CHANGED CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
- CHANGED Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules
- NEW Confirmed dual-backend session confusion: session endpoints (text/json, no PHPSESSID) vs data endpoints (text/json;charset=UTF-8, sets PHPSESSID) operate independently
- NEW Data endpoints (/devices/get, /evn/get) authorize by body SID only; PHPSESSID cookie set decoratively in parallel
- NEW /api/2.0/session returns cleartext UUID v4 SID in body; presenting bogus PHPSESSID causes `PHPSESSID=deleted; Max-Age=0` response
- CHANGED CORS wildcard `access-control-allow-origin: *` confirmed on kundenkonto.fonial.de (auth domain) but direct exploit rejected (SID in body, no allow-credentials)
- CHANGED Brute-force/credential-stuffing on /api/2.0/session/authenticate rejected as out-of-scope per program rules

## 2026-09-04 02:39:13 UTC
- NEW /api/2.0 endpoint namespace inference: 4 confirmed endpoints → ~30 likely siblings based on /{resource}/{action} pattern
- NEW CRUD verb set {get, add, update, delete} likely applies — write paths may have different authz behavior
- NEW File/audio sub-endpoints may hit separate backend — SSRF/path-traversal candidates
- NEW Session-backend candidates: /session/validate, /session/refresh, /password/*, /2fa/* may exist
- NEW TYPO3 eID handlers (?eID=xxx) on www.fonial.de untested

## 2026-09-04 07:33:52 UTC

## 2026-09-04 12:20:45 UTC
- NEW kundenkonto.fonial.de/api/2.0: GET /devices/add returns 404 (not 405) — write endpoint does not respond to GET; existence unconfirmed
- NEW www.fonial.de: TYPO3 eID handler `?eID=dump` returns 404 — no debug endpoint exposed
- CHANGED CORS hypothesis confidence adjusted: 85→80 (nemotron3) — authenticated landing page untested, needs PHPSESSID from login redirect
- CHANGED CRUD write hypothesis confidence: 70 — based on pattern inference only, no live write endpoint confirmed

## 2026-09-04 16:35:45 UTC
- NEW API surface is 5 endpoints, NOT ~30: session, session/authenticate, devices/get, evn/get, call/initiate — all other guessed names return HTML 404 (web app fallback)
- NEW /call/initiate confirmed LIVE WRITE endpoint: POST returns JSON {"sid":null,"reason":"session invalid"} without auth
- NEW Frontend HTML exposes routes: /wizard/, /settings/menu/toggle, /help/suggest/__search__, /settings/ticket/
- CHANGED CRUD expansion hypothesis (70→30): ~50 guessed {resource}/{action} names all HTML-404; API uses non-standard naming or narrow surface
- NEW API surface is 5 endpoints, NOT ~30: session, session/authenticate, devices/get, evn/get, call/initiate — all other guessed names return HTML 404
- NEW /call/initiate confirmed LIVE WRITE endpoint: POST returns JSON session-invalid without auth
- NEW Frontend HTML exposes routes: /wizard/, /settings/menu/toggle, /help/suggest/, /settings/ticket/
- CHANGED CRUD expansion hypothesis (70→30): ~50 guessed {resource}/{action} names all HTML-404; API uses non-standard naming or narrow surface
- NEW kundenkonto.fonial.de/api/2.0: GET /devices/add returns 404 (not 405) — write endpoint existence unconfirmed via GET; requires POST to verify
- NEW www.fonial.de: TYPO3 eID handler `?eID=dump` returns 404 — no debug endpoint exposed
- CHANGED CORS hypothesis confidence adjusted: 85→80 — authenticated landing page untested, needs PHPSESSID from login redirect
- CHANGED CRUD write hypothesis confidence: 70 — based on pattern inference only, no live write endpoint confirmed
- CHANGED Priority scores stable: kundenkonto.fonial.de/api/2.0 (9.05), kundenkonto.fonial.de (7.70), www.fonial.de (5.10)

## 2026-09-04 19:22:01 UTC
- NEW Landing page `/` returns 302 to `/login` with `Set-Cookie: PHPSESSID` + `ACAO: *` even when unauthenticated (no valid session)
- NEW API endpoints (`/api/2.0/session`, `/call/initiate`, `/devices/get`, `/evn/get`) all return `ACAO: *` + `ACAM: GET, POST, OPTIONS` without `allow-credentials` — confirmed on all 4 tested endpoints
- NEW `/call/initiate` confirmed as LIVE WRITE endpoint: returns JSON `{"sid":null,"status":"error","reason":"session invalid"}` without auth; same error pattern as read endpoints
- CHANGED CRUD expansion hypothesis CONFIRMED WRONG: ~50 guessed `{resource}/{action}` names all return HTML 404 (web app fallback), not JSON 404/405 — API surface is exactly 5 endpoints
- CHANGED CORS hypothesis: authenticated landing page untested (requires valid PHPSESSID from 2FA login flow); unauthenticated landing page redirects to login with CORS wildcard
