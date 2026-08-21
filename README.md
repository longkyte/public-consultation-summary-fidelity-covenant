# Public Consultation Summary Fidelity Covenant

## Live deployment

The exact source in this repository is deployed on GenLayer Studionet at [`0x308ABE09600D5C18eB5935979CEe36706c15e3C0`](https://explorer-studio.genlayer.com/address/0x308ABE09600D5C18eB5935979CEe36706c15e3C0). Deployment transaction [`0xb1d635763952ecf617c7f4b2b55bb858995fe92fedd288e9a9dc9560077058af`](https://explorer-studio.genlayer.com/tx/0xb1d635763952ecf617c7f4b2b55bb858995fe92fedd288e9a9dc9560077058af) is `FINALIZED` with leader execution `SUCCESS`.

The contract seals a bounded set of public responses plus a public summary, verifies canonical visible-text hashes, and evaluates material omission or distortion. It does not measure representativeness, policy merit, legality, or democratic legitimacy. Retrieval, hash, malformed-output, and ambiguity failures are fail-closed as `UNRESOLVED`.

## Why GenLayer

Deterministic contracts can bind lifecycle, authority, URLs, hashes, masks, revisions, and immutable history, but cannot compare the meaning of a summary against natural-language responses. GenLayer supplies validator-executed web retrieval and LLM evaluation under consensus. The model supplies only fidelity, issue masks, and confidence; contract code binds schema, consultation identity, revision, and derives the redundant reason code.

## Lifecycle and API

`DRAFT → SEALED → ASSESSED → APPEALED → ASSESSED`. Only the owner can add documents or seal; appeals are permissionless after assessment and append exactly one new revision.

```text
create_consultation(id, title_hash, criteria_hash)
add_document(id, url, document_hash)
seal_consultation(id, summary_url, summary_hash)
assess(id)
open_appeal(id, issue_mask, evidence_manifest)
resolve_appeal(id)
read_fidelity(id)
read_manifest_hash(id)
read_history(id, revision)
```

See [docs/specification.md](docs/specification.md) for canonicalization, invariants, masks, consensus binding, prompt-injection controls, and limitations.

## Reproducible verification

```powershell
wsl.exe bash -lc "cd '/mnt/e/Intelligent Contracts_Project/Public Consultation Summary Fidelity Covenant' && uv run genvm-lint contracts/public_consultation_summary_fidelity_covenant.py && uv run pytest -q"
```

Final local result: GenVM lint passed and **49 tests passed**. Exact SHA-256:

- Contract: `104C609B4831E83C83B3B27990EC70D386C7DFDDBC9C649186A0DE7D600799DF`
- Tests: `2EEEABF3DA1D599E9EA9337F0DE7FE230EAF09DAC33A9C89E54C517CA3E6817D`

## Studionet E2E evidence

The controlled public corpus is transparently published at [longkyte/pcsfc-e2e-public-corpus](https://github.com/longkyte/pcsfc-e2e-public-corpus), Git revision `609b777fc47ee96eda941c6f1da223568846d1e3`. It is a stable E2E fixture, not a real consultation or public-opinion claim.

### Faithful case

Case `pcsfc-v9-faithful-pages-20260821`: create [`0xadc6e52b…`](https://explorer-studio.genlayer.com/tx/0xadc6e52b2686bbf12ab64d7f07db1918948c7073bf31a86bbaef91a72a4807b9), add [`0xb45c9552…`](https://explorer-studio.genlayer.com/tx/0xb45c95524a2f5e67434634cc0c7902baf208e444eaa48946050474cd6c55a4b9) and [`0x66143fe3…`](https://explorer-studio.genlayer.com/tx/0x66143fe3d74ad4b38ebdb5a8a43b5b65ebaeff8e46b183feed33692576943e71), seal [`0xc8a34972…`](https://explorer-studio.genlayer.com/tx/0xc8a3497287048a7fa117cf9a37dac9f822c4faaf35fc4b3ea306d17849542709), assess [`0x9c5fc3e9…`](https://explorer-studio.genlayer.com/tx/0x9c5fc3e9b107e43b358ed50cb2094bb864f525a8007e84b72059f3a662e474e3). Every transaction is `FINALIZED/SUCCESS`. Readback is `FAITHFUL, 0, 0, revision 1`; manifest hash is `6836c11ca5bae8e968091ac4528340d9a8a91cdc9d68567cd3ad16ca1638e661`.

### Material-omission and appeal case

Case `pcsfc-v9-omission-pages-20260821`: create [`0x0663553a…`](https://explorer-studio.genlayer.com/tx/0x0663553a7e5c10def637865233341a553a0c451e2d18f9aa42fac67a2fd0af77), add [`0x2aa24daf…`](https://explorer-studio.genlayer.com/tx/0x2aa24daf5f3e5bed7618a1107e23a0e9bf0068d3082cf6987267538a95874870) and [`0x2bcfeb81…`](https://explorer-studio.genlayer.com/tx/0x2bcfeb8123c4b775473ee0f154468793840caff9d1d8fc37b6ec474865f14d5b), seal [`0xb9c0aa41…`](https://explorer-studio.genlayer.com/tx/0xb9c0aa41f7143b207801cd72002960117699eea0ce7905a13201fe890592d04e), assess [`0x46064533…`](https://explorer-studio.genlayer.com/tx/0x460645338933f2190e4db6ccbdb89b38c2433b65c5145932bd2c55f80867e306), open appeal [`0x8cd1f506…`](https://explorer-studio.genlayer.com/tx/0x8cd1f506a71d474e70787ce1dc2c3208935bb371f49713ecd8beef9d125f4e9c), resolve [`0xb1230b23…`](https://explorer-studio.genlayer.com/tx/0xb1230b23ab9c13a3185ae378e6782e20e1d0f8f5cbf2bde02d09a867fcb40455). All are `FINALIZED/SUCCESS`.

Revision 1 and revision 2 both read `MATERIAL_OMISSION`, omission mask `12`, distortion mask `0`, `HIGH`, `OMISSION_DETECTED`. Distinct evidence hashes `4f53cda18c2baa0c0354bb5f9a3ecbe5ed12ab4d8e11ba873c2f11161202b945` and `8cbc162b294763ef171b01c66c868cbd8c1b7f5aba7385ca10dd93776e078ded` prove append-only appeal history.

### Negative state guards

Add-after-seal [`0x352b14f7…`](https://explorer-studio.genlayer.com/tx/0x352b14f7e96065db5e83bd714a0066faa783e58a6a2d348f8e09b37df86b15ba) finalized `ERROR / CONSULTATION_NOT_DRAFT`; replay assess [`0xe0487150…`](https://explorer-studio.genlayer.com/tx/0xe04871505b3e9df4b4f82359e1ea09f8f79418ef93951e65979cef175ca72dc5) finalized `ERROR / CONSULTATION_NOT_SEALED`. Readback remained `FAITHFUL, 0, 0, revision 1`.

## Limitations and integrations

Public HTTPS HTML must remain retrievable and hash-stable. PDFs, authenticated/personalized pages, DNS rebinding protection, archival availability, frontend UX, identity, payments, policy enforcement, demographic inference, and legal/democratic conclusions are out of scope. Integrators should pin source snapshots, treat `UNRESOLVED` as non-favourable, inspect revision history, and use masks only as review-routing signals.

## License

[MIT](LICENSE)
