# Knowledge Base (seed)
- 2026-09-03 REJECTED SSRF @ www.fonial.de: No URL parameters or webhook endpoints found in passive recon
- 2026-09-03 REJECTED IDOR @ www.fonial.de: Pure marketing site, no object references
- 2026-09-03 ACCEPTED MISCONFIG @ kundenkonto.fonial.de: Wildcard CORS with credentials on auth-enabled domain confirmed
- 2026-09-03 REJECTED CORS wildcard direct-exploit @ kundenkonto.fonial.de/api/2.0: SID in body (not cookies), no `allow-credentials`, browser won't send cookies cross-origin → low direct impact without SID leak chain.
- 2026-09-03 ACCEPTED dual-backend architecture @ kundenkonto.fonial.de/api/2.0: Two distinct servers (session vs data) with different response headers and session mechanisms → session confusion attack surface.
