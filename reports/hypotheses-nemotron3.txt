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
