# MGA Compliant Game Client: Implementation Plan & Technical Overview

Document Version: 1.0  
Date: 2025-09-09  
Status: For Implementation  
Audience: CTO, Lead Architects, Frontend Team, Backend/RGS, DevOps, QA

This document consolidates the implementation plan for building an MGA-compliant client and surrounding platform interfaces. It derives directly from and references:

- docs/MGAclient_OV.md (System-level spec and MGA build sheet)
- docs/MGAclient_arch.md (Frontend architectural spec)

Where helpful, file and line references are included for traceability.

---

Companion Backlog: See `docs/MGAclient_Backlog.md` for a ticketized, Jira-style backlog derived from this plan (epics → stories with acceptance criteria, estimates, and dependencies).

## 1. Technical Overview

- Objective: Build an MGA-compliant B2C game client (slots and similar) that is a secure, presentation-only SPA, driven exclusively by an authoritative Remote Game Server (RGS). See `docs/MGAclient_OV.md:5`, `docs/MGAclient_arch.md:11`.
- Prime Directive: No determinative logic in the client; all gameplay outcomes, balances, and session state come from the RGS over WSS with server-authoritative state. See `docs/MGAclient_arch.md:11`, `docs/MGAclient_OV.md:11`.
- Separation of Concerns: Client renders UI and captures input; RGS controls RNG, round resolution, wallet debits/credits, and state transitions. See `docs/MGAclient_OV.md:24`, `docs/MGAclient_OV.md:39`.

## 2. System Architecture

- Tiers: Client SPA ↔ API Gateway ↔ RGS ↔ Wallet/Payments ↔ Primary DB; plus centralized logging/auditing and a Malta-located replication server for regulatory data. See `docs/MGAclient_OV.md:24`, `docs/MGAclient_OV.md:39`, `docs/MGAclient_OV.md:57`.
- Data Flow: Login over HTTPS → JWT; persistent WSS to RGS for gameplay; idempotent financial calls; immediate replication of committed transactions to MGA server in Malta. See `docs/MGAclient_OV.md:39`, `docs/MGAclient_OV.md:129`, `docs/MGAclient_OV.md:57`.
- Hosting: ISO 27001:2013 certified facilities in EEA; critical components isolated; replication server physically in Malta with secure, low-latency link. See `docs/MGAclient_OV.md:48`, `docs/MGAclient_OV.md:57`.

## 3. Frontend Architecture

- SPA Framework: React 18 + TypeScript + Vite (fast builds, TS safety), or Vue 3 equivalent (choose one and stick to it). See `docs/MGAclient_OV.md:68`, `docs/MGAclient_arch.md:22`.
- State Management: Redux Toolkit (RTK) with slices matching compliance needs; RTK Query optional for HTTPS. See `docs/MGAclient_arch.md:60`.
- Networking: Axios (HTTPS) with auth/error interceptors; native/WebSocket client with backoff, heartbeat, and full resync. See `docs/MGAclient_arch.md:41`.
- Rendering Engine: PixiJS or Phaser for WebGL canvas rendering; engine is pure view/input and must pause on suspension or disconnect. See `docs/MGAclient_arch.md:90`.
- UI System: Headless component library + Tailwind or MUI; dedicated Compliance UI (balance, clock, reality checks, limits, self-exclusion). See `docs/MGAclient_OV.md:89`, `docs/MGAclient_arch.md:70`.
- Security in Client: Enforce HTTPS/WSS, CSP, no secrets in bundle, strict token storage/rotation, and 30-minute inactivity logout. See `docs/MGAclient_OV.md:165`, `docs/MGAclient_arch.md:28`.

## 4. Core Client Modules

### 4.1 CoreApp

- App shell, layout, session inactivity (28-min warning, 30-min logout), global suspension overlay on connection loss or reality checks. See `docs/MGAclient_arch.md:28`.

### 4.2 ApiService

- HTTPS with JWT interceptor (401 → global logout), WSS connect/send; every bet includes UUIDv4 `transactionId` idempotency key; exponential backoff and resync. See `docs/MGAclient_arch.md:41`.

### 4.3 StateService

- Central store with slices: session, player, game, responsibleGaming. Includes `isConnectionLost` and `isGameSuspended`. See `docs/MGAclient_arch.md:60`.

### 4.4 ComplianceUI

