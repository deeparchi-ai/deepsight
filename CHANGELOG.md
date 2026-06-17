# Changelog

All notable changes to DeepSight. Format loosely follows
[Keep a Changelog](https://keepachangelog.com/). Until now the version history
lived implicitly inside `task_registry.json`; this file makes it explicit.

## [Unreleased]

Issue-driven hardening (PRs against `main`; see GitHub Issues `P0`/`P1`/`P2`):
- **Docs** — add MIT `LICENSE`; rewrite `README` to document the methodology (#1, #2).
- **Council rigor** — `references/council-protocol.md`: model-family diversity metric, disagreement floor, honest-degrade ladder tied to evidence tier, algorithmic consensus, adjudication independence ("referee ≠ player") (#3, #4).
- **Method hardening** — closed adversarial-conflict loop + cross-model red-team; `examples/` worked walkthrough; Phase 3 reframed as optional delivery layer; `references/council-capability-spec.md` (vendor-agnostic audit-seat requirements) (#5, #6, #7).
- **Calibration & depth** — evidence-tier thresholds given rationale; framework-selection disambiguation; depth三件套 (sensitivity / 2nd-order / time-gradient) restored as Phase 1.6; this `CHANGELOG` + `task_registry.schema.json` (#8, #9).

## [2.5.0] — 2026-06-11

UX refactor.
- Flattened step numbering to **Phase 0/1/2/3**; merged the audit layer.
- Added **lightweight mode** (2 constraint rounds + 1 adversarial pass).
- Split templates into `references/` (`constraint-chains.md`, `adversarial-templates.md`, `framework-catalog.md`).
- Fixed the「六套」→「八套」text bug. `SKILL.md` 688 → 190 lines.

## [2.4.0] — 2026-06-11

Breadth expansion.
- Frameworks **6 → 8**: added Organization Design (Galbraith Star + McKinsey 7S) and Platform/Ecosystem (Rochet & Tirole + network effects).
- Adversarial templates **6 → 7**: added Geopolitical / Supply-Chain Security.
- Fixed file duplication / numbering errors.
- _Note: depth work (sensitivity analysis, 2nd-order effects, time-gradient projection) was shelved here — restored in Unreleased (#8)._

## [2.3.0]

- Cross-provider audit infrastructure (the original Step 3c → now Phase 2 Council); see `references/council-model-setup.md`.

## [2.0.0]

- Earlier methodology baseline. Notable lesson retained in `SKILL.md`: a fully self-coherent analysis (the v2.0 "B₄C optimal" chain) can still be factually wrong — 自洽 ≠ 正确.

[Unreleased]: https://github.com/deeparchi-ai/deepsight/compare/main...HEAD
