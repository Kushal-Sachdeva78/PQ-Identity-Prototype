# Changelog

All notable changes to this repository are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

This is an archival research artifact: the code is meant to stay functionally
identical to what produced the paper's evaluation, so changes after v1.0 are
documentation and repository tooling only.

## [Unreleased]

### Added

- README top block: one-line description, quick-start command, demo GIF
  placeholder and a status note pointing to
  [PQ-Identity-Prototype-v4](https://github.com/Kushal-Sachdeva78/PQ-Identity-Prototype-v4).
- Issue forms for reproduction problems and for documentation or portability
  improvements (`.github/ISSUE_TEMPLATE/`).
- A response-time note in `CONTRIBUTING.md`.
- This changelog.

### Changed

- README: added the IEEE Access publication section and a measured results
  summary (2026-07-18).
- README: linked the badges and added the published IEEE Xplore article link
  (2026-07-22).
- README: updated the author's affiliation (2026-10-21).

## [1.0] - 2026-06-13

### Added

- Initial public release of the PQ-ID prototype for IEEE Access manuscript
  Access-2026-15409: a credential lifecycle on ML-DSA-44 and ML-KEM-512
  (liboqs), a 21,715-constraint Groth16 credential circuit with Poseidon
  sparse-Merkle revocation, an EVM registry layer, and a measured, regenerable
  benchmark harness.

### Changed

- Reconciled the public release to the submitted manuscript (V6.6).
- CI: scoped the Linux test job to the cross-platform suites.
- `docs/VALIDATION_REPORT.md` records the green CI run and the scoped Linux job.
- Versioned the paper-claim reference fields to V6.6; no measured values changed.

[Unreleased]: https://github.com/Kushal-Sachdeva78/PQ-Identity-Prototype/compare/v1.0...HEAD
[1.0]: https://github.com/Kushal-Sachdeva78/PQ-Identity-Prototype/releases/tag/v1.0
