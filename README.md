# PQ-ID Prototype

Reproducible reference prototype for the IEEE Access paper **"A Privacy-Focused, Post-Quantum-Oriented Digital Identity System Using Blockchain and Zero-Knowledge Proofs"**: a full digital credential lifecycle on real post-quantum cryptography (ML-DSA-44, ML-KEM-512 via liboqs) with a 21,715-constraint Groth16 zero-knowledge circuit, Poseidon sparse-Merkle revocation, an EVM registry layer, and measured, regenerable benchmarks.

[![Paper](https://img.shields.io/badge/Paper-IEEE%20Access-blue)](https://ieeexplore.ieee.org/document/11614404)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

## Publication

The paper is published in IEEE Access.

- **Title**: A Privacy-Focused, Post-Quantum-Oriented Digital Identity System Using Blockchain and Zero-Knowledge Proofs
- **DOI**: 10.1109/ACCESS.2026.3715111
- **Manuscript ID**: Access-2026-15409

Full article: [ieeexplore.ieee.org/document/11614404](https://ieeexplore.ieee.org/document/11614404)

## What it demonstrates

A full credential lifecycle on real cryptography:

```
DID registration -> credential issuance -> off-circuit Dilithium verify ->
witness build (depth-32 Merkle non-membership) -> Groth16 proof (snarkJS + rapidsnark) ->
verifier resolve + equality + verify + nonce -> revocation -> re-verify rejects
```

plus a benchmark harness that regenerates the paper's PQC, Groth16 (dual prover on byte-identical inputs), and centralized-baseline tables as measured JSON, and a passing negative test (revoked credential produces no accepted proof).

The prototype exists to produce real, measured evidence for the paper's claims and to make its protocol executable. It optimizes for correctness, reproducibility, and reviewer defensibility rather than product polish. Every metric is measured at runtime and written to `results/*.json` with the host CPU, OS build, and toolchain embedded; nothing is hard-coded or tuned to match the paper.

## Measured results

All figures below are committed, machine-generated measurements in `results/*.json` (host: Intel Core i7-1355U, Windows 11 + WSL2; full mapping in `RESULTS.md`):

| Metric | Value |
| --- | --- |
| Circuit size | 21,715 constraints, 5 public / 43 private signals (BN254) |
| Groth16 prove (rapidsnark, native) | 177.4 ms cool, 258.6 ms sustained mean |
| Groth16 prove (snarkJS) | ~981 ms cool, 2095 ms sustained mean |
| Groth16 verify | 39.9 ms |
| On-chain verification gas | 242,931 (bare `verifyProof`), 268,893 via a calling contract with a result store |
| ML-DSA-44 (Dilithium-II) | sign 0.19 ms, verify 0.06 ms |
| ML-KEM-512 (Kyber-512) | encap 0.018 ms, decap 0.022 ms |
| Proof size | 723 B (snarkJS) / 707 B (rapidsnark) JSON encoding |

## Quick start

```bash
# 1. native toolchain (liboqs, GMP, rapidsnark, python venv) inside WSL2/Linux:
make provision            # or: bash docker/provision-wsl.sh

# 2. toolchains + ptau (pinned) + circuit + zkey (pinned) + Solidity verifier:
make setup

# 3. tests (unit, integration, negative, interop, determinism):
make test

# 4. measured benchmarks -> results/*.json, then regenerate RESULTS.md:
make bench
make tables

# 5. one-command lifecycle demo (incl. post-revocation rejection):
make demo
```

On Windows, run `make` from Git Bash, or use the `npm run` scripts directly (see `package.json`). Native pieces are containerized under `docker/` for a clean Linux host.

## Key technical decisions

- **In-circuit hash: Poseidon, not SHA-3.** A single Keccak block costs 239,176 R1CS constraints (measured in `circuits/research/`), incompatible with the ~21k budget. The circuit uses Poseidon for the depth-32 Merkle tree; `credID = SHA3-256(cred || pk_issuer)` is computed off-circuit, with issuer binding enforced by the wallet's Dilithium check and the verifier's on-chain `pk_issuer` equality.
- **Five public signals**: `[Poseidon(pk_issuer), revRoot, Poseidon(policy), nonce, stmtCode]`, with verifier-domain separation (`stmtCode = Poseidon(STMT_V1, domainTag)`) so cross-verifier replay is rejected cryptographically.
- **credID field packing**: the 256-bit digest is carried as two 128-bit limbs; the SMT key is `Poseidon(credIDHi, credIDLo)`, identical in the JS tree and the circuit.
- **Holder binding**: `holderCommit = Poseidon(holderSecret)`; the circuit proves knowledge of `holderSecret`.
- **Revocation**: depth-32 Poseidon sparse-Merkle tree; non-membership is an empty leaf at the key.
- **DID method**: `did:pq:<base58(SHA3-256(pk)[:20])>`.
- **Ledger**: single-node EVM (Anvil) hosting DID, Issuer, Revocation, and Schema registries. Multi-node BFT is future work.

See `ARCHITECTURE.md` for the full trust-boundary and public-signal specification.

## Measurement discipline

- Every timing run is guard-protected (refuses to run on a contended machine), executed at least 3 times in independent processes, and ships its controls (affinity policy, governor, power source) inside the results JSON, with rapidsnark inputs staged on native ext4.
- The two Groth16 provers are compared only on byte-identical zkey and witness inputs (SHA-256 equality asserted), on matching cold and amortized bases.
- The circuit passes a build-blocking circomspect gate (2 findings, both the intentional public-input binding squares, triaged with justification).
- A malicious-wallet probe (`harness/malicious_wallet_probe.ts`) honestly demonstrates the documented trust boundary: the issuer's Dilithium signature is verified off-circuit by design (in-circuit ML-DSA would cost millions of constraints), so a wallet that skips the check can produce an accepted proof for a forged credential. Mitigations are future work.

## Repository layout

```
circuits/            credential_auth.circom (Poseidon) + research/ SHA-3 cost report
setup/               ptau pin, zkey/vkey gen, SHA-256 pins, Solidity export
packages/
  common/            encodings, hashing, did:pq, metadata, stats
  pqc/               liboqs ML-DSA-44 + ML-KEM-512 bridge + bench + Kyber demo
  issuer/            VC build + credID + Dilithium sign
  wallet/            off-circuit checks + witness + dual prover
  revocation/        depth-32 SMT
  verifier/          resolve + equality + Groth16.verify + nonce
  ledger/            EVM registries + did:pq resolver
  baseline/          OAuth2 + ECDSA P-256 + PostgreSQL comparison baseline
onchain/             Solidity Groth16 verifier deploy + gas measurement
harness/             benches, table generator, e2e, negative test, malicious probe
cli/                 demo CLI
fixtures/            VC schema, revoked-set, bench config
results/             measured JSON (+ sample/)
tests/               unit, integration, negative, interop, determinism
docker/              reproducible liboqs + rapidsnark builds + compose
```

Every quantitative output is labeled, matching the paper: `[M]` measured, `[S]` simulated/estimated, `[A]` assumption, `[F]` future work.

## Documents

- `RESULTS.md`: every measured value mapped to its paper table cell, with a divergences section.
- `ARCHITECTURE.md`: subsystems, data flow, trust boundary, five public signals, Assumption 5.
- `REPRODUCE.md`: exact commands and Docker paths to regenerate every number; ptau/zkey pin verification.
- `MANUSCRIPT_RECONCILIATION.md`: the precise manuscript edits the prototype implies.

Task-focused guides live under [`docs/`](docs/): [`SETUP.md`](docs/SETUP.md), [`REPRODUCIBILITY.md`](docs/REPRODUCIBILITY.md), [`BENCHMARKS.md`](docs/BENCHMARKS.md), [`TESTING.md`](docs/TESTING.md), [`TRACEABILITY.md`](docs/TRACEABILITY.md), [`VALIDATION_REPORT.md`](docs/VALIDATION_REPORT.md), and [`MISSING_ARTIFACTS.md`](docs/MISSING_ARTIFACTS.md). See [`CITATION.cff`](CITATION.cff) for how to cite.

## Author

**Kushal Sachdeva** ([ORCID 0009-0007-5126-0696](https://orcid.org/0009-0007-5126-0696)), Research Intern, Algorand Foundation; Vasant Valley School, New Delhi, India.

More of my work: [github.com/Kushal-Sachdeva78](https://github.com/Kushal-Sachdeva78), including [autonomous RoboCup soccer robots](https://github.com/Kushal-Sachdeva78/VVS-Ballers-RoboCup) (9th place, RoboCup 2026).

## License

MIT. See [LICENSE](LICENSE).
