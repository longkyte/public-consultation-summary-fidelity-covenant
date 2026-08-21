# Implementation Specification

## 1. Scope and terminology

`Consultation` means one owner-created, bounded review record. Its sealed corpus is the complete input to one assessment; the contract never infers that it represents every response in an external consultation.

An official summary is a public summary or decision document supplied by its canonical HTTPS URL and SHA-256 digest of canonical visible text. A document is one public response with the same URL-and-digest pair. Canonical visible text removes HTML comments, `script` and `style` elements, remaining tags, then HTML-decodes and collapses whitespace before UTF-8 hashing. An appeal manifest is compact canonical JSON with 1–3 additional public documents, each with the same URL-and-digest pair. Appeal evidence can clarify the assessment but cannot add, remove, or substitute a sealed-corpus document.

| Bit | Code | Meaning |
| ---: | --- | --- |
| 1 | `MAJOR_THEME` | A material recurring theme is omitted or materially misrepresented. |
| 2 | `MINORITY_OBJECTION` | A substantive dissenting position is omitted or materially misrepresented. |
| 4 | `CONDITION` | A stated scope, qualification, or implementation condition is omitted or materially misrepresented. |
| 8 | `RISK` | A material process, implementation, or impact risk is omitted or materially misrepresented. |

Materiality is qualitative but constrained: an item must be explicit in a sealed document and relevant to the official summary's topic. The contract does not assess counts, demographic representativeness, policy merits, or legal compliance.

## 2. State and invariants

Persistent records use `@allow_storage @dataclass` and typed GenVM collections. `TreeMap` keys are deterministic consultation IDs and revision/index composites; `DynArray` keeps the ordered sealed URLs.

| Record | Immutable fields | Mutable fields |
| --- | --- | --- |
| `Consultation` | owner, title hash, criteria hash, document URL/digest list, sealed manifest hash, summary URL/digest | lifecycle status, latest revision, active appeal flag |
| `Assessment` | consultation ID, revision, fidelity, omission mask, distortion mask, confidence band, reason code, evidence manifest hash | none |
| `Appeal` | appellant, disputed issue mask, evidence manifest, opening revision | resolution revision |

1. Only `owner` may create, add documents, or seal; document addition is allowed only in `DRAFT`.
2. A consultation contains 1–12 unique canonical URLs and unique document digests before sealing.
3. `seal_consultation` stores the summary URL/digest and exactly one manifest hash. The corpus never changes afterwards.
4. Assessment revisions strictly increase. Existing assessments and history rows are append-only.
5. One active appeal is allowed. `open_appeal` is permissionless after assessment, but its disputed mask must be non-zero and use only the four defined bits. It is deliberately not limited to the prior assessment's issue mask, so a `FAITHFUL` assessment can be challenged with new public evidence.
6. `UNRESOLVED` is fail-closed: it has zero omission and distortion masks, `LOW` confidence, and a failure reason; consumers must not treat it as faithful.
7. `FAITHFUL` has both masks zero and reason `NONE`.
8. `MATERIAL_OMISSION` has a non-zero omission mask and zero distortion mask; `MATERIAL_DISTORTION` is the inverse; `BOTH` has both masks non-zero.
9. Decisive outcomes use only `MEDIUM` or `HIGH` confidence. The stored `issue_mask` is derived as `omission_mask | distortion_mask`, not redundantly stored.

## 3. Public API

```text
create_consultation(id, title_hash, criteria_hash)
add_document(id, url, document_hash)
seal_consultation(id, summary_url, summary_hash)
assess(id)
open_appeal(id, issue_mask, evidence_manifest)
resolve_appeal(id)
read_fidelity(id) -> (fidelity, omission_mask, distortion_mask, revision)
read_manifest_hash(id)
read_history(id, revision)
```

`id`, hashes, URLs, JSON payloads, and text fields have conservative, fixed maximum lengths. Before any non-deterministic call the contract deterministically rejects duplicate IDs, non-HTTPS URLs, credentials, fragments, literal local/private IP hosts, duplicate canonical URLs/digests, malformed JSON, unknown fields, invalid digest length, invalid masks, stale/duplicate appeals, invalid transitions, and unauthorized calls. DNS rebinding is outside this syntax check; authenticated, personalized, dynamic, or inaccessible sources are unsupported and must resolve as `UNRESOLVED`.

## 4. Consensus protocol

