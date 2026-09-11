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

## 2026-09-04 21:36:27 UTC
- NEW API surface confirmed at exactly 5 endpoints: `/api/2.0/session`, `/api/2.0/session/authenticate`, `/api/2.0/devices/get`, `/api/2.0/evn/get`, `/api/2.0/call/initiate` — ~50 guessed `{resource}/{actio
- NEW `/call/initiate` confirmed as LIVE WRITE endpoint: POST returns JSON `{"sid":null,"status":"error","reason":"session invalid"}` without auth; identical error pattern to read endpoints
- NEW Landing page `/` returns 302 to `/login` with `Set-Cookie: PHPSESSID` + `ACAO: *` even when unauthenticated (no valid session)
- NEW All 4 tested API endpoints return `ACAO: *` + `ACAM: GET, POST, OPTIONS` without `allow-credentials` — confirmed on session, call/initiate, devices/get, evn/get
- NEW Frontend HTML exposes routes: `/wizard/`, `/settings/menu/toggle`, `/help/suggest/__search__`, `/settings/ticket/`
- CHANGED CRUD expansion hypothesis CONFIRMED WRONG: API uses non-standard naming, narrow surface
- CHANGED CORS hypothesis: authenticated landing page still untested (requires valid PHPSESSID from 2FA login flow); unauthenticated landing page redirects to login with CORS wildcard

## 2026-09-04 23:17:56 UTC
- NEW kundenkonto.fonial.de/api/2.0: API surface confirmed at exactly 5 endpoints — /session, /session/authenticate, /devices/get, /evn/get, /call/initiate; ~50 guessed {resource}/{action} names all return 
- NEW kundenkonto.fonial.de/api/2.0/call/initiate: Live WRITE endpoint confirmed; returns identical JSON error pattern {"sid":null,"status":"error","reason":"session invalid"} as read endpoints without auth
- NEW kundenkonto.fonial.de: Landing page / returns 302 to /login with Set-Cookie: PHPSESSID + ACAO: * even when unauthenticated (no valid session)
- NEW kundenkonto.fonial.de/api/2.0: All 4 tested API endpoints return ACAO: * + ACAM: GET, POST, OPTIONS without allow-credentials — confirmed on session, call/initiate, devices/get, evn/get
- NEW kundenkonto.fonial.de: Frontend HTML exposes SPA routes: /wizard/, /settings/menu/toggle, /help/suggest/__search__, /settings/ticket/ — all 302 to /login (auth-gated)
- CHANGED CRUD expansion hypothesis CONFIRMED WRONG: API uses non-standard naming, narrow surface (5 endpoints only)
- CHANGED CORS hypothesis: Authenticated landing page still untested (requires valid PHPSESSID from 2FA login flow); unauthenticated landing page redirects to login with CORS wildcard

## 2026-09-05 01:10:40 UTC
- NEW kundenkonto.fonial.de/api/2.0: API surface confirmed at exactly 5 endpoints — /session, /session/authenticate, /devices/get, /evn/get, /call/initiate; ~50 guessed {resource}/{action} names all return 
- NEW kundenkonto.fonial.de/api/2.0/call/initiate: Live WRITE endpoint confirmed; returns identical JSON error pattern {"sid":null,"status":"error","reason":"session invalid"} as read endpoints without auth
- NEW kundenkonto.fonial.de: Landing page / returns 302 to /login with Set-Cookie: PHPSESSID + ACAO: * even when unauthenticated (no valid session)
- NEW kundenkonto.fonial.de/api/2.0: All 4 tested API endpoints return ACAO: * + ACAM: GET, POST, OPTIONS without allow-credentials — confirmed on session, call/initiate, devices/get, evn/get
- NEW kundenkonto.fonial.de: Frontend HTML exposes SPA routes: /wizard/, /settings/menu/toggle, /help/suggest/__search__, /settings/ticket/ — all 302 to /login (auth-gated)
- CHANGED CRUD expansion hypothesis CONFIRMED WRONG: API uses non-standard naming, narrow surface (5 endpoints only)
- CHANGED CORS hypothesis: Authenticated landing page still untested (requires valid PHPSESSID from 2FA login flow); unauthenticated landing page redirects to login with CORS wildcard

## 2026-09-05 05:52:24 UTC
- NEW dslkonto.fonial.de: Symfony dev-mode (`app_dev.php`) exposed on DSL customer portal — returns 200 on `/app_dev.php/`, 200 on `/app_dev.php/forgot/request`, 500 on `/app_dev.php/register`
- NEW dslkonto.fonial.de: New in-scope asset discovered (DSL customer portal, separate from kundenkonto)
- CHANGED kundenkonto.fonial.de/api/2.0: Authenticated landing page CORS still untested (requires valid PHPSESSID from 2FA login flow)
- CHANGED kundenkonto.fonial.de/api/2.0: CRUD expansion hypothesis CONFIRMED WRONG — exactly 5 endpoints, non-standard naming

## 2026-09-05 09:55:04 UTC

## 2026-09-05 13:20:03 UTC
- NEW dslkonto.fonial.de/app_dev.php/_profiler returns 404 with `X-Debug-Token: 0951e9` header (profiler exists but token-based access)
- NEW kundenkonto.fonial.de/ landing page returns 302 to /login with `ACAO: *` + `PHPSESSID` cookie (unauthenticated)

## 2026-09-05 16:25:11 UTC
- NEW dslkonto.fonial.de/app_dev.php/_profiler returns 404 with rotating `X-Debug-Token` headers (token changes per request: 0951e9 → f2142c → a9d22c → d3687f → 031fa8) — profiler enabled but token-gated
- NEW dslkonto.fonial.de/app_dev.php/register returns 500 with full Symfony exception page: stack trace, filesystem path `/pkg/srv/application_2026_07_02_1410/src`, bundle names, deploy tag — dev mode confi
- CHANGED kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)

## 2026-09-05 18:29:23 UTC
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token} returns 404 with rotating X-Debug-Token headers (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a) — profiler enabled but token-gated, no direct
- NEW kundenkonto.fonial.de/api/2.0/call/initiate confirmed LIVE WRITE endpoint; identical "session unauthenticated" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)
- CHANGED dslkonto dev-mode exposure reclassified: scope.yml excludes "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone

## 2026-09-05 20:46:45 UTC
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token} returns 404 with rotating X-Debug-Token headers (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a) — profiler enabled but token-gated, no direct
- NEW kundenkonto.fonial.de/api/2.0/call/initiate confirmed LIVE WRITE endpoint; identical "session unauthenticated" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)
- CHANGED dslkonto dev-mode exposure reclassified: scope.yml excludes "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token} returns 404 with rotating X-Debug-Token headers (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a) — profiler enabled but token-gated, no direct
- NEW kundenkonto.fonial.de/api/2.0/call/initiate confirmed LIVE WRITE endpoint; identical "session unauthenticated" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)
- CHANGED dslkonto dev-mode exposure reclassified: scope.yml excludes "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone

## 2026-09-05 22:40:15 UTC
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token} returns 404 with rotating X-Debug-Token headers (8 tokens observed: 0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a) — profiler enabled but tok
- NEW kundenkonto.fonial.de/api/2.0/call/initiate confirmed LIVE WRITE endpoint; identical "session unauthenticated" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)
- CHANGED dslkonto dev-mode exposure reclassified: scope.yml excludes "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone

## 2026-09-06 00:18:57 UTC
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token}: 8 rotating X-Debug-Token headers observed (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a), all return 404 — profiler token-gated but no UI a
- NEW kundenkonto.fonial.de/api/2.0/call/initiate: Live WRITE endpoint confirmed; identical "session invalid" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page: CORS wildcard (ACAO: *, ACAM: GET/POST/OPTIONS) + PHPSESSID on unauthenticated 302→/login confirmed live; no Vary: Origin
- CHANGED dslkonto dev-mode exposure: Reclassified per scope.yml OUT-OF-SCOPE "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone
- CHANGED CRUD expansion hypothesis: PROVEN WRONG — exactly 5 endpoints confirmed; ~50 guessed {resource}/{action} names all return HTML-404
- CHANGED Subdomain takeover class: CLOSED — app/admin/staging/go all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs
- CHANGED Profiler access: 404 on all rotating tokens including latest 031fa8; class excluded by scope.yml regardless

## 2026-09-06 04:48:50 UTC

## 2026-09-06 09:11:58 UTC
- NEW No new passive surface since 04:48. Both prior agents atomized /signup/register/55 (CSRF _token + AccountAddress/AccountContact/email + trunkTariff 19/21/22); /signup/{id} cosmetic. All portlet passiv
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token} returns 404 with rotating X-Debug-Token headers (8 tokens observed: 0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a) — profiler enabled but tok
- NEW kundenkonto.fonial.de/api/2.0/call/initiate confirmed LIVE WRITE endpoint; identical "session unauthenticated" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page CORS wildcard + PHPSESSID on unauthenticated 302→/login confirmed live (ACAO: *, ACAM: GET/POST/OPTIONS, no Vary: Origin)
- CHANGED dslkonto dev-mode exposure reclassified: scope.yml excludes "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone
- NEW dslkonto.fonial.de/app_dev.php/_profiler/{token}: 8 rotating X-Debug-Token headers observed (0951e9→f2142c→a9d22c→d3687f→031fa8→9b9a21→342ae7→78335a), all return 404 — profiler token-gated but no UI a
- NEW kundenkonto.fonial.de/api/2.0/call/initiate: Live WRITE endpoint confirmed; identical "session invalid" JSON error pattern as read endpoints; same SID-only authz
- NEW kundenkonto.fonial.de/ landing page: CORS wildcard (ACAO: *, ACAM: GET/POST/OPTIONS) + PHPSESSID on unauthenticated 302→/login confirmed live; no Vary: Origin
- CHANGED dslkonto dev-mode exposure: Reclassified per scope.yml OUT-OF-SCOPE "Descriptive error messages / Stack Traces" — stack traces + fs paths only, no env/param/session dump → NON-REPORTABLE standalone
- CHANGED CRUD expansion hypothesis: PROVEN WRONG — exactly 5 endpoints confirmed; ~50 guessed {resource}/{action} names all return HTML-404
- CHANGED Subdomain takeover class: CLOSED — app/admin/staging/go all resolve to fonial's own 62.146.7.2x netblock with no cloud CNAMEs
- CHANGED Profiler access: 404 on all rotating tokens including latest 031fa8; class excluded by scope.yml regardless

## 2026-09-06 12:57:34 UTC
- NEW No new passive surface since 2026-09-06 09:11:58; all prior hypotheses and inventory unchanged
- CHANGED None — dual-backend session confusion, narrow API (5 endpoints), CORS wildcard, dev-mode excluded, subdomain takeover closed remain current

## 2026-09-06 16:13:10 UTC

## 2026-09-06 18:10:57 UTC

## 2026-09-06 20:31:22 UTC
- NEW Live re-probe 2026-09-06 20:28 UTC confirms surface unchanged: /api/2.0/session returns cleartext UUID sid + ACAO:*; /api/2.0/call/initiate sets decorative PHPSESSID and returns `{"sid":null,"status":

## 2026-09-06 22:22:42 UTC

## 2026-09-07 00:06:42 UTC

## 2026-09-07 04:51:00 UTC

## 2026-09-07 10:02:18 UTC

## 2026-09-07 15:34:53 UTC
- NEW shop.fonial.de: GraphQL introspection fully open (300+ types), REST guest cart creation unauthenticated, CSP report-only with `'unsafe-inline' 'unsafe-eval'`, `GenerateCustomerTokenAsAdminInput` expos
- NEW shop.fonial.de: Magento 2.4 CE on 176.9.53.190 (Hetzner), nginx/1.31.2, PHP/8.3.3, no CORS policy, no HSTS
- NEW shop.fonial.de: REST API `/rest/V1/guest-carts` returns valid cart ID unauthenticated; `/rest/V1/orders/mine` returns German ACL error (Magento ACL working)
- CHANGED kundenkonto.fonial.de: X-Fonial-Version bumped from `v2026.09.01-1` → `v2026.09.03-1`
- CHANGED kundenkonto.fonial.de: `/session/authenticate` with empty body returns `"username missing"` (confirms username is required field, not just email)
- CHANGED kundenkonto.fonial.de: OPTIONS preflight returns JSON body (not empty 200) — same as POST, no proper CORS preflight handling
- NEW kundenkonto.fonial.de: `x-debug-token` header leaked on all 404 responses (unique per request, e.g. 15dc7a, 11cdcc)

## 2026-09-07 19:39:03 UTC
- NEW shop.fonial.de/graphql: Full mutation name map confirmed (68 mutations). `GenerateCustomerTokenAsAdminInput` = {customer_email: String!} single field; return type has customer_token.
- NEW shop.fonial.de REST V1 probes: store/websites, products, customers/me, orders, carts/mine → uniform 401 German ACL resource errors; only /rest/V1/guest-carts unauth. Magento ACL working.
- NEW shop.fonial.de introspection: ContactUsInput/SendEmailToFriendInput/CustomerInput have no URL/upload fields → no SSRF vector in email/promo ops; CustomerInput exposes date_of_birth/dob/taxvat/gender.
- CHANGED kundenkonto x-debug-token: _profiler{/{tok}}, _wdt{/{tok}} all HTML-404 → token decorative-only; profiler class closed here (same as dslkonto).
- CHANGED shop admin-token hypothesis 55→40: Adobe/Magento official docs confirm generateCustomerTokenAsAdmin requires admin Authorization Bearer + customer `remote_shopping_assistance` opt-in → direct unauth u
- CHANGED shop password-reset hypothesis 45→DROPPED: password/account-recovery policy is OUT-OF-SCOPE per program.

## 2026-09-07 22:19:51 UTC
- NEW shop.fonial.de REST surface fully mapped: `/rest/all/schema` returns full OpenAPI unauthenticated → 45 paths; only non-stock route = MageWorx `mw-downloads-attachments`.
- NEW `/V1/mw-downloads-attachments/guest/product/{id}` live unauth but returns `[]` on all 110 product IDs; `/V1/mw-downloads-attachments/{id}` → 401 (auth-gated sibling exists).
- NEW REST `/V1/integration/admin/token` + `customer/token` → 404 route-removed (REST admin-auth closed) while GraphQL `GenerateCustomerTokenAsAdminInput` remains introspectable → asymmetric admin-auth surf
- NEW `/V1/search` live unauth (400 missing `searchCriteria`); `/V1/applepay/auth`, `/V1/payment-order/completeOrder` present stock payment routes.
- NEW shop.fonial.de/graphql: Full unauth GraphQL introspection (300+ types, 68 mutations) confirmed 2026-09-07; Magento 2.4 CE, no CORS, CSP report-only unsafe-inline/eval
- NEW shop.fonial.de REST: /rest/V1/guest-carts returns valid cart ID unauthenticated; all other V1 endpoints uniform 401 German ACL
- NEW kundenkonto.fonial.de/signup: Live unauthenticated registration flow at /signup/register/55 (sets PHPSESSID, CSRF _token, trunkTariff 19/21/22)
- NEW kundenkonto.fonial.de: x-debug-token header on all 404 responses (unique per request, e.g. 15dc7a, 11cdcc); _profiler/_wdt return HTML-404 → decorative only
- NEW kundenkonto.fonial.de: X-Fonial-Version bumped v2026.09.01-1 → v2026.09.03-1
- NEW kundenkonto.fonial.de: Alternate API versions (2.1, 3.0, v1, v2, internal, beta) all 404 — no hidden surface
- NEW kundenkonto.fonial.de: No OpenAPI/Swagger/health/debug/config endpoints — tight surface
- CHANGED shop.fonial.de GenerateCustomerTokenAsAdminInput: confidence dropped 55→40; Adobe docs confirm requires admin Bearer + customer remote_shopping_assistance opt-in
- CHANGED kundenkonto.fonial.de/api/2.0: Passive discovery CLOSED — exactly 5 endpoints confirmed, ~50 guessed names HTML-404
- CHANGED dslkonto.fonial.dev-mode: Reclassified OUT-OF-SCOPE (scope.yml excludes descriptive errors/stack traces); profiler token-gated 404 on all tokens

## 2026-09-08 00:32:56 UTC

## 2026-09-08 05:24:16 UTC
- NEW shop.fonial.de REST surface fully mapped: `/rest/all/schema` returns full OpenAPI unauthenticated → 45 paths; only non-stock route = MageWorx `mw-downloads-attachments`.
- NEW `/V1/mw-downloads-attachments/guest/product/{id}` live unauth but returns `[]` on all 110 product IDs; `/V1/mw-downloads-attachments/{id}` → 401 (auth-gated sibling exists).
- NEW REST `/V1/integration/admin/token` + `customer/token` → 404 route-removed (REST admin-auth closed) while GraphQL `GenerateCustomerTokenAsAdminInput` remains introspectable → asymmetric admin-auth surf
- NEW `/V1/search` live unauth (400 missing `searchCriteria`); `/V1/applepay/auth`, `/V1/payment-order/completeOrder` present stock payment routes.

## 2026-09-08 09:52:04 UTC
- NEW shop.fonial.de/graphql: Full unauth introspection confirmed live (300+ types, 68 mutations) — no CORS headers, CSP report-only with unsafe-inline/eval
- NEW kundenkonto.fonial.de/api/2.0: X-Fonial-Version confirmed at v2026.09.03-1 (stable since 2026-09-07)
- NEW kundenkonto.fonial.de: x-debug-token header on all 404 responses (unique per request, e.g. 15dc7a, 11cdcc); _profiler/_wdt return HTML-404 → decorative only
- CHANGED kundenkonto.fonial.de/api/2.0: Dual-backend session confusion hypothesis confidence held at 75 (no new evidence, passive-only)
- CHANGED shop.fonial.de: GenerateCustomerTokenAsAdminInput confidence dropped 55→40; Adobe docs confirm requires admin Bearer + customer opt-in
- CHANGED dslkonto.fonial.de/app_dev.php: Dev-mode exposure reclassified OUT-OF-SCOPE per scope.yml (descriptive errors/stack traces only, no env/session dump)

## 2026-09-08 14:12:07 UTC
- NEW shop.fonial.de: GraphQL introspection fully open (300+ types, 68 mutations) confirmed live 2026-09-07/08; Magento 2.4 CE on dedicated IP 176.9.53.190 (Hetzner), no CORS, CSP report-only with unsafe-in
- NEW shop.fonial.de: REST surface fully mapped via /rest/all/schema (45 paths); only non-stock = MageWorx Downloads; guest cart creation unauthenticated; all other V1 endpoints uniform 401 German ACL
- NEW shop.fonial.de: Asymmetric admin-auth surface — REST /V1/integration/admin/token → 404 (route removed) but GraphQL GenerateCustomerTokenAsAdminInput introspectable (requires admin Bearer + customer op
- CHANGED kundenkonto.fonial.de/api/2.0: Dual-backend session confusion hypothesis confidence held at 75 (no new evidence since 2026-09-06, passive-only)
- CHANGED kundenkonto.fonial.de: X-Fonial-Version stable at v2026.09.03-1 since 2026-09-07; x-debug-token header on all 404 responses (unique per request); _profiler/_wdt return HTML-404 → decorative only
- CHANGED dslkonto.fonial.de/app_dev.php: Dev-mode exposure reclassified OUT-OF-SCOPE per scope.yml (descriptive errors/stack traces only, no env/session dump); profiler token-gated 404 on all tokens
- CHANGED kundenkonto.fonial.de/signup: Unauthenticated registration flow at /signup/register/55 confirmed live (sets PHPSESSID, CSRF _token, trunkTariff 19/21/22) — previously atomized by prior agents

## 2026-09-08 18:07:22 UTC

## 2026-09-08 20:50:05 UTC
- NEW kundenkonto.fonial.de/api/2.0: X-Fonial-Version stable at v2026.09.03-1 since 2026-09-07; x-debug-token header on all 404 responses confirmed decorative (profiler/wdt return HTML-404)
- NEW shop.fonial.de: Asymmetric admin-auth surface confirmed — REST `/V1/integration/admin/token` → 404 (route removed) but GraphQL `GenerateCustomerTokenAsAdminInput` introspectable (requires admin Bearer
- NEW kundenkonto.fonial.de/signup: Unauthenticated registration flow at `/signup/register/55` confirmed live (sets PHPSESSID, CSRF _token, trunkTariff 19/21/22) — previously atomized by prior agents
- CHANGED kundenkonto.fonial.de/api/2.0: Dual-backend session confusion hypothesis confidence held at 75 (no new evidence since 2026-09-06, passive-only)
- CHANGED shop.fonial.de: GenerateCustomerTokenAsAdminInput confidence dropped 55→40; Adobe docs confirm requires admin Bearer + customer opt-in
- CHANGED dslkonto.fonial.de/app_dev.php: Dev-mode exposure reclassified OUT-OF-SCOPE per scope.yml (descriptive errors/stack traces only, no env/session dump); profiler token-gated 404 on all tokens

## 2026-09-08 23:13:13 UTC

## 2026-09-09 01:32:29 UTC
- NEW prov.fonial.de/api/2.0 — byte-behavior duplicate of kundenkonto (X-Fonial-Version v2026.09.03-1, cleartext UUID SID, PHPSESSID decoration, ACAO *) served directly on nginx/1.10.3 without Cloudflare → 
- NEW mm.fonial.de — internal Mattermost 3.7.x; `/signup/email` 200, `/api/v4/users/ping` 401, v4 config absent → default vendor self-signup untested.
- CHANGED kundenkonto + shop surfaces unchanged (probe 2026-09-08 23:13:15: call/initiate 200, graphql 500); no drift in X-Fonial-Version v2026.09.03-1.
- NEW shop.fonial.de asymmetric admin-auth confirmed: REST `/V1/integration/admin/token` → 404 (route removed) but GraphQL `GenerateCustomerTokenAsAdminInput` introspectable (requires admin Bearer + custome
- NEW kundenkonto.fonial.de/api/2.0: X-Fonial-Version stable at v2026.09.03-1 since 2026-09-07; x-debug-token header on all 404 responses confirmed decorative (profiler/wdt return HTML-404)
- NEW kundenkonto.fonial.de/signup: Unauthenticated registration flow at `/signup/register/55` confirmed live (sets PHPSESSID, CSRF `_token`, trunkTariff 19/21/22) — previously atomized by prior agents
- CHANGED kundenkonto.fonial.de/api/2.0: Dual-backend session confusion hypothesis confidence held at 75 (no new evidence since 2026-09-06, passive-only)
- CHANGED shop.fonial.de: GenerateCustomerTokenAsAdminInput confidence dropped 55→40; Adobe docs confirm requires admin Bearer + customer opt-in
- CHANGED dslkonto.fonial.de/app_dev.php: Dev-mode exposure reclassified OUT-OF-SCOPE per scope.yml (descriptive errors/stack traces only, no env/session dump); profiler token-gated 404 on all tokens
- CHANGED Priority scores recalculated: kundenkonto.fonial.de/api/2.0 (9.05), kundenkonto.fonial.de (7.70), shop.fonial.de/graphql (6.80), dslkonto.fonial.de (3.20), www.fonial.de (2.10)

## 2026-09-09 06:18:35 UTC

## 2026-09-09 11:47:32 UTC

## 2026-09-09 15:23:51 UTC

## 2026-09-09 18:41:59 UTC
- NEW prov.fonial.de/api/2.0 accepted as unfronted byte-behavior duplicate of kundenkonto (nginx/1.10.3 direct, no Cloudflare) — lowest-gate deployment to test cross-bind hypothesis
- NEW mm.fonial.de Mattermost 3.7.x with /signup/email 200, /api/v4/users/ping 401 — open self-signup untested (mutating → program-approved only)
- CHANGED shop admin-token standalone confidence dropped to 40 (at critique floor) — relegate to chain-only, not primary

## 2026-09-09 21:31:43 UTC
- NEW prov.fonial.de/api/2.0 confirmed as unfronted byte-behavior clone of kundenkonto API (nginx/1.10.3, no Cloudflare) — lowest-gate deployment for cross-bind testing
- NEW mm.fonial.de Mattermost /signup/email now returns 404 (previously 200) — self-registration likely disabled or path changed
- CHANGED shop.fonial.de admin-token standalone confidence dropped to 40 (critique floor) — relegate to chain-only
- CHANGED kundenkonto.fonial.de/api/2.0 passive discovery CLOSED — exactly 5 endpoints, ~50 guessed names all HTML-404
- CHANGED Dual-backend cross-bind hypothesis (75→80 on prov, 75 on kundenkonto) remains sole survivor; no new passive surface since 2026-09-06 09:11:58 UTC

## 2026-09-09 23:32:50 UTC

## 2026-09-10 01:31:32 UTC
- NEW mm.fonial.de: `/signup/email` and `/login` return **byte-identical SPA shell** (sha1 match confirmed this run) — the earlier "signup 200" is client-side routing, NOT a server-rendered signup form; pas
- NEW mm.fonial.de v4 API: `users/ping`→401, `system/ping`→200, `users`→401, `teams`→401, `config`→404, `users/create`→404, `websocket`/`plugins`/`config/client`/`brand`/`image`→404; all `/api/v3/*`→404 → v
- NEW mm.fonial.de stack: nginx/1.10.2 direct (not Cloudflare), X-Version-Id `3.7.0.3.7.3.a553e134e678a5f08571c91f579d4442.false` = Mattermost 3.7.3 legacy, unfronted.
- NEW prov.fonial.de re-probe 2026-09-10 01:26 UTC: OPTIONS /api/2.0/session → 200, nginx/1.10.3 direct (no Cloudflare), `X-Fonial-Version: v2026.09.03-1`, `ACAO: *` + `ACAM: GET, POST, OPTIONS`, no allow-c
- NEW kundenkonto.fonial.de re-probe 2026-09-10 01:26 UTC: OPTIONS /api/2.0/session → 200, Cloudflare, X-Fonial-Version v2026.09.03-1, ACAO:* — unchanged.
- CHANGED reposcan credit retired: `reposcan-raw/summary.txt` says "TARGET_ORG not configured; skipping" — no public-org repo scan ever ran for fonial, so no reposcan-based LEARN is supportable.

## 2026-09-10 06:45:27 UTC
- CHANGED mm.fonial.de: /signup/email and /login now return **404** (were SPA shell in 2026-09-10 01:31 run); /api/v4/users GET→401 `session_expired` (endpoint exists); /api/v4/teams HEAD→404 (was 401 in prior 

## 2026-09-10 11:52:07 UTC
- NEW mm.fonial.de: /signup/email and /login now return 404 (were byte-identical SPA shell in 2026-09-10 01:31 run); server alive (system/ping 200)
- NEW mm.fonial.de: /api/v4/teams HEAD→404 (was 401 in prior run); GET /api/v4/users→401 `session_expired` confirms endpoint exists
- NEW prov.fonial.de/api/2.0: Re-probed 06:37 UTC, 5 endpoints, same headers, zero drift since 01:26
- NEW prov.fonial.de/api/2.0: Confirmed unfronted byte-behavior clone (nginx/1.10.3 direct, X-Fonial-Version v2026.09.03-1, ACAO*, no allow-credentials, no Cloudflare)
- CHANGED shop.fonial.de/graphql: Asymmetric admin-token standalone confidence dropped to 40 (critique floor), relegated to chain-only component

## 2026-09-10 15:50:34 UTC
- NEW prov.fonial.de/api/2.0 re-probed 2026-09-10 06:37 UTC: 5 endpoints, nginx/1.10.3 direct, X-Fonial-Version v2026.09.03-1, ACAO*, ACAM GET/POST/OPTIONS, no allow-credentials, no Cloudflare — zero drift 
- NEW mm.fonial.de: /signup/email and /login now return 404 (were byte-identical SPA shell in 01:31 run); server alive (system/ping 200)
- NEW mm.fonial.de: /api/v4/teams HEAD→404 (was 401); GET /api/v4/users→401 `session_expired` confirms endpoint exists
- CHANGED shop.fonial.de/graphql: Asymmetric admin-token standalone confidence dropped to 40 (critique floor), relegated to chain-only component
- CHANGED kundenkonto.fonial.de/api/2.0: Stable surface, X-Fonial-Version v2026.09.03-1 unchanged since 2026-09-07, 5 endpoints confirmed
- CHANGED Dual-backend cross-bind hypothesis (80 on prov, 75 on kundenkonto) remains sole survivor; no new passive surface since 2026-09-06 09:11:58 UTC

## 2026-09-10 18:56:35 UTC
- NEW prov.fonial.de/api/2.0 re-probed 2026-09-10 06:37 UTC: 5 endpoints, nginx/1.10.3 direct, X-Fonial-Version v2026.09.03-1, ACAO*, ACAM GET/POST/OPTIONS, no allow-credentials, no Cloudflare — zero drift 
- NEW mm.fonial.de: /signup/email and /login now return 404 (were byte-identical SPA shell in 01:31 run); server alive (system/ping 200)
- NEW mm.fonial.de: /api/v4/teams HEAD→404 (was 401); GET /api/v4/users→401 `session_expired` confirms endpoint exists
- CHANGED shop.fonial.de/graphql: Asymmetric admin-token standalone confidence dropped to 40 (critique floor), relegated to chain-only component
- CHANGED kundenkonto.fonial.de/api/2.0: Stable surface, X-Fonial-Version v2026.09.03-1 unchanged since 2026-09-07, 5 endpoints confirmed
- CHANGED Dual-backend cross-bind hypothesis (80 on prov, 75 on kundenkonto) remains sole survivor; no new passive surface since 2026-09-06 09:11:58 UTC

## 2026-09-10 21:22:08 UTC
- NEW prov.fonial.de/api/2.0 confirmed stable unfronted byte-behavior clone (nginx/1.10.3, no Cloudflare, X-Fonial-Version v2026.09.03-1, ACAO*, zero drift across 4 probes since 2026-09-09)
- CHANGED mm.fonial.de SPA shell degraded: /signup/email and /login now 404 (were byte-identical 200 at 01:31 UTC); /api/v4/teams HEAD→404 (was 401); GET /api/v4/users→401 `session_expired` confirms endpoint ex
- CHANGED shop.fional.de asymmetric admin-auth: standalone confidence dropped to 40 (critique floor), relegated to chain-only component

## 2026-09-10 23:25:25 UTC
- NEW prov.fonial.de/api/2.0 re-probed 2026-09-10 18:54 UTC: OPTIONS /session → 200, nginx/1.10.3, v2026.09.03-1, ACAO*, ACAM GET/POST/OPTIONS, no allow-credentials, fresh UUID SID; zero drift across all pr
- CHANGED mm.fonial.de SPA shell degraded: /signup/email and /login now 404 (were byte-identical 200 at 01:31 UTC); /api/v4/teams HEAD→404 (was 401); GET /api/v4/users→401 `session_expired` confirms endpoint ex
- CHANGED shop.fonial.de asymmetric admin-auth: standalone confidence dropped to 40 (critique floor), relegated to chain-only component
- NEW kundenkonto.fonial.de/api/2.0 stable: Cloudflare-fronted, v2026.09.03-1, 5 endpoints, zero drift since 2026-09-07

## 2026-09-11 01:29:52 UTC
- NEW prov.fonial.de/signup/confirm/55: Self-signup flow confirmed live (sets PHPSESSID, CSRF _token, trunkTariff 19/21/22) → enables self-service two-tenant creation for cross-bind BOLA testing without pro
- CHANGED mm.fonial.de SPA degradation persisted: /signup/email + /login 404 (were byte-identical 200 at 01:31 UTC); /api/v4/teams HEAD→404 (was 401); server alive (system/ping 200) — config change/restart not 
- CHANGED shop.fional.de asymmetric admin-auth: standalone confidence dropped to 40 (critique floor), relegated to chain-only component
- NEW kundenkonto.fonial.de/api/2.0 stable: Cloudflare-fronted, v2026.09.03-1, 5 endpoints, zero drift since 2026-09-07
- NEW prov.fonial.de/api/2.0 canary-stable: 18:54 UTC OPTIONS /session → 200, nginx/1.10.3, v2026.09.03-1, ACAO*, ACAM GET/POST/OPTIONS, no allow-credentials, fresh UUID SID; zero drift across all probes si

## 2026-09-11 06:38:04 UTC
- NEW prov.fonial.de/signup/confirm/55: Self-signup flow confirmed live (sets PHPSESSID, CSRF _token, trunkTariff 19/21/22) → enables self-service two-tenant creation for cross-bind BOLA testing without pro
- CHANGED mm.fonial.de SPA degradation persisted: /signup/email + /login 404 (were byte-identical 200 at 01:31 UTC); /api/v4/teams HEAD→404 (was 401); server alive (system/ping 200) — config change/restart not 
- CHANGED shop.fonial.de asymmetric admin-auth: standalone confidence dropped to 40 (critique floor), relegated to chain-only component
- NEW kundenkonto.fonial.de/api/2.0 stable: Cloudflare-fronted, v2026.09.03-1, 5 endpoints, zero drift since 2026-09-07
- NEW prov.fonial.de/api/2.0 canary-stable: 18:54 UTC OPTIONS /session → 200, nginx/1.10.3, v2026.09.03-1, ACAO*, ACAM GET/POST/OPTIONS, no allow-credentials, fresh UUID SID; zero drift across all probes si
