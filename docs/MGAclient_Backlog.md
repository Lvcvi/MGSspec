# MGA Compliant Game Client: Ticketized Backlog

Version: 1.0  
Date: 2025-09-09  
Status: Planned  
Scope: Frontend game client and cross-cutting compliance/governance items required for MGA alignment.

Legend:  
- SP: Story points (1,2,3,5,8,13).  
- Labels: [frontend], [backend], [compliance], [security], [ws], [pci], [ops], [observability], [infra].  
- Refs: Inline references to requirements in `docs/MGAclient_OV.md` and `docs/MGAclient_arch.md`.

Index of Epics:
- GOV: Governance & MGA Submission
- APP: App Shell & Session
- NET: Networking & State
- CMP: Compliance UI
- ENG: Game Engine Integration
- PAY: Payments Integration
- OBS: Observability & Telemetry
- SEC: Hardening & Certification
- OPS: CI/CD & Release Governance
- INF: Client Hosting & Edge Controls

---

## Epic GOV: Governance & MGA Submission

Objective: Establish certified hosting, third-party engagements, and documentation required for MGA submission and audits.  
Refs: `docs/MGAclient_OV.md:48`, `docs/MGAclient_OV.md:57`, `docs/MGAclient_OV.md:208`, `docs/MGAclient_OV.md:228`, `docs/MGAclient_OV.md:192`, `docs/MGAclient_OV.md:199`  
Labels: [compliance], [infra], [ops]

- MGA-GOV-01 — Select ISO 27001:2013 EEA datacenter
  - Desc: Evaluate and contract an ISO 27001 certified facility in the EEA.
  - AC: Contract signed; certification verified; IP ranges documented.
  - Deps: None  
  - SP: 5  
  - Refs: `docs/MGAclient_OV.md:48`

- MGA-GOV-02 — Procure Malta replication server provider
  - Desc: Contract a Malta-based provider and define secure connectivity.
  - AC: SLA signed; network plan approved; security controls documented.
  - Deps: MGA-GOV-01  
  - SP: 5  
  - Refs: `docs/MGAclient_OV.md:57`

- MGA-GOV-03 — Engage MGA-approved testing/audit labs
  - Desc: Select labs for RNG/game certification and pen tests.
  - AC: Engagement letters signed; test scopes and timelines agreed.
  - Deps: None  
  - SP: 3  
  - Refs: `docs/MGAclient_OV.md:192`, `docs/MGAclient_OV.md:199`

- MGA-GOV-04 — System & Application architecture packs
  - Desc: Author network topology, hosting map, and sequence diagrams (client ↔ RGS).
  - AC: Diagram set approved by architecture board; version-controlled.
  - Deps: MGA-GOV-01, MGA-GOV-02  
  - SP: 8  
  - Refs: `docs/MGAclient_OV.md:208`

- MGA-GOV-05 — Data replication plan (RPO/RTO, failover)
  - Desc: Define real-time replication mechanics, validation, and drills.
  - AC: Runbook exists; test drill completed; results logged.
  - Deps: MGA-GOV-02, MGA-GOV-04  
  - SP: 5  
  - Refs: `docs/MGAclient_OV.md:57`, `docs/MGAclient_OV.md:208`

- MGA-GOV-06 — Security policies (ISMS, change mgmt, IRP)
  - Desc: Publish InfoSec policy, change management, and incident response plans.
  - AC: Policies approved; linked in submission; version-controlled.
  - Deps: None  
  - SP: 5  
  - Refs: `docs/MGAclient_OV.md:208`, `docs/MGAclient_OV.md:252`

- MGA-GOV-07 — MGA submission pack skeleton
  - Desc: Create submission structure and checklist coverage.
  - AC: All required sections stubbed; owner assigned per section.
  - Deps: MGA-GOV-04, MGA-GOV-06  
  - SP: 3

---

## Epic APP: App Shell & Session

Objective: Provide the SPA shell, routing, layout, inactivity timer, and auth/logout flow.  
Refs: `docs/MGAclient_arch.md:28`, `docs/MGAclient_OV.md:144`  
Labels: [frontend], [compliance]