- ComplianceBar (real-time balance, clock); RealityCheckModal (suspends gameplay and shows stats); LimitsManager (tighten immediate, loosen with 24h cool-off); SelfExclusion flow (immediate logout). See `docs/MGAclient_arch.md:70`.

### 4.5 GameEngine

- Visualizes server outcomes; checks suspension/connection flags each frame; ignores input when suspended/disconnected. See `docs/MGAclient_arch.md:90`.

### 4.6 PaymentIntegration

- PSP-hosted iframe, no card fields in client; `postMessage` to detect completion. See `docs/MGAclient_arch.md:98`, `docs/MGAclient_OV.md:176`.

## 5. Message Contracts (Client ↔ RGS, WSS)

- Authenticate: `{ type: "auth", jwt }` → `{ type: "auth.ok", session, player, gameState }` or `{ type: "auth.err" }`. See `docs/MGAclient_OV.md:39`, `docs/MGAclient_OV.md:144`.
- Place Bet (Idempotent): `{ type: "bet.place", gameId, stake, lines, transactionId, deviceInfo }` → `{ "bet.ack" }` and then `{ "round.result", result, balance }`. See `docs/MGAclient_OV.md:129`, `docs/MGAclient_arch.md:41`.
- State Sync: `{ type: "state.sync" }` → `{ "state.full", gameState, player.balance, cooldowns, sessionStats }` on reconnect. See `docs/MGAclient_arch.md:41`.
- Balance Update: `{ "balance.update", balance, currency }` for real-time display. See `docs/MGAclient_arch.md:70`.
- Responsible Gaming: `{ "reality.check", sessionStats, interval }` → UI modal; `{ "limits.updated", cooldownUntil }`, `{ "self.exclude" }` → force logout via 401. See `docs/MGAclient_OV.md:98`, `docs/MGAclient_arch.md:70`.
- Session Control: `{ "session.terminated", reason }` → global logout; client also enforces inactivity timeout. See `docs/MGAclient_OV.md:144`, `docs/MGAclient_arch.md:28`.
- Errors: `{ "error", code, message, correlationId }` with non-deterministic client action; server remains authority.

## 6. Compliance Requirements in Client

- 30-Min Inactivity: Warning at 28 minutes; auto-logout at 30; clear state and redirect. See `docs/MGAclient_arch.md:28`.
- Reality Checks: Suspend gameplay, display session totals/net win-loss, require explicit acknowledgement to continue. See `docs/MGAclient_arch.md:70`.
- Limits: Tighten immediate; loosen after 24h cool-off surfaced in UI with countdown. See `docs/MGAclient_arch.md:70`.
- Self-Exclusion: Immediate, final; server ends session, client logs out. See `docs/MGAclient_arch.md:70`.
- Seamless Wallet: All bets/wins go through server; client never mutates balance locally. See `docs/MGAclient_OV.md:129`.

## 7. Security & SDLC

- Transport: Enforce HTTPS/WSS only; secure cookies if used, HSTS, TLS1.2+. See `docs/MGAclient_OV.md:165`.
- AppSec: CSP, SRI for third-party scripts, dependency auditing, no secrets in code, environment-driven config, code review gate for compliance items. See `docs/MGAclient_OV.md:165`, `docs/MGAclient_OV.md:252`.
- PCI Scope: Payments only via PSP iframe; no PAN in client or APIs. See `docs/MGAclient_OV.md:176`.
- Session: JWT with short TTL and refresh; 401 global handler; 30-min inactivity enforced client-side; server maintains canonical session state/expiry. See `docs/MGAclient_OV.md:144`, `docs/MGAclient_arch.md:41`.

## 8. Regulatory Logging (Server-Side, Client-Triggered)

- Events to Log: Authentication, session start/stop, bets, round results, balance changes, limits changes, self-exclusions, connection loss/reconnects, errors with correlationId. See `docs/MGAclient_OV.md:149`.
- Immutability: Append-only, tamper-evident store; replicated to Malta server in near real-time. See `docs/MGAclient_OV.md:57`, `docs/MGAclient_OV.md:149`.
- Idempotency: Unique `transactionId` for all financial actions; RGS deduplicates and logs once. See `docs/MGAclient_arch.md:41`.

## 9. Infrastructure & Environments

