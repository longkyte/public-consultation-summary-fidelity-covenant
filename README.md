# Public Consultation Summary Fidelity Covenant

## Live deployment

The reviewed source is deployed on GenLayer Studionet at [`0x2e8bc9c4611918B5ffDCDFeA76809591F0c30B95`](https://explorer-studio.genlayer.com/address/0x2e8bc9c4611918B5ffDCDFeA76809591F0c30B95). The deployment transaction [`0x6150701b25aa52028cbef2d938009c1a9f9237b5b670532070960c45acce1c04`](https://explorer-studio.genlayer.com/tx/0x6150701b25aa52028cbef2d938009c1a9f9237b5b670532070960c45acce1c04) is finalized, with leader execution `SUCCESS` and a three-agree / two-idle quorum. `genlayer schema` readback exposed the complete contract interface; this is the deployment used by all evidence below.

This Intelligent Contract seals a bounded consultation corpus, records a hash-bound public summary, and evaluates whether the summary represents the material source documents. Deterministic input and lifecycle checks run before the nondeterministic comparison. Source retrieval or hash validation failure returns a fail-closed `UNRESOLVED` result rather than a favourable finding.

## What is in this repository

- `contracts/public_consultation_summary_fidelity_covenant.py` — contract.
- `docs/specification.md` — lifecycle, authority, and assessment model.
- `samples/faithful-consultation.json` — bounded public-source candidate.
- `samples/material-omission.json` — explicitly synthetic, direct-test-only material-omission fixture; it is not presented as public evidence.
- `tests/test_public_consultation_summary_fidelity_covenant.py` — unit and adversarial test suite.

## Reproducible local verification

In isolated WSL, Python 3.13.14 with `genlayer-py==0.16.3`, `genlayer-test==0.29.2`, `genvm-linter==0.11.0`, and `pytest==9.1.1` completed `uv pip check`, GenVM lint/check, type/schema checks, and **46 passed** tests.

```powershell
wsl.exe bash -lc "cd '/mnt/e/Intelligent Contracts_Project/Public Consultation Summary Fidelity Covenant' && uv run pytest -q"
```

Reviewed source hashes:

- Contract: `196CA2EB2CBA115B5B17400F1EC5E951D3E10842ECA7DC62E42D1308B387658F`
- Tests: `DB3C059316D20F5DA70D42A05B58C13C9F8B8F515908A4AE61FF3EDD5097D861`

## Studionet evidence

### Lifecycle and immutable source binding

For `scotland-fruit-veg-live-20260821`, create and all three public-response adds finalized successfully. The seal transaction [`0x3bb65b0c8e7c8dc419fa21ae632a304c0678b6ea888de59442e8f17e1d1befe7`](https://explorer-studio.genlayer.com/tx/0x3bb65b0c8e7c8dc419fa21ae632a304c0678b6ea888de59442e8f17e1d1befe7) finalized with execution `SUCCESS`; `read_manifest_hash` returned `0b10883c03cd4f5be498afaf0e471356277b5cde1c87255356dec6c3fca472ce`.

A separate parity scenario sealed a one-document manifest and then attempted another add. Its finalized receipt [`0x87d23efc266938abbc8952c7a087571be1978a7f78de52b9139dba2c841d0e27`](https://explorer-studio.genlayer.com/tx/0x87d23efc266938abbc8952c7a087571be1978a7f78de52b9139dba2c841d0e27) records leader and validator rollback `CONSULTATION_NOT_DRAFT`. This confirms the after-seal mutation guard on the deployed, schema-matched source.

### Public-source assessment and appeal replay

The public corpus assessment [`0xe11d2066475dd3cb69084e5496a6b00f9ac2c95d8a5f03abd386e97b5a6c07b4`](https://explorer-studio.genlayer.com/tx/0xe11d2066475dd3cb69084e5496a6b00f9ac2c95d8a5f03abd386e97b5a6c07b4) finalized with `UNRESOLVED`, `SOURCE_UNAVAILABLE`, revision `1`, and zero omission/distortion masks. An appeal was opened with a separately hash-bound, public consultation landing page and resolved in [`0xe2b34e4d3174d09b9320ddadb32abc8cb66a9d7846369cd7cce7d939caa05afd`](https://explorer-studio.genlayer.com/tx/0xe2b34e4d3174d09b9320ddadb32abc8cb66a9d7846369cd7cce7d939caa05afd). The final readback is `UNRESOLVED, 0, 0, 2`; history readbacks preserve both revision 1 and revision 2 with distinct evidence-manifest hashes.

The finality receipts use quorum consensus (three agree and two idle after quorum), not a claim of unanimous validator execution. `SOURCE_UNAVAILABLE` is a live environmental limitation in validator-side retrieval or canonical hash validation. It is deliberate fail-closed behaviour, not a substantive fidelity conclusion. The material-omission branch is fully covered by the synthetic direct test fixture, but not claimed as a live public-source result.

## License

[MIT](LICENSE)