- MGA-APP-01 — Scaffold Vite + React + TS workspace
  - AC: App builds/runs; ESLint/Prettier configured; Vitest/Playwright set up.
  - Deps: None  
  - SP: 3

- MGA-APP-02 — App layout and router
  - AC: MainLayout with header/footer; routes for login, lobby, game, settings.
  - Deps: MGA-APP-01  
  - SP: 3

- MGA-APP-03 — Inactivity timer (28m warn, 30m logout)
  - AC: User input resets timer; 28m modal warns; 30m triggers logout + state clear.
  - Deps: MGA-APP-02  
  - SP: 5  
  - Refs: `docs/MGAclient_arch.md:28`

- MGA-APP-04 — Global suspension overlay
  - AC: Overlay blocks input when `isGameSuspended` or `isConnectionLost` true.
  - Deps: MGA-APP-02, MGA-NET-05  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:28`

- MGA-APP-05 — OIDC login flow and token storage
  - AC: Successful login retrieves JWT; secure storage rules documented; logout clears state.
  - Deps: MGA-APP-01  
  - SP: 5  
  - Refs: `docs/MGAclient_OV.md:39`, `docs/MGAclient_arch.md:41`

- MGA-APP-06 — Global 401 handler → logout
  - AC: Any 401 triggers central logout and redirect to login; tested e2e.
  - Deps: MGA-NET-01  
  - SP: 2  
  - Refs: `docs/MGAclient_arch.md:41`

- MGA-APP-07 — Feature flags and environment config loader
  - AC: Env-driven flags (e.g., payments enabled); no secrets in bundle.
  - Deps: MGA-APP-01  
  - SP: 3  
  - Refs: `docs/MGAclient_OV.md:252`

- MGA-APP-08 — Localization baseline (i18n)
  - AC: At least EN + one additional locale; modal strings externalized.
  - Deps: MGA-APP-02  
  - SP: 3

---

## Epic NET: Networking & State

Objective: Central ApiService (HTTPS + WSS), idempotency, reconnection, and StateService slices.  
Refs: `docs/MGAclient_arch.md:41`, `docs/MGAclient_arch.md:60`, `docs/MGAclient_OV.md:129`  
Labels: [frontend], [ws], [security]

- MGA-NET-01 — Axios instance with auth/error interceptors
  - AC: Authorization header added; 401 routed to global logout; 5xx surfaced.
  - Deps: MGA-APP-05  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:41`

- MGA-NET-02 — JWT refresh/rotation plumbing
  - AC: Short-lived access tokens; refresh path; rotation tested.
  - Deps: MGA-APP-05  
  - SP: 5

- MGA-NET-03 — WebSocket client with backoff + heartbeat
  - AC: Exponential backoff; ping/pong; network drop simulated in tests.
  - Deps: MGA-NET-01  
  - SP: 5  
  - Refs: `docs/MGAclient_arch.md:41`

- MGA-NET-04 — WSS auth handshake
  - AC: connect(token) authenticates; unauthenticated closes; errors handled.
  - Deps: MGA-NET-03  
  - SP: 3  
  - Refs: `docs/MGAclient_OV.md:39`

- MGA-NET-05 — Reconnect → full state sync
  - AC: On reconnect, send `state.sync`; store updated with authoritative state; UI unlocks.
  - Deps: MGA-NET-04, MGA-STA-01  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:41`

- MGA-NET-06 — Idempotent bet sending with UUID v4
  - AC: `transactionId` always present for financial actions; duplicates deduped server-side.
  - Deps: MGA-NET-04  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:41`, `docs/MGAclient_OV.md:129`

- MGA-NET-07 — Error model + correlationId propagation
  - AC: Standard error envelope; correlationId attached to client logs.
  - Deps: MGA-OBS-01  
  - SP: 2

---

## Epic STA: StateService

Objective: Centralized store slices and selectors.  
Refs: `docs/MGAclient_arch.md:60`  
Labels: [frontend]

- MGA-STA-01 — Configure Redux Toolkit store
  - AC: Store configured with middleware; DevTools gated by env.
  - Deps: MGA-APP-01  
  - SP: 2