- Regions & DCs: EEA-certified primary; Malta replication host. Isolate RNG, jackpots, and player/financial DBs (private VPC). See `docs/MGAclient_OV.md:48`, `docs/MGAclient_OV.md:57`.
- Environments: Dev, QA, Staging (cert mirror), Production; separate secrets and CI/CD lanes. See `docs/MGAclient_OV.md:165`, `docs/MGAclient_OV.md:252`.
- Networking: API Gateway/WAF, L7 rate limiting, DDoS protections, mutual TLS for internal services.
- Observability: Structured logs, metrics (SLOs), distributed tracing (correlationId from client), SIEM feed.

## 10. Preferred Tech Stack

- Frontend: React 18, TypeScript, Vite, Redux Toolkit, Axios, native WebSocket, PixiJS/Phaser, Tailwind/MUI, Zod/Yup (validation), UUID v4, date-fns/luxon, i18n.
- Backend (indicative): RGS (Node/NestJS or Java/Spring Boot), Wallet service, OAuth2/OIDC IdP, Postgres primary, Kafka (event bus), Elasticsearch/OpenSearch (audit index), S3-compatible WORM/Azure Immutable Blob for regulatory archives, Redis (caching/queues).
- Dev Tooling: ESLint/Prettier, Jest/Vitest, Playwright, GitHub Actions/Azure DevOps, Snyk/OWASP Dependency-Check, Semgrep, Trivy, SonarQube.

## 11. APIs and Contracts

- HTTPS APIs: Auth (OIDC/OAuth2), limits, self-exclusion, lobby/catalog, non-financial operations. 401 global handling. See `docs/MGAclient_arch.md:41`.
- WSS Protocol: Auth handshake; delta/state messages; deterministic round resolution events; heartbeats and pings; resync on reconnect. See `docs/MGAclient_OV.md:39`, `docs/MGAclient_arch.md:41`.
- Idempotent Financials: `transactionId` on bet; server returns final authoritative result; client never retries blindly—only on explicit server signal. See `docs/MGAclient_arch.md:41`, `docs/MGAclient_OV.md:129`.

## 12. Testing & Certification

- Unit & Contract Tests: Reducers/selectors; ApiService interceptors; WebSocket connect/retry; idempotency key generation; suspension/lock behavior. See `docs/MGAclient_arch.md:107`.
- E2E (Playwright): Full gameplay, reconnect scenarios, reality checks, limits changes, self-exclusion, inactivity logout.
- Security Testing: SAST, DAST, dependency audits; CSP/SRI verification; TLS checks. See `docs/MGAclient_OV.md:165`.
- Performance: WSS fan-in tests, animation smoothness under packet loss, reconnect under load.
- Third-Party & MGA: Engage MGA-approved labs for RNG/game certification and pen tests; pass MGA audit gates. See `docs/MGAclient_OV.md:192`, `docs/MGAclient_OV.md:199`.

## 13. CI/CD & Governance

- Pipelines: Build → lint → test (unit) → test (e2e headless) → bundle analysis → SAST/DAST gates → sign artifacts → deploy to env. See `docs/MGAclient_OV.md:252`.
- Branching: GitFlow (feature → develop → release → main) with mandatory code review including compliance checklist. See `docs/MGAclient_OV.md:252`.
- Config/Secrets: Environment-specific, injected at build/deploy; no secrets in repo. See `docs/MGAclient_OV.md:252`.
- Release Gating: Block releases if compliance checks fail (inactivity timer, lock-on-disconnect, idempotency key in bets, HTTPS/WSS enforced, PSP iframe usage only). See `docs/MGAclient_arch.md:107`.

## 14. Work Breakdown (Epics)

- Epic: App Shell & Session — App shell, routing, layout, inactivity timer, auth flows, 401/logout. See `docs/MGAclient_arch.md:28`.
- Epic: Networking & State — Axios + JWT, WebSocket service (backoff, heartbeat, resync), Redux slices and selectors. See `docs/MGAclient_arch.md:41`, `docs/MGAclient_arch.md:60`.
- Epic: Compliance UI — ComplianceBar (balance, clock), reality check modal, limits manager, self-exclusion. See `docs/MGAclient_arch.md:70`.
- Epic: Game Engine Integration — Engine bootstrap, asset pipeline, frame loop with suspension checks, input gating. See `docs/MGAclient_arch.md:90`.
- Epic: Payments Integration — PSP iframe integration, postMessage handling, error/receipt surfacing. See `docs/MGAclient_arch.md:98`.
- Epic: Observability & Telemetry — Client logs/metrics, correlationId propagation, error reporting, feature flags.
- Epic: Hardening & Cert — Security headers/CSP, perf optimizations, accessibility, localization, audit support. See `docs/MGAclient_OV.md:182`.

