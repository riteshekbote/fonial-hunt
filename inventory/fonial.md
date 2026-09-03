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