- MGA-STA-02 — Session slice
  - AC: Fields: isAuthenticated, token, user, sessionStartTime, isSessionExpired.
  - Deps: MGA-STA-01  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:60`

- MGA-STA-03 — Player slice
  - AC: Fields: balance, currency; selectors for formatted balance.
  - Deps: MGA-STA-01  
  - SP: 2

- MGA-STA-04 — Game slice
  - AC: Fields: gameId, isGameSuspended, isConnectionLost, currentRound.
  - Deps: MGA-STA-01  
  - SP: 3

- MGA-STA-05 — ResponsibleGaming slice
  - AC: limits, limitChangeCooldown, realityCheck, sessionStats.
  - Deps: MGA-STA-01  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:60`

---

## Epic CMP: Compliance UI

Objective: UI components mandated by compliance.  
Refs: `docs/MGAclient_OV.md:89`, `docs/MGAclient_arch.md:70`  
Labels: [frontend], [compliance]

- MGA-CMP-01 — ComplianceBar (balance + clock)
  - AC: Balance updates live from store; system clock ticks each second.
  - Deps: MGA-STA-03  
  - SP: 2

- MGA-CMP-02 — RealityCheck modal (blocking)
  - AC: Triggers via timer; blocks gameplay; shows total wagered and net win/loss; continue/exit actions.
  - Deps: MGA-STA-05, MGA-APP-04  
  - SP: 5  
  - Refs: `docs/MGAclient_arch.md:70`

- MGA-CMP-03 — Limits Manager (tighten/loosen + cooldown)
  - AC: Tighten immediate; loosen sets 24h cooldown; countdown displayed.
  - Deps: MGA-NET-01, MGA-STA-05  
  - SP: 5  
  - Refs: `docs/MGAclient_arch.md:70`

- MGA-CMP-04 — Self-Exclusion flow
  - AC: Confirmation UX; initiate call; server terminates session; 401 handler logs out.
  - Deps: MGA-NET-01, MGA-APP-06  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:70`

- MGA-CMP-05 — Suspension UX integration
  - AC: Reality checks and connection loss both set suspension; engine/input blocked.
  - Deps: MGA-APP-04, MGA-ENG-03  
  - SP: 3

- MGA-CMP-06 — Accessibility pass for compliance UI
  - AC: Modals trap focus; ARIA roles; keyboard navigation; contrast AA.
  - Deps: MGA-CMP-02, MGA-CMP-03  
  - SP: 3

---

## Epic ENG: Game Engine Integration

Objective: Rendering, input gating, and outcome visualization without determinative logic.  
Refs: `docs/MGAclient_arch.md:90`  
Labels: [frontend]

- MGA-ENG-01 — Choose engine and bootstrap (PixiJS/Phaser)
  - AC: Engine initialized; canvas mounted in GameView; hot reload OK.
  - Deps: MGA-APP-02  
  - SP: 3

- MGA-ENG-02 — Asset loader & preloading
  - AC: Preload pipeline; fallback error handling; progress UI.
  - Deps: MGA-ENG-01  
  - SP: 3

- MGA-ENG-03 — Frame loop gating on suspension/disconnect
  - AC: Each frame checks flags; pauses loop; input ignored while blocked.
  - Deps: MGA-STA-04, MGA-APP-04  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:90`

- MGA-ENG-04 — Input subsystem with gating
  - AC: Input disabled when suspended/disconnected; re-enabled on resume.
  - Deps: MGA-ENG-03  
  - SP: 3

- MGA-ENG-05 — Outcome visualization pipeline
  - AC: Consumes server outcomes; renders symbols/spins; no determinative logic.
  - Deps: MGA-NET-05, MGA-STA-04  
  - SP: 5

- MGA-ENG-06 — Deterministic animations from payload
  - AC: Animations seeded solely from server payload; reproducible in tests.
  - Deps: MGA-ENG-05  
  - SP: 5

- MGA-ENG-07 — Error boundary around renderer
  - AC: Renderer failures caught; user-friendly fallback; telemetry captured.
  - Deps: MGA-OBS-01  
  - SP: 2

---

## Epic PAY: Payments Integration