## 15. Detailed Implementation Steps

### 15.1 Foundations

- Repo scaffolding (workspace, CI), Vite+TS app, ESLint/Prettier, Jest/Vitest, Playwright.
- Path-based routing; environment config loader; feature flagging.

### 15.2 Session & Auth

- OIDC sign-in; token storage/rotation; 28-min warning modal; 30-min auto-logout. See `docs/MGAclient_arch.md:28`.

### 15.3 ApiService

- Axios instance; JWT interceptor; 401 global logout; standardized error model; retry policy for idempotent GETs only.

### 15.4 WebSocket Service

- Connect(token), send(payload with `transactionId` for financial ops), backoff, heartbeat, reconnection with `state.sync`. Lock UI via `isConnectionLost`. See `docs/MGAclient_arch.md:41`.

### 15.5 StateService

- Slices: session, player, game, responsibleGaming; selectors; hydration/dehydration on auth transitions. See `docs/MGAclient_arch.md:60`.

### 15.6 Compliance UI

- ComplianceBar: real-time balance subscription; system clock.
- RealityCheckModal: timers, sessionStats, blocking UX until acknowledge/exit. See `docs/MGAclient_arch.md:70`.
- LimitsManager: tighten/loosen logic, cooldown surface with countdown. See `docs/MGAclient_arch.md:70`.
- SelfExclusion: irreversible confirmation; server-initiated session end; 401 handler triggers logout. See `docs/MGAclient_arch.md:70`.

### 15.7 Game Engine

- Render loop gating on suspension/disconnect; outcome visualization only; deterministic animations seeded from server payloads.

### 15.8 Payments

- PSP iframe host; postMessage handshake; success/failure events; UX for KYC/AML triggers.

### 15.9 Accessibility & UX

- WCAG AA where practical; keyboard navigable modals; focus management on suspension dialogs.

### 15.10 Security Controls

- CSP (default-src 'self'; frame-ancestors 'none'; allow PSP domain for frame-src), SRI, HSTS, no inline scripts, no `eval`.

### 15.11 E2E & Load

- Playwright scenarios; WebSocket chaos tests (drop, jitter); perf budgets for TTI and animation frame stability.

## 16. MGA Documentation Deliverables

- System Architecture Pack: Network diagrams, components, geo/hosting, IP ranges. See `docs/MGAclient_OV.md:208`.
- Application Architecture: Inventory, versions, hosting map, sequence diagrams (client ↔ RGS flows). See `docs/MGAclient_OV.md:208`.
- Security Policies: InfoSec policy, change management, incident response. See `docs/MGAclient_OV.md:208`.
- Data Replication Plan: Topology, RPO/RTO, failover drills, validation. See `docs/MGAclient_OV.md:208`.

## 17. Risks & Mitigations

- Client Logic Drift: Strict lint rule/code review gate against determinative logic; property tests assert engine never mutates authoritative state. See `docs/MGAclient_arch.md:11`.
- Idempotency Gaps: Centralized `sendBet()` enforces `transactionId`; server rejects duplicates; add contract tests. See `docs/MGAclient_arch.md:41`.
- Reconnect Edge Cases: Mandatory `state.sync` on reconnect; deterministic UX re-entry points. See `docs/MGAclient_arch.md:41`.
- PCI Scope Creep: Payments only in PSP iframe; CSP denies card fields elsewhere. See `docs/MGAclient_OV.md:176`.
- Compliance Regression: Automated “Compliance Gate” in CI using checklist assertions from `docs/MGAclient_arch.md:107`.

## 18. Project Phasing