1. Copy all sealed primitives into memory before creating closures; no closure captures `self` or storage.
2. Leader and validator independently fetch the summary and every sealed response URL with `gl.nondet.web.get`, require a successful UTF-8 HTML response, canonicalize visible text exactly as above, cap each text block, and verify its SHA-256 against the registered digest. PDF, authenticated, personalized, or non-text sources are out of scope until the current SDK documents a safe text-extraction path.
3. Each invocation sends the same explicit criterion profile to the LLM. Source text is untrusted data: instructions inside it cannot alter the role, criteria, schema, or output.
4. The LLM returns only fidelity, omission mask, distortion mask, and confidence band. Optional echoed metadata is accepted only when it exactly matches trusted state. Contract code binds schema, consultation ID, revision, and derives the redundant reason code deterministically from fidelity.
5. The validator first requires `gl.vm.Return`, then validates trusted metadata, allowed keys, enums, mask range, and every cross-field invariant. It independently repeats retrieval and evaluation, then requires exact equality of every consequential bound field.
6. Controlled retrieval, digest, parse, and LLM failures normalize to the closed `UNRESOLVED` payload. The validator independently reruns the same work and accepts that payload only when its own closed result agrees; otherwise consensus rejects and state remains unchanged. After `gl.vm.run_nondet_unsafe` returns an accepted payload, deterministic code appends the assessment and changes lifecycle state. Partial coverage is never presented as a faithful decision.

`reason_code` allowlist: `NONE`, `OMISSION_DETECTED`, `DISTORTION_DETECTED`, `BOTH_DETECTED`, `SOURCE_UNAVAILABLE`, `MALFORMED_OR_AMBIGUOUS`.

## 5. Consensus Binding Matrix

| Field | Source | Stored? | Downstream effect | Validator check | Binding | Differential test |
| --- | --- | ---: | --- | --- | --- | --- |
| `fidelity` | independent corpus/summary evaluation | yes | primary oracle signal | exact enum plus independent recomputation | exact | same masks, changed fidelity rejects |
| `omission_mask` | independent issue classification | yes | review routing | exact bits and independent recomputation | exact | same fidelity, mask `1` vs `4` rejects |
| `distortion_mask` | independent issue classification | yes | escalation routing | exact bits and independent recomputation | exact | mask `1` vs `4` rejects |
| `confidence_band` | independent evidence sufficiency evaluation | yes | consumer filtering | exact enum and invariant check | exact | `MEDIUM` vs `HIGH` rejects |
| `reason_code` | deterministic fidelity/source-failure mapping | yes | explains fail-closed state | recomputed by contract | deterministic | wrong model echo is ignored; stored value follows fidelity |
| `issue_mask` | bitwise OR of accepted masks | no | consumer routing | never LLM supplied | deterministic | returned OR is verified from both masks |
| appeal disputed mask | appellant input | yes | scopes a later reassessment | deterministic 1–15 range check | deterministic input binding | zero or out-of-range mask rejects |
| revision/status/history | accepted revision and contract transition | yes | append-only auditability | never LLM supplied | deterministic | replay/failure leaves history unchanged |
| rationale | not accepted | no | none | extra key rejects | excluded | extra field rejects |

## 6. Test plan

Direct Mode uses strict `mock_web` and `mock_llm`, `check_pickling`, real captured validator callbacks, sender changes, and expected reverts. It covers:

- DRAFT-only document rules; owner authorization; 1–12 document bound; duplicate ID/URL/digest; invalid schemes and oversized input.
- Seal/assess transition rules and exact manifest hash readback.
- Faithful, omission, distortion, both, and unavailable-source paths.
- Malformed JSON, unknown enum, extra key, bad digest, invalid mask, and every cross-field mismatch.
- Prompt-injection text within a corpus document; LLM timeout/error; HTTP 404; one missing material document.
- Every binding-matrix differential case through `direct_vm.run_validator()`; agreement, disagreement, and no state mutation on rejection.
- Appeal availability, subset mask, stale/duplicate/replay rejection, old history retention, and exactly-one revision increment.

After Direct Mode passes, the PRE-DEPLOY package must include lint, typecheck, schema extraction, and a Studionet test/deployment matrix. No Studionet action is allowed until both PRE-DEPLOY verdicts approve the exact revision.

## 7. Acceptance criteria

The MVP is ready for PRE-DEPLOY only when a sealed corpus produces an exact-bound closed decision, each consequential field has a passing differential validator test, appeal history is append-only, and failure paths preserve state. A future Studionet release additionally requires a finalized/successful deployment, one consensus assessment, one deterministic negative transaction, authoritative state readback, and exact public Explorer evidence.

## 8. Deliberate exclusions

No frontend, archive service, policy enforcement, payment, identity system, demographic inference, exact confidence score, free-form rationale, automatic source discovery, or claim of democratic/legal legitimacy is in scope.