Objective: PSP iframe integration with strict PCI isolation.  
Refs: `docs/MGAclient_arch.md:98`, `docs/MGAclient_OV.md:176`  
Labels: [frontend], [pci], [security]

- MGA-PAY-01 — PSP iframe host and layout
  - AC: Secure iframe container; responsive; hidden unless in payment flow.
  - Deps: MGA-APP-02  
  - SP: 3

- MGA-PAY-02 — postMessage handshake and events
  - AC: Window message handling; origin allowlist; success/failure parsed.
  - Deps: MGA-PAY-01  
  - SP: 3

- MGA-PAY-03 — Payment start/complete UX
  - AC: Start action opens iframe; completion updates state and balance.
  - Deps: MGA-PAY-02, MGA-STA-03  
  - SP: 3

- MGA-PAY-04 — CSP allowlist for PSP domain
  - AC: frame-src includes PSP; tested e2e; documented in security policy.
  - Deps: MGA-SEC-01  
  - SP: 2  
  - Refs: `docs/MGAclient_OV.md:176`

- MGA-PAY-05 — Error/receipt surfacing
  - AC: Error paths handled; receipts displayed; retriable vs. non-retriable states.
  - Deps: MGA-PAY-03  
  - SP: 3

- MGA-PAY-06 — QA PSP sandbox wiring
  - AC: Test mode domain; credentials via env; demo flows pass.
  - Deps: MGA-PAY-02, MGA-OPS-01  
  - SP: 2

---

## Epic OBS: Observability & Telemetry

Objective: Client-side telemetry, error tracking, and correlationId propagation.  
Refs: `docs/MGAclient_OV.md:149`  
Labels: [observability], [frontend]

- MGA-OBS-01 — Client telemetry SDK
  - AC: Standard event API; buffer/flush; opt-in controls; PII redaction.
  - Deps: MGA-APP-01  
  - SP: 5

- MGA-OBS-02 — Error tracking integration
  - AC: Global error boundary; unhandled rejections; source maps uploaded.
  - Deps: MGA-OBS-01  
  - SP: 3

- MGA-OBS-03 — CorrelationId propagation
  - AC: Attach correlationId to outbound events and logs; visible in server logs.
  - Deps: MGA-NET-07  
  - SP: 2

- MGA-OBS-04 — Performance metrics and budgets
  - AC: CLS/LCP/TTI tracked; budgets set; CI fails on regressions.
  - Deps: MGA-OPS-02  
  - SP: 3

- MGA-OBS-05 — SIEM feed documentation
  - AC: Event schemas; transport; retention; security owner signoff.
  - Deps: MGA-OBS-01  
  - SP: 2

---

## Epic SEC: Hardening & Certification

Objective: Security controls, compliance gates, and penetration test readiness.  
Refs: `docs/MGAclient_OV.md:165`, `docs/MGAclient_OV.md:182`, `docs/MGAclient_arch.md:107`  
Labels: [security], [compliance]

- MGA-SEC-01 — CSP policy and enforcement
  - AC: CSP defined; enforced in environments; violations reported; no inline scripts.
  - Deps: MGA-APP-01  
  - SP: 5

- MGA-SEC-02 — Dependency audit in CI
  - AC: Snyk/OWASP checks; gating thresholds set; remediation workflow.
  - Deps: MGA-OPS-01  
  - SP: 3

- MGA-SEC-03 — Secrets scanning & config policy
  - AC: No secrets in repo; environment-driven config; scanners active.
  - Deps: MGA-OPS-01  
  - SP: 2  
  - Refs: `docs/MGAclient_OV.md:252`

- MGA-SEC-04 — HTTPS/WSS enforcement tests
  - AC: E2E confirms refusal to downgrade; mixed content blocked.
  - Deps: MGA-OPS-02, MGA-NET-04  
  - SP: 3

- MGA-SEC-05 — Determinative logic lint/semgrep rules
  - AC: Rule set blocks RNG/outcome logic in client; CI gate.
  - Deps: MGA-ENG-05  
  - SP: 3  
  - Refs: `docs/MGAclient_arch.md:11`

