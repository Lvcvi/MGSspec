# MGA Client: Additional Ideas & Future Enhancements

Version: 1.0  
Date: 2025-09-09  
Status: Ideas/Backlog Candidates

Purpose: Supplement the core backlog with high‑value, optional enhancements that improve resilience, observability, developer velocity, and audit readiness while staying aligned with MGA mandates.

## Engineering & Developer Experience

- RGS Protocol Simulator: Local CLI + mock server to simulate RGS WSS flows (auth, bet, result, state.sync) with scripted timelines and packet loss/jitter injection.
- Contract Tests Pack: JSON schemas for WSS payloads + auto‑generated TypeScript types and validators (Zod) to enforce message contracts in CI.
- Determinism Harness: Seeded playback tool that replays server outcomes to validate rendering determinism frame‑by‑frame.
- Asset Pipeline: Texture atlas generation, LQIP thumbnails, and automatic sprite sheet optimization in CI with size budgets.
- Feature Flags Admin: Minimal internal UI to toggle compliance intervals and enable/disable modules per environment.

## Reliability & Chaos

- WebSocket Chaos Tests: CI job that randomly drops connections, delays messages, and asserts correct lock/resume behavior.
- Synthetic Monitors: Headless probes (from Malta PoP) for login, WSS handshake, and first bet flow with alerting.
- Error Budget Policy: SLOs for WSS availability and UI lock correctness; integrate into release gates.

## Compliance & Audit Readiness

- Audit Log Browser (Internal): Read‑only UI for viewing server‑side regulatory logs by player/session/transactionId with export to WORM storage.
- Compliance Gate CLI: Verifies inactivity timers, idempotency key presence, PSP iframe isolation, and HTTPS/WSS headers in pre‑release checks.
- Reality Check Tuning: Configurable intervals by jurisdiction with feature flag controls and content slots for localized messaging.

## Security & Privacy

- Security Header Scanner: Automated verification of CSP, HSTS, frame‑ancestors, and SRI across environments post‑deploy.
- Device Context Telemetry: Non‑deterministic device info (UA, OS, timezone) for fraud signals sent alongside bets (documented, privacy‑aware).
- Session Anomaly Detection: Simple client‑side heuristics that surface unusual reconnect patterns to the telemetry backend (no local blocking logic).

## Accessibility & Localization

- Pseudo‑Localization: Automatic lengthening and diacritic replacement to catch truncation issues; RTL smoke tests.
- Accessibility Audit: Integrate axe‑core scripts into Playwright E2E and fail builds on critical violations.

## Performance & UX

- Code‑Splitting Strategy: Route‑ and engine‑level chunks with prefetch hints; CI budget fail if bundle exceeds thresholds.
- Animation Frame Budgeting: Telemetry on frame drops during spins; alerts when exceeding defined thresholds under network stress.

## Operations & Docs

- Living Diagrams: Generate architecture and sequence diagrams from source (e.g., PlantUML/Mermaid) and publish with each release.
- Runbooks: Game day drills for WSS outage, PSP iframe failure, and Malta link degradation with RTO/RPO target checks.
- Data Residency Checks: Automated verification that regulatory data mirrors are healthy and up‑to‑date in Malta.