- Phase 0 – Governance & Vendors: Select EEA DC and Malta replication provider; engage MGA-approved labs; finalize architecture docs for submission. See `docs/MGAclient_OV.md:228`.
- Phase 1 – Foundations: Repo, CI/CD, security baselines, scaffolding, design system, CoreApp/session, ApiService, WebSocket service.
- Phase 2 – Compliance Features: ComplianceBar, reality checks, limits + cooldown, self-exclusion; 401/global logout; locking behavior.
- Phase 3 – Engine & Gameplay: Integrate PixiJS/Phaser; implement rendering + input gating; end-to-end game loop with server outcomes.
- Phase 4 – Payments & Lobby: PSP iframe integration; lobby/catalog; session stats views.
- Phase 5 – Hardening: Perf, accessibility, localization, SAST/DAST, CSP, observability, load/chaos tests.
- Phase 6 – Certification: Internal QA, third-party testing, MGA audits and document pack. See `docs/MGAclient_OV.md:186`, `docs/MGAclient_OV.md:192`, `docs/MGAclient_OV.md:199`.
- Phase 7 – Go-Live & Operate: Cutover, monitoring, runbooks, incident drills, continuous compliance.

## 19. Acceptance Criteria (Sample, per module)

- CoreApp: Inactivity timer warns at 28m, logs out at 30m; verified by E2E test; 401 handling proven. See `docs/MGAclient_arch.md:28`.
- ApiService: Bets always include `transactionId`; 401 handler dispatches logout; WSS resync works after simulated drop. See `docs/MGAclient_arch.md:41`.
- ComplianceUI: Reality check blocks engine; limits show 24h countdown; self-exclusion ends session immediately. See `docs/MGAclient_arch.md:70`.
- Engine: Input ignored during suspension/disconnect; outcome visualization matches payloads.

## 20. Traceability (Excerpt)

- Prime Directive → Engine no state logic: `docs/MGAclient_arch.md:11` → GameEngine implementation/tests.
- Seamless Wallet → No client balance mutation: `docs/MGAclient_OV.md:129` → ComplianceBar subscribes to server balance only.
- Idempotency → transactionId on bets: `docs/MGAclient_arch.md:41` → ApiService `sendBet()` unit test.
- Session mgmt → inactivity + 401 logout: `docs/MGAclient_OV.md:144`, `docs/MGAclient_arch.md:28` → CoreApp tests.
- UI Mandates → balance/clock/reality checks: `docs/MGAclient_OV.md:89`, `docs/MGAclient_arch.md:70` → ComplianceUI components.

## 21. Next Steps

- Confirm Stack Choices: React + RTK + PixiJS, or request Vue alternative confirmation. See `docs/MGAclient_OV.md:68`.
- Approve Plan & Phases: Lock scope and sequencing; assign owners per epic.
- Provision CI & Baselines: Initialize repo, CI, and compliance gates based on `docs/MGAclient_OV.md:252`.
- Kickoff Development: Start Phase 1 tasks and produce first architecture diagrams for MGA submission. See `docs/MGAclient_OV.md:208`, `docs/MGAclient_OV.md:261`.

---

## References

- docs/MGAclient_OV.md
  - 5: 1.0 Executive Mandate & Core Architectural Principles
  - 24: 2.1 High-Level Components
  - 39: 2.2 Data & Communication Flow
  - 48: 3.1 Primary Hosting Environment
  - 57: 3.2 MGA Real-Time Data Replication Server
  - 68: 4.1 Technology Stack
  - 89: 4.3 Mandatory UI & Display Requirements
  - 98: 4.4 Player Protection Features (UI Implementation)
  - 118: 5.1 API Architecture
  - 129: 5.2 "Seamless Wallet" Transaction Flow (CRITICAL)
  - 144: 5.3 Session Management
  - 149: 5.4 Regulatory Data Logging Matrix (CRITICAL)
  - 165: 6.1 Secure SDLC (ISO 27001 Annex A.14)
  - 176: 6.2 PCI DSS Level 1 Compliance
  - 186: 7.0 Testing, Verification & Go-Live Plan
  - 192: Phase 2: Mandatory Third-Party Auditing (Pre-Launch)
  - 199: Phase 3: MGA Official Audits (Go/No-Go Gates)
  - 208: 8.0 Documentation Requirements
  - 228: 9.1 Critical Compliance Task Checklist
  - 252: 9.2 Establishing a Compliant Development Workflow
  - 261: 9.3 Next Steps

- docs/MGAclient_arch.md
  - 11: 1.1 The Prime Directive
  - 22: 2.0 High-Level Client Architecture
  - 28: 3.1 CoreApp Module
  - 41: 3.2 ApiService Module
  - 60: 3.3 StateService Module
  - 70: 3.4 ComplianceUI Module
  - 90: 3.5 GameEngine Module
  - 98: 3.6 PaymentIntegration Module
  - 107: 4.0 Developer Compliance Checklist (For Code Reviews)