- MGA-SEC-06 — E2E security tests (CSP, clickjacking, XSS)
  - AC: Playwright suite covers core threats; passes in CI.
  - Deps: MGA-SEC-01, MGA-OPS-02  
  - SP: 5

- MGA-SEC-07 — Privacy & cookie policy alignment
  - AC: Consent surfaces; storage usage documented; legal signoff.
  - Deps: MGA-APP-05  
  - SP: 2

---

## Epic OPS: CI/CD & Release Governance

Objective: Pipelines, gates, artifact signing, promotion, and rollback.  
Refs: `docs/MGAclient_OV.md:252`  
Labels: [ops], [security]

- MGA-OPS-01 — Frontend CI pipeline
  - AC: Lint, unit, e2e stages; cache; parallelization; artifact upload.
  - Deps: MGA-APP-01  
  - SP: 3

- MGA-OPS-02 — Compliance gates in CI
  - AC: Fails build on inactivity timer, lock-on-disconnect, idempotency, HTTPS/WSS, PSP iframe policies.
  - Deps: MGA-APP-03, MGA-APP-04, MGA-NET-06, MGA-SEC-04, MGA-PAY-04  
  - SP: 5  
  - Refs: `docs/MGAclient_arch.md:107`

- MGA-OPS-03 — Artifact signing and provenance
  - AC: Build attestation; signed artifacts; verification at deploy.
  - Deps: MGA-OPS-01  
  - SP: 3

- MGA-OPS-04 — Environment promotion workflow
  - AC: Dev → QA → Staging → Prod with approvals; changelog auto-generated.
  - Deps: MGA-OPS-01  
  - SP: 3

- MGA-OPS-05 — Rollback & incident runbooks
  - AC: Tested rollback; incident response playbooks; on-call roster.
  - Deps: MGA-OPS-04  
  - SP: 3  
  - Refs: `docs/MGAclient_OV.md:252`

---

## Epic INF: Client Hosting & Edge Controls

Objective: Secure hosting and edge policies for the client application.  
Refs: `docs/MGAclient_OV.md:48`, `docs/MGAclient_OV.md:165`  
Labels: [infra], [security]

- MGA-INF-01 — Web hosting + CDN with HSTS
  - AC: HSTS enabled; TLS1.2+; gzip/brotli; cache policies defined.
  - Deps: MGA-GOV-01  
  - SP: 3

- MGA-INF-02 — WAF/CDN rules and headers
  - AC: Security headers injected; CSP alignment; DDoS protections enabled.
  - Deps: MGA-SEC-01  
  - SP: 3

- MGA-INF-03 — Environment-specific config delivery
  - AC: Config served per env; no secrets; integrity checked.
  - Deps: MGA-OPS-04  
  - SP: 2

- MGA-INF-04 — Domain & certificate management
  - AC: Automated renewals; monitored expiry; validated chain and ciphers.
  - Deps: MGA-GOV-01  
  - SP: 2

---

## Backlog Cross-References

- Prime Directive (no determinative client logic): `docs/MGAclient_arch.md:11` → SEC-05, ENG-05/06.
- Seamless Wallet / Idempotency: `docs/MGAclient_OV.md:129`, `docs/MGAclient_arch.md:41` → NET-06, OPS-02.
- Session mgmt & inactivity: `docs/MGAclient_OV.md:144`, `docs/MGAclient_arch.md:28` → APP-03, OPS-02.
- Compliance UI mandates: `docs/MGAclient_OV.md:89`, `docs/MGAclient_arch.md:70` → CMP-01..06.
- PCI boundaries: `docs/MGAclient_OV.md:176`, `docs/MGAclient_arch.md:98` → PAY-01..06, SEC-01.
- Documentation for MGA: `docs/MGAclient_OV.md:208` → GOV-04..07.

## Phasing Guidance (Mapping to Plan)

- Phase 0: GOV-01..07  
- Phase 1: APP-01..08, STA-01..05  
- Phase 2: NET-01..07, CMP-01..06  
- Phase 3: ENG-01..07  
- Phase 4: PAY-01..06  
- Phase 5: OBS-01..05, SEC-01..07, INF-01..04  
- Phase 6: OPS-01..05 + GOV closeout and audit readiness

