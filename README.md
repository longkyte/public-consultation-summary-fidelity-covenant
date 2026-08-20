# Public Consultation Summary Fidelity Covenant

An Intelligent Contract that records a sealed consultation corpus and assesses
whether a public summary faithfully represents its material source documents.
It uses a bounded, evidence-first procedure: deterministic integrity checks
run before the nondeterministic assessment, and unresolved source retrieval or
validation fails closed rather than producing a favourable finding.

## Repository contents

- `contracts/public_consultation_summary_fidelity_covenant.py` — contract
  implementation.
- `tests/test_public_consultation_summary_fidelity_covenant.py` — unit and
  adversarial test suite.
- `docs/specification.md` — specification and assessment model.
- `samples/` — representative public-consultation fixtures.

## Local verification

The implementation was verified in an isolated WSL environment with Python
3.13.14 and these pinned tools:

- `genlayer-py==0.16.3`
- `genlayer-test==0.29.2`
- `genvm-linter==0.11.0`
- `pytest==9.1.1`

`uv pip check`, GenVM lint/check, type-checking, schema checks, and the test
suite completed successfully. The suite result was **46 passed**.

Reviewed implementation hashes:

- Contract: `196CA2EB2CBA115B5B17400F1EC5E951D3E10842ECA7DC62E42D1308B387658F`
- Tests: `DB3C059316D20F5DA70D42A05B58C13C9F8B8F515908A4AE61FF3EDD5097D861`

## Studionet evidence

The reviewed contract was deployed to GenLayer Studionet:

- Contract: [`0x6dBDcC298Fa2859b9Dc039eCEC15C0f5E17065dB`](https://explorer-studio.genlayer.com/address/0x6dBDcC298Fa2859b9Dc039eCEC15C0f5E17065dB)
- Deploy transaction: [`0x29c4821f7bfc8e1b9f1e31f2bb253c5d5c7ca6a8fc81624241046d6919378a40`](https://explorer-studio.genlayer.com/tx/0x29c4821f7bfc8e1b9f1e31f2bb253c5d5c7ca6a8fc81624241046d6919378a40)

The create, three document-recording, seal, and assessment transactions all
reached finality with GenVM execution `SUCCESS`. The assessment transaction
reached finalized, accepted quorum consensus (three validators agreed; two
were stopped after quorum):

- Assessment: [`0x48e0f6cc7e45b26190ae160cd60c905eecbaa1943a8e6ca0d0b9e5b5795dbca0`](https://explorer-studio.genlayer.com/tx/0x48e0f6cc7e45b26190ae160cd60c905eecbaa1943a8e6ca0d0b9e5b5795dbca0)

For the historical public-source manifest used in this live run, the finalized
readback was `UNRESOLVED`, `SOURCE_UNAVAILABLE`, revision `1`, with zero
omission and distortion masks. This is the intended fail-closed outcome: the
live retrieval could not establish the frozen source text needed to support a
substantive fidelity finding. It is not a claim that the published summary was
faithful or unfaithful.

## License

[MIT](LICENSE)
