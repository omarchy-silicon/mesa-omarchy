# Omarchy Silicon Mesa/AGX downstream design

Status: DESIGN NOTE, correction round 2. G-01 through G-05 are BLOCKED or TODO; nothing in this document is DONE, implemented, built, qualified, compatible, supported, or release-ready.

Owner: Mesa/AGX downstream design lane (DESIGN-SWE)

Repository: `omarchy-silicon/mesa-omarchy`

Branch: `factory/design-mesa-release`

Frozen source evidence base: the review snapshot of freedesktop.org Mesa `main` and the owned mirror `main` was `d870cef8b7c8a4a11edc669669c9f18ae402314a` on 2026-09-02 (`VERSION` file reads `26.3.0-devel`). The live authoritative ref is now `c0682c54768a5e247c6627fc3b632eb81b7af2a4` while the live mirror remains at `d870cef8b7c8a4a11edc669669c9f18ae402314a`; this is a provenance HOLD, not PASS. Every `src/`, `include/`, `subprojects/`, and `meson.options` citation below refers to the frozen review tree and was read without modification.

This document is a design contract for the Mesa component of the Omarchy Silicon platform program. It does not modify Mesa graphics code, establish hardware support, claim conformance, or promote any board or release. A build, recognized GPU, booting compositor, or passing subset of tests is evidence for a gate only; it is never a support claim by itself.

## 0. Correction status and reading rules

This revision answers the coordinator gate on tip `6438517fd9f91c2ae0b297ea8acb00782c9034ee` (ten blocking findings, hostile census 0 PASS / 79 NOT EXECUTABLE). It is a design-only correction. No schema, validator, fixture, generated binding, consumer guard, CI job, build directory, package, or physical evidence was created by this change.

The authoritative program input is `PROGRAM.md` in `omarchy-apple-platform` at ratified commit `58302d148f0e8b855578f9aa518ff1c5eb48c515`. The only F-02 text available to this lane is the rejected candidate at `omarchy-apple-platform` commit `c315c7e79928d0041deb582bed79a61074361b21`, file `docs/design/platform-schema.md`. PROGRAM section 16 and its decision log freeze that tip after three rejected correction rounds pending an owner exception. Therefore:

- Every `Trusted<T>`, `AuthorityRoleBinding`, `ExpectedContext`, JSON path, failure code, grammar name, and role name cited below is marked PROVISIONAL. It is a dependency on a future ratified F-02/F-03 contract, not a statement that the rejected candidate is authority.
- This lane does not copy, restate, or extend that candidate's schema text as a local schema. Where this lane needs something the candidate lacks, it records an F-02-owned extension requirement and marks the consuming gate BLOCKED.
- If ratified F-02/F-03 rename or reshape any cited item, the generated bindings are the only source of the new names; this document is then corrected in a new round. No Mesa code may be written against the provisional names.
- F-02/F-03 consumption is permitted only after the future ratification gate in section 3.3 passes: the generated binding identity, version, output digest, constructor boundary, authority context, role binding, schema-set digest, clock, replay reservation, and anti-transplant checks must all match the consumer lock. A name in this document is not canonical merely because it is precise.
- The opaque human-produced boot artifact boundary named in PROGRAM section 6 is not inspected, characterized, or depended on here. Boot facts reach this lane only as coordinator-supplied signed opaque envelope identities inside a verified platform manifest.

Two-space indentation and full Markdown lines are used throughout, and no field is left to be filled in later. Ledger vocabulary follows PROGRAM section 12: TODO, IN PROGRESS, BLOCKED, HUMAN-ONLY BLOCKED, DONE. Only the coordinator changes a slice to DONE.

## 1. Authority, ownership, and boundaries

The authoritative Mesa history is freedesktop.org Mesa at `https://gitlab.freedesktop.org/mesa/mesa`. `https://github.com/omarchy-silicon/mesa-omarchy` is the Omarchy-owned mirror and downstream integration point. The GitHub repository may be a standalone owned mirror rather than a GitHub-native fork; GitHub ownership, repository URL, default branch, and upstream history must all be checked explicitly.

`origin/main` is the pure mirror branch. It must represent the exact authoritative Mesa `main` tip after each accepted synchronization. Omarchy changes are carried on named downstream branches or release refs above a recorded upstream base. A downstream commit must never be silently placed on the mirror branch.

The canonical `omarchy-apple-platform` repository owns `board-registry/v1`, `platform-manifest/v1`, `qualification-record/v1`, and the other five authenticated payload types, the schema set, generated bindings, trust context (F-03), builder definitions (F-04), candidate assembly (F-05), release compliance (F-06), and the sole stable-promotion terminal (F-07). This repository consumes generated bindings from those interfaces and records references to them. It must not publish a second board map, capability vocabulary, AGX profile authority, qualification schema, signing authority, promotion record type, or release authority.

The kernel and firmware are external component boundaries owned by `linux-omarchy` (K-01 through K-06) and the firmware bundle component of the platform manifest. Mesa declares the exact kernel/DTB/firmware tuple it was built and qualified against; it does not infer compatibility from a product name, SoC family, `uname`, device node presence, or a successful probe.

Mesa emits candidate artifacts and evidence only. It cannot write, sign, copy into, or advertise a stable channel. F-07 is the sole stable-promotion writer (PROGRAM sections 11, 12.1, 13, 17).

## 2. Design principles

- Preserve upstream Mesa history and keep the Omarchy patch queue minimal, reviewable, and removable.
- Fail closed on mirror drift, source ambiguity, ABI mismatch, unknown or unbounded GPU generation, virtual device, profile mismatch, missing evidence, signature failure, replay, expiry, or non-reproducible output.
- Tie every result to exact typed identities: source, patch queue, build recipe, kernel/DTB/firmware tuple, board, AGX profile, artifact, and evidence. Never let one identity type stand in for another.
- Keep generation support board-specific. A shared SoC label, chip family, or `gpu_generation` value is diagnostic metadata, not qualification.
- Separate source synchronization, provenance, build, ABI, conformance, compositor, physical, packaging, candidate, and promotion gates.
- Promote only an immutable artifact digest, and only through F-07. Mesa never promotes.
- Keep the last-known-good complete platform tuple available for rollback for the supported lifetime of each board.
- Report failures, exclusions, and residuals alongside passing results. No warning may be converted into a success state. Software fallback never proves hardware capability.

## 3. Trusted consumer seam

### 3.1 What a consumer is in this lane

Two consumers exist. The candidate builder runs in the F-04 builder and consumes the predecessor manifest, registry, qualification records, and trust context to assemble Mesa candidate artifacts and evidence. The renderer admission guard runs on the target machine inside the platform update and boot-health tooling before the graphics stack is selected, using the generated Python binding (F-02 handoff target) or, once ratified, a bounded generated constants input compiled into the downstream Mesa build (section 5.4). Mesa driver code itself never parses, verifies, or trusts signed metadata, and never holds a key.

Both consumers accept board, manifest, qualification, trust-context, and evidence inputs only as values produced by the ratified F-02/F-03 verification seam. There is no Mesa-local identity, digest alias, signing key, trust flag, environment override, or unchecked constructor. A parsed, projected, cached, caller-supplied, or partially verified value has no authority.

### 3.2 Trusted input paths

| ID | Trusted path (PROVISIONAL name) | Produced by | Exact consumer use | Status |
| --- | --- | --- | --- | --- |
| TP-01 | `Trusted<BoardRegistry>` | F-02 `verify` with F-03 `Trusted<TrustContext>`, signer role `board-admission` | Resolve the admitted `board_id`, `identity_match.macos.chip_id_u32`, `soc`, `firmware`, `physical_capabilities` for `gpu`, `media`, `internal-display`, `external-display`, `qualification_profile`, `lifecycle` | PROVISIONAL, pending F-02 ratification |
| TP-02 | `Trusted<PlatformManifest>` | F-02 `verify`, signer role `manifest-release` | Read `$.payload.components.mesa_stack`, `components.linux_kernel`, `components.dtb_set`, `components.firmware_bundle`, `components.boot_stack` (opaque digests only), `board_targets`, `qualification_bindings`, `channel`, `consumer_schema_set`, `rollback`; verify all five projections equal the component tree | PROVISIONAL, pending F-02 ratification |
| TP-03 | `Trusted<QualificationRecord>` (one per manifest binding) | F-02 `verify`, signer role `qualification-lab` | Verify `manifest.manifest_id` and `manifest.manifest_digest` equal the trusted manifest; verify the board is a manifest target; read `test_results`, `evidence`, `residuals`, `outcome` for the AGX profile rows | PROVISIONAL, pending F-02 ratification |
| TP-04 | `Trusted<TrustContext>` | F-03 signed trust-root bundle | Supplies `authority_bindings`, `key_set_digest`, `revocation_epoch`, `expires_at`; nothing else can resolve a role | PROVISIONAL, pending F-03 |
| TP-05 | `AuthorityRoleBinding` inside TP-04 | F-03 | Roles consumed: `manifest-release`, `board-admission`, `qualification-lab`, `ci-conformance` (signer of Mesa build/ABI/conformance reports), `evidence-reader`; a role string, key ID, or account without a binding resolves nothing | PROVISIONAL, pending F-03 |
| TP-06 | `ExpectedContext` | Typed constructor in the consumer, from generated constants only | Fixed per payload: `payload_type`, `domain`, `context`, `operation = inspect/v1`, `project_id`, `repository_id`, `slice_id`, `board_id`, `manifest_id`, `manifest_digest`, `schema_set_digest`; target-account fields are `not-applicable` for these read-only payloads | PROVISIONAL, pending F-02 ratification |
| TP-07 | `VerifiedClock` | F-02 `verify_clock` over an attested monotonic sample | Expiry and replay decisions for every payload; a caller wall-clock value is rejected | PROVISIONAL, pending F-02 ratification |
| TP-08 | `Trusted<Observation>` | F-02 `make_trusted_observation` from `linux-sysfs/v1` source evidence on the target | Board identity on the running machine, compared with TP-01 predicates and with the DRM device parameters (section 5.4) before renderer selection | PROVISIONAL, pending F-02 ratification |
| TP-09 | `Admitted<Policy>` | F-02 `admit_policy` from `Trusted<PolicySource>` | AGX profile disposition policy, rollback policy, and required-gate policy; no consumer-local policy map | PROVISIONAL, pending F-02 ratification |
| TP-10 | `ConsumerCapabilities` | F-02 `load_consumer_capabilities(Trusted<CompiledLock>, Trusted<GeneratedOutputLock>, Trusted<GeneratedBindingMetadata>)` | Exact schema-set, binding, parser, API, and output-digest negotiation against `$.payload.consumer_schema_set` before any payload is read | PROVISIONAL, pending F-02 ratification |
| TP-11 | `Trusted<EvidenceReadAuthorization>` | F-02 evidence service, role `evidence-reader` | Reading raw build logs, traces, captures, and reports referenced by `evidence_ids`; bytes are accepted only after `expected_content_digest` matches | PROVISIONAL, pending F-02 ratification |
| TP-12 | Generated AGX profile binding | F-02-owned generator output from the ratified AGX profile extension (section 5) | Exact chip, generation, feature, and property disposition for the admitted board; consumed as generated constants, never as a Mesa-local table | BLOCKED, extension not ratified |
| TP-13 | Opaque boot artifact identities | Coordinator-supplied signed envelope, visible only as `components.boot_stack.artifacts[*].content_digest` inside TP-02 | Membership in the ABI tuple and rollback set; never opened, parsed, or characterized by this lane | PROVISIONAL, human-owned producer |

Thirteen trusted paths are defined. Anything not in this table has no authority in this lane.

### 3.3 Future-ratified F-02/F-03 consumption gate

All names in this section are PROVISIONAL and are requirements for a future ratified interface, not authority derived from the rejected F-02 candidate. A consumer may consume one payload only when the following closed gate has passed in order: G0 loads the generated binding metadata; G1 authenticates the F-03 trust context; G2 checks the generated binding identity and version; G3 checks the generated output digest and schema-set digest; G4 constructs `ExpectedContext` from generated constants; G5 verifies the envelope and signature role; G6 recomputes the payload digest and rejects duplicate keys or unknown members; G7 checks expiry against `VerifiedClock`; G8 reserves and checks the replay identity; G9 checks the anti-transplant tuple and every cross-document binding; and G10 returns a `Trusted<T>` value through the sole typed constructor. No field is read for admission before its phase has passed.

The generated binding handoff is a typed artifact with `binding_id`, `binding_version`, `generator_id`, `generator_version`, `source_schema_set_digest`, `output_digest`, `language`, and `artifact_content_digest`. Its exact preimage is `JCS({binding_id,binding_version,generator_id,generator_version,source_schema_set_digest,language,artifact_content_digest,output_digest})`; the output digest is `sha256(LF-normalized generated bytes)`. The consumer lock compares every member and the binding artifact bytes. A changed binding identity, version, source schema-set digest, generator identity, or output digest is a binding failure even when parsing still succeeds.

The only constructor boundary is the future generated `Trusted<T>::from_verified_envelope(VerifiedEnvelope<T>, ExpectedContext, VerifiedClock, ReplayReservation)`; it is private to the verifier package, cannot be called with raw JSON, a parsed value, a digest alias, a role string, an unchecked cast, a caller policy, or a boolean trust flag, and returns no value on failure. `ExpectedContext` is generated and exact for `payload_type`, `payload_version`, `domain`, `context`, `operation`, `project_id`, `repository_id`, `slice_id`, `board_id`, `manifest_id`, `manifest_digest`, and `schema_set_digest`. TP-05 resolves the signer role and key through the authenticated F-03 `AuthorityRoleBinding`; a role name or key ID alone has no authority. Expiry uses TP-07, replay uses a durable reservation keyed by the authenticated replay identity, and transplant protection recomputes the closed tuple `(document_id,schema,payload_type,payload_version,schema_set_digest,domain,context,project_id,repository_id,slice_id,board_id,manifest_id,manifest_digest)` before construction.

The gate is not implemented or canonical at this tip. It becomes consumable only when F-02 and F-03 publish the generated binding identity/version/digest, ratified constructor API, closed schema-set vocabulary, role bindings, clock and replay service, positive fixtures, and the exact rejection interface, and the coordinator records that ratification. Until then TP-01 through TP-11 remain PROVISIONAL, TP-12 remains BLOCKED, and every dependent G-01 gate remains BLOCKED.

### 3.4 Closed Mesa-facing error interface

The future-ratified Mesa-facing error record is exactly `{code,phase,path,decision,process_result}`. `code` is one stable member of the closed code registry in section 20 and has one meaning; alternatives, broad classes, free text, or an outcome without a code are invalid. `phase` is one of the following total order values: `P00_PARSE`, `P01_BINDING`, `P02_TRUST_CONTEXT`, `P03_SCHEMA`, `P04_SIGNATURE`, `P05_CLOCK`, `P06_REPLAY`, `P07_CROSS_DOCUMENT`, `P08_SOURCE`, `P09_RECIPE`, `P10_ABI`, `P11_EVIDENCE`, `P12_QUALIFICATION`, `P13_CANDIDATE`, `P14_PROMOTION`, `P15_ADMISSION`, `P16_HEALTH_ROLLBACK`. `path` is an exact JSON Pointer for an authenticated document or an exact `$meta.*`, `$build.*`, `$cache.*`, `$runtime.*`, `$qualification.*`, or `$promotion.*` path for a non-payload record. `decision` is exactly one of `REJECT`, `HOLD`, `RECORD_FAIL`, or `NO_PUBLISH`. `process_result` is exactly `exit=78` for a rejected process, `exit=79` for a held process, `record=FAIL;exit=78` for a failed qualification record, or `publish=NONE;exit=78` for a promotion refusal.

The first fault is deterministic: choose the lowest phase, then the lowest canonical field ordinal from the ratified grammar, then the lexicographically lowest exact path, then the lowest code. Canonical field ordinals are defined by the generated binding and are independent of input object member order; JCS normalization is performed before checking them, and semantic array order is never replaced by arrival order. Thus simultaneous faults produce the same code, phase, path, decision, and process result under every permutation of object members. A consumer emits one error record, does not continue to a later phase, and never downgrades the decision to a warning or success.

### 3.5 Deterministic consumer behavior

| Consumer | Accepted input | Error behavior | Success behavior |
| --- | --- | --- | --- |
| F-04 candidate builder | Only `Trusted<T>` values from the future gate and the exact BuildRecipe/v1 input | Emit the section 20 code record, exit 78 or 79 as specified, emit no candidate artifact, and retain no partially trusted value | Continue only after the complete tuple, recipe, evidence, and package closure passes |
| Renderer admission guard | Only `Trusted<PlatformManifest>`, `Trusted<BoardRegistry>`, `Trusted<Observation>`, the generated profile binding, and the closed ABI tuple | Emit the first code record, exit 78, keep the last-known-good tuple active, select no AGX renderer, and expose only the redacted code/path | Select the exact profile-bound renderer only after every comparison passes |
| F-07 promotion terminal | Only a future ratified F-06 compliance bundle, F-05 candidate, F-07 transaction record, and complete tuple | Emit the first code record, exit 78, publish no stable marker, and preserve recovery state | Atomically publish the exact candidate digest and append the public-ledger record |

Every failure is fail-closed. There is no warning-success, degraded mode, software-renderer substitution, relaxed retry, or stale-cache reuse. The gate is specified only; no consumer, interface, fixture, or result is implemented by this document.

## 4. Typed identities and digests

Every identity below has one type, one producer, one recomputable preimage where it is a digest, and one path family. A value of one type is never accepted where another type is expected, even when both are `sha256:` strings or both are 40-character hexadecimal. Comparisons are typed equality; no consumer compares a document ID with a digest, a commit with a tree, or a patch digest with an artifact digest.

| ID | Identity or digest type | Authority and preimage (PROVISIONAL where F-02/F-03) | Exact path or record | Substitution prohibited with |
| --- | --- | --- | --- | --- |
| ID-01 | `document_id` (any payload) | F-02; stable lowercase token, immutable one-ID-to-one-payload-digest history; never a content digest | `$.payload.document_id`, `DocumentIdRecord`, `DocumentIdLineage` | ID-02, ID-04, ID-05 |
| ID-02 | `payload_digest` | F-02; `sha256(JCS(P))` over the closed payload | signature preimage `A.payload_digest` | ID-01, ID-03 |
| ID-03 | `schema_set_digest` | F-02; SHA-256 over the ASCII domain string `omarchy-schema-set/v1`, one zero byte, and the JCS bytes of the schema input lock | `$.schema_set_digest`, `$.payload.schema_set_digest`, `consumer_schema_set.schema_set_digest`, both locks | ID-02, ID-20 |
| ID-04 | `manifest_id` | F-02; the `document_id` of a `platform-manifest/v1` payload | `QualificationManifestBinding.manifest_id`, `ExpectedContext.manifest_id`, `rollback.previous_manifest_ids[*]` | ID-05, ID-23 |
| ID-05 | manifest payload digest (`manifest_digest`) | F-02; ID-02 computed over the manifest payload | `QualificationManifestBinding.manifest_digest`, `ExpectedContext.manifest_digest`, `PlanSelection.manifest_digest` | ID-04, ID-24 |
| ID-06 | board registry document ID | F-02; ID-01 for `board-registry/v1` | `$.payload.document_id` of the registry | ID-07 |
| ID-07 | `board_registry_digest` | F-02; ID-02 of the registry payload | `$.payload.board_registry_digest` in the manifest, `QualificationManifestBinding.board_registry_digest` | ID-06, ID-05 |
| ID-08 | source repository identity | F-02 `RepositoryId` plus the immutable HTTPS URL recorded in the provenance record (section 7.1); never a branch | `components.mesa_stack.source.repository_id`, provenance `upstream_url` | ID-09A, ID-09B, ID-10 |
| ID-09A | Git tag or ref object identity | Exact Git object ID of the fetched object with Git object type `tag` for an annotated tag or `commit` for an explicitly permitted signed ref; preimage is the canonical Git object bytes including the object header; authority is the upstream signer or the explicitly ratified F-03 attestation authority | provenance `tag_object_id` or `ref_object_id`, `object_type`, `signer_evidence` | ID-09B, ID-10, ID-11 |
| ID-09B | ref-advertisement digest | `sha256(CONCAT(ASCII("omarchy-ref-advertisement/v1"),0x00,JCS(sorted({remote,ref_name,object_id,peeled_object_id,fetch_clock}))))`; owner is the synchronization job; authority is the signer/attestor named in the provenance record | provenance `ref_advertisement_digest` and its `fetch_clock` | ID-09A, ID-10, ID-11 |
| ID-10 | peeled commit identity (`git_commit`) | Git commit object ID, 40 or 64 lowercase hexadecimal | `components.mesa_stack.source.upstream_commit` (verified mirror base), `source.source_commit` (queue tip) | ID-09A, ID-09B, ID-11, ID-12 |
| ID-11 | tree identity | Git tree object ID of the commit named in ID-10 | provenance `upstream_tree_id`, `queue_tip_tree_id` | ID-10, ID-13 |
| ID-12 | patch queue digest (`patch_lock.lock_digest`) | F-02 lock digest over the sorted `PatchEntry` list; each entry has `patch_id`, `source_digest`, `patch_digest`, `order` | `components.mesa_stack.patch_lock` | ID-10, ID-13, ID-15 |
| ID-13 | generated-source input digest | F-02 `ConfigInput.source_digest` and `normalized_content_digest` for every generator input (Mako templates, XML, Python generator sources) | `components.mesa_stack.config_inputs[*]` | ID-14 |
| ID-14 | generated-source output digest | Digest over LF-normalized generated output bytes, recorded in the recipe transcript and compared across two builders | recipe transcript entry per generated file; `report_lock` entry of kind `build/v1` | ID-13, ID-15 |
| ID-15 | artifact content digest | F-02 `ComponentArtifact.content_digest` over the exact artifact bytes | `components.mesa_stack.artifacts[*].content_digest`, `PackageRecord.content_digest` | ID-12, ID-14, ID-16 |
| ID-16 | signature identity | F-03 key custody; the closed tuple `(key_id, signer_role, algorithm, signature_format)` plus the detached signature bytes for artifacts | `$.signatures[i]`, artifact signature policy `signature_policy_id` | ID-15, ID-17 |
| ID-17 | ABI contract ID | F-02 `abi_contract_id` on the component and `TypedCompatibilityRelation.contract_id` | `components.linux_kernel.abi_contract_id`, `components.mesa_stack.abi_contract_id`, relation `kernel-abi/v1` | ID-18 |
| ID-18 | ABI contract digest | `TypedCompatibilityRelation.evidence_digest` over the closed ABI report (section 8.3) produced independently by K-01 tooling and the Mesa builder | `components.linux_kernel.compatibility_relations[*].evidence_digest` | ID-17, ID-15 |
| ID-19 | qualification record ID | F-02; ID-01 for `qualification-record/v1` | `$.payload.qualification_bindings[*].qualification_record_id` | ID-20 |
| ID-20 | qualification record digest | F-02; ID-02 of the qualification payload; appears only on the record side and in F-07 closure, never inside a manifest | signature preimage of the record; F-07 closure input | ID-19, ID-05 |
| ID-21 | recipe digest | F-04; digest over the closed recipe object (section 9.1) | `components.mesa_stack.recipe_digest` | ID-22 |
| ID-22 | toolchain lock digest | F-02 `toolchain_lock.lock_digest` over sorted `ToolchainEntry` records | `components.mesa_stack.toolchain_lock` | ID-21, ID-12 |
| ID-23 | candidate ID | F-05; the `document_id` of the F-05-assembled manifest at channel `edge` or `rc`. Mesa has no local candidate ID; Mesa outputs are identified only by ID-15 and report digests until F-05 assembles them | F-05 assembler output | ID-04 of a stable manifest, ID-24 |
| ID-24 | candidate digest | F-05; the payload digest of that manifest, which F-07 copies unchanged into `stable` | F-07 promotion input | ID-23, ID-05 of a different manifest |
| ID-25 | AGX profile ID and digest | F-02-owned extension (section 5); profile identity and `sha256(JCS(profile))` over the closed profile record | `$.payload.gpu_profile.profile_id`, `$.payload.gpu_profile.profile_digest` | ID-07, ID-17 |
| ID-26 | generated binding identity/version/digest | F-02 generator; `sha256(JCS({binding_id,binding_version,generator_id,generator_version,source_schema_set_digest,language,artifact_content_digest,output_digest}))` with output digest over LF-normalized generated bytes | `consumer_schema_set.generated_binding`, TP-10, TP-12 | ID-03, ID-13, ID-25 |

Twenty-six identity types are defined. Recomputation rule: any digest in this table is recomputed by the consumer from its named preimage bytes; a digest that cannot be recomputed because the preimage is absent, mutable, or network-only is treated as mismatched. A mutable branch, tag name, URL, or ref locator is never an identity and never substitutes for ID-09A, ID-09B, or ID-10.

## 5. AGX profile coverage without a shadow authority

### 5.1 Frozen source evidence

The current Mesa tree decides GPU behavior from kernel-reported parameters without any board, profile, or upper-bound input:

- `include/drm-uapi/asahi_drm.h` defines `DRM_ASAHI_GET_PARAMS = 0` and `struct drm_asahi_params_global` with exactly `features`, `gpu_generation`, `gpu_variant`, `gpu_revision`, `chip_id`, `num_dies`, `num_clusters_total`, `num_cores_per_cluster`, `max_frequency_khz`, `core_masks[64]`, `vm_start`, `vm_end`, `vm_kernel_min_size`, `max_commands_per_submission`, `max_attachments`, and `command_timestamp_frequency_hz`.
- `src/asahi/lib/agx_device.c:662-670` maps `gpu_generation >= 14` with more than one cluster to `AGX_CHIP_G14X`, any other `>= 14` to `AGX_CHIP_G14G`, `>= 13` with more than one cluster to `AGX_CHIP_G13X`, and everything else to `AGX_CHIP_G13G`. There is no upper bound; a generation 15, 20, or 99 device is silently treated as generation 14.
- `src/asahi/lib/agx_device.c:544` asserts `gpu_generation >= 13` only in assertion-enabled builds; release builds have no lower bound either.
- `src/asahi/lib/agx_device.c:655-659` selects the `libagx_g13x` or `libagx_g13g` program set from a coherency tristate, not from a profile.
- `src/asahi/lib/decode.c:924-959` (`chip_id_to_params`) enumerates exactly eight chip IDs: `0x6000`, `0x6001`, `0x6002` (generation 13, variants S/C/D, clusters `2 << (chip_id & 15)`), `0x6020`, `0x6021`, `0x6022` (generation 14, same variant rule), `0x8112` (generation 14, variant G, one cluster), and `0x8103` (generation 13, variant G, one cluster). `case 0x8103:` shares the `default:` arm, so every unknown chip ID is decoded as a generation 13 G part. `decode.c:974` also calls `chip_id_to_params(&params, 0x8103)` as a default.
- `src/asahi/vulkan/hk_physical_device.c:234-270` (`hk_get_device_features`) sets Vulkan 1.0 core features to constants; only `vertexPipelineStoresAndAtomics` reads a driconf option. `hk_get_device_extensions` at lines 48-232 is likewise largely unconditional. No board or profile input exists.
- `src/asahi/lib/agx_device.c:519-527` accepts a DRM driver named `virtio_gpu` as an Asahi native context; `agx_device.c:46-47` and `313-317` then route ioctls and handles through the virtio path, implemented in `src/asahi/lib/agx_device_virtio.c`. The coordinator's cited range `agx_device.c:1222-1229` does not exist in this tree, which has 960 lines; the second acceptance site is recorded here as lines 46-47 and 313-317.
- `src/asahi/lib/agx_device.c:895-901` derives the device UUID from `gpu_generation`, `gpu_variant`, and `gpu_revision` only; two boards with the same SoC are indistinguishable at this layer.

These observations are hostile surfaces, not implementation work. They remain open residuals RES-01 through RES-04 in section 21.

### 5.2 F-02-owned AGX profile extension requirement

The board registry candidate carries `identity_match.macos.chip_id_u32`, `soc`, and the capability vocabulary members `gpu`, `media`, `internal-display`, and `external-display`, but no GPU property disposition. This lane therefore records the following extension requirement for the F-02 owner. The extension is owned, versioned, signed, and generated by `omarchy-apple-platform`; this repository will consume only its generated binding (TP-12). Nothing below is a Mesa-local schema.

| ID | Requirement for the ratified AGX profile extension | Fail-closed rule |
| --- | --- | --- |
| REQ-AGX-01 | Exactly one profile per registered board with `gpu` capability `present`; the board record binds the profile by ID and digest (ID-25); a profile is never keyed by SoC, family, or `gpu_generation` alone | Board without a bound profile is quarantined |
| REQ-AGX-02 | Exact expected values for every `drm_asahi_params_global` identity field: `gpu_generation`, `gpu_variant`, `chip_id`, `num_dies`, `num_clusters_total`, `num_cores_per_cluster`, and a closed inclusive set for `gpu_revision`; `chip_id` must equal the registry `chip_id_u32` | Any field not equal, or `gpu_revision` outside the closed set, quarantines |
| REQ-AGX-03 | Explicit generation bounds carried by the extension: `min_gpu_generation = 13` and `max_gpu_generation` equal to the highest generation of any ratified profile (14 on the frozen evidence); the bound moves only by ratifying a new profile, never by a code path | Generation outside the bounds quarantines even when a profile exists elsewhere |
| REQ-AGX-04 | Exact chip ID allowlist as a closed list; the frozen tree's eight IDs are evidence of what the source can decode, not an allowlist; the ratified list is board-derived through Q-00 | Chip ID not in the list, or in the list but bound to a different board, quarantines |
| REQ-AGX-05 | Virtual-device policy: DRM driver name must equal `asahi`; `virtio_gpu` and every other name is `virtual-or-foreign` and quarantined for release consumers; a separately labeled non-qualifying test profile may exist but can never produce qualification evidence or a candidate | Virtual device quarantines |
| REQ-AGX-06 | Per-feature disposition, one row per feature, each `required`, `optional`, or `unsupported`, for compiler (`libagx` program set, coherency mode, instruction capabilities), decode tooling (`agxdecode` parameter table), OpenGL/GLES version and required extensions, Vulkan version, extension list, and each `vk_features` member, compute (Vulkan compute and any OpenCL exposure), media (`video-codecs` and `gallium-va` disposition), and display/WSI (`platforms`, KMS/DRM presentation, internal and external display topology classes from the registry) | A feature advertised by the driver but `unsupported` in the profile, or `required` in the profile but not advertised, quarantines |
| REQ-AGX-07 | Per-property disposition for memory (`vm_start`, `vm_end`, `vm_kernel_min_size`), submission (`max_commands_per_submission`, `max_attachments`), clocks (`max_frequency_khz`, `command_timestamp_frequency_hz`), and the `features` bit set, each with an exact expected value or closed inclusive range | Property outside its disposition quarantines |
| REQ-AGX-08 | Quarantine classes closed to `unknown-chip`, `ambiguous-identity`, `future-generation`, `below-minimum-generation`, `virtual-or-foreign`, `profile-mismatch`, `profile-unbound`, and `profile-unratified`, each mapped to the ratified failure code and JSON path | A device in any class never selects the AGX renderer and never contributes evidence |
| REQ-AGX-09 | A generated constants output for the downstream Mesa build (section 5.4) produced by the F-02 generator from the ratified profile, digest-locked as a `generated-from-locked-input/v1` config input; this requires the F-02 generator target set to admit a constants target | Until ratified, no downstream Mesa patch may hardcode chip, generation, or feature tables |
| REQ-AGX-10 | The extension is signed under the `board-admission` role with the registry, shares the registry `schema_set_digest`, and is versioned by document lineage; consumers compare it by digest through TP-01 and TP-12 | Unsigned, stale, or forked profile quarantines |
| REQ-AGX-11 | The generated profile has `runtime_compatible_allowlist`, an explicit non-empty closed array of exact strings beginning `apple,agx-`; it has no wildcard, prefix-only, regex, or future placeholder semantics. The profile also binds `runtime_compatible_observation_path = $.payload.observation.device_tree.compatible` and the DTB identity (ABI-09/ABI-10) into ID-25 | A value not in the exact allowlist is `MESA_E_AGX_COMPAT_UNKNOWN` at `$.payload.observation.device_tree.compatible`, a value syntactically in the `apple,agx-*` family but absent from the ratified list is `MESA_E_AGX_COMPAT_FUTURE` at the same path, and an allowlisted value whose board, DTB, or profile identity differs is `MESA_E_AGX_COMPAT_PROFILE_MISMATCH` at `$.payload.gpu_profile.runtime_compatible_allowlist`; all are `REJECT`, `exit=78` at P15 |
| REQ-AGX-12 | TP-08 reads the runtime device-tree compatible value at `$.payload.observation.device_tree.compatible`; the generated binding supplies the allowlist and its digest, and the guard compares the observed value, board ID, DTB artifact digest, and ID-25 profile digest as one admission seam | Missing observation, duplicate observation, cross-board transplant, or stale profile is quarantined with a closed code/path and cannot select AGX or contribute evidence |

Twelve extension requirements are recorded. Until the coordinator ratifies the extension, G-01 is BLOCKED (section 19) and TP-12 has no producer.

### 5.3 Board-to-profile disposition at admission

At renderer admission the guard holds TP-01, TP-02, TP-08, TP-09, TP-10, and TP-12. It reads `drm_asahi_params_global` from the physical device and the runtime compatible value at `$.payload.observation.device_tree.compatible`, compares every REQ-AGX-02 and REQ-AGX-07 field to the generated profile constants, checks the driver name per REQ-AGX-05, checks the generation bounds per REQ-AGX-03, checks exact membership in `runtime_compatible_allowlist` per REQ-AGX-11, and checks that the observed board and DTB (TP-08) are the board and DTB bound to the profile (REQ-AGX-01 and REQ-AGX-12). Every comparison is equality or closed-set membership. The first mismatch quarantines the device in its REQ-AGX-08 or REQ-AGX-11 class with the exact code/path from the error registry; the guard then leaves the last-known-good tuple active and records the class and path. There is no nearest-generation fallback, no G13G default, no software-rasterizer substitution reported as graphics success, and no feature advertisement beyond the profile.

### 5.4 Downstream build binding

A future downstream patch (G-02 or later, never G-01) may compile the generated profile constants into `src/asahi/lib/agx_device.c` and `src/asahi/vulkan/hk_physical_device.c` so that chip mapping, decode parameters, exact `apple,agx-*` compatible admission, and feature advertisement are table-driven from the ratified profile and reject everything else. That patch is admissible only when REQ-AGX-09 through REQ-AGX-12 are ratified, the constants file is a digest-locked config input (ID-13), and fixtures FX-15 through FX-22 and FX-80 through FX-82 execute against the built driver and admission guard. Until then, the unconditional code paths in section 5.1 remain residuals and no board is admitted.

## 6. G-01 authoritative mirror synchronization

### 6.1 Mirror invariants

The mirror is healthy only if all of the following hold:

- The GitHub repository is owned by `omarchy-silicon`, its canonical URL is the one recorded in the platform manifest source provenance, and its default branch is `main`.
- The authoritative source URL is exactly `https://gitlab.freedesktop.org/mesa/mesa` or a coordinator-approved immutable equivalent.
- `origin/main` and authoritative `main` resolve to the same commit object (ID-10). Equality is required; ancestry is insufficient for the pure mirror branch.
- The mirror retains the authoritative commit history needed to reproduce the selected source. A shallow or object-missing checkout is rejected for release assembly; blob filtering is permitted only when the object set is recorded (section 7.5).
- The mirror branch has no Omarchy-only commits, generated release edits, vendored graphics code, or merge-only synchronization commits.
- The commit, tree, parent set, and repository URL recorded in the provenance record (section 7.1) agree with the verified remote values.
- Any downstream patch branch declares the exact mirror commit from which it was based and the ordered downstream commit IDs (section 7.2).

The snapshot observation `d870cef8b7c8a4a11edc669669c9f18ae402314a` on 2026-09-02 is an observed synchronization input, not a claim that future mirror state, conformance, or hardware support is complete. As of the review, freedesktop `main` is `c0682c54768a5e247c6627fc3b632eb81b7af2a4` while GitHub mirror `main` is `d870cef8b7c8a4a11edc669669c9f18ae402314a`; this live drift is a provenance HOLD. The owned auditable synchronization and re-attestation required to close it is outside this documentation slice and cannot be converted to PASS here.

### 6.2 Synchronization procedure

1. Fetch authoritative refs and the GitHub mirror with pruning, accepting only the allowlisted immutable URLs, and record the ref advertisement digest (ID-09B).
2. Resolve both `refs/heads/main` values and record the full object IDs, commit metadata, tree ID (ID-11), and parent IDs.
3. Verify repository ownership and default-branch metadata through an authenticated owner check.
4. Verify that the authoritative commit is present locally and that no object needed for the candidate is missing.
5. Require exact equality of the upstream and mirror commit IDs.
6. Verify that the candidate downstream base equals that commit and that the patch queue applies cleanly in recorded order.
7. Write the provenance record (section 7.1), check time from TP-07, tool versions, and raw command results into evidence signed under the `ci-conformance` role, referenced by the manifest `report_lock`.

### 6.3 Mirror drift is a hard gate

| Condition | Required result |
| --- | --- |
| `origin/main` commit differs from authoritative `main` commit | Stop synchronization, block patch rebase and candidate assembly, and open a coordinator ruling. |
| `origin/main` contains extra Omarchy commits | Reject the mirror branch; move work to a downstream ref only through an auditable correction. Never auto-reset or hide the commits. |
| Remote, owner, default branch, or repository identity changes unexpectedly | Stop before accepting new source and require explicit owner/coordinator review. |
| Authoritative `main` is force-rewritten or the recorded base is no longer an ancestor | Stop candidate assembly and invalidate affected provenance records until history is reconciled. |
| Required commit/tree/tag/blob objects cannot be fetched from the allowlisted source | Reject the candidate as non-reproducible; do not substitute a local cache or another fork. |
| Patch queue base does not equal the verified mirror commit | Reject the queue; rebase or regenerate it and rerun all dependent gates. |
| Source identity absent, mutable, unsigned where signature is required, or inconsistent across provenance, build, and package metadata | Reject before build or package publication. |
| Drift discovered after build or packaging | Mark artifacts quarantined, preserve evidence, and assemble a new candidate from a clean verified base. |

The gate applies to edge, RC, and stable. A drift warning is not an acceptable status, and a successful build cannot override this gate. The current pair `c0682c54768a5e247c6627fc3b632eb81b7af2a4` versus `d870cef8b7c8a4a11edc669669c9f18ae402314a` remains HOLD until the owned synchronization job records the new advertisement digest, fetch clock, object type, signed/attested ref, peeled commit, tree, parent set, materialized object-set digest, and rerun evidence. That sync and re-attestation are outside this docs-only slice. A coordinator-approved mirror repair is a new auditable event recording old and new commit IDs, reason, impact, reruns, and retained failed evidence. The design never auto-merges upstream into a diverged mirror branch and never reports this drift as PASS.

## 7. Source, patch, dependency, and generated-source provenance

### 7.1 Source provenance record

The Mesa component provenance report referenced by `components.mesa_stack.source.provenance_report_digest` is a closed record produced by the synchronization job and signed under `ci-conformance`. Its required members are:

| Member | Content | Reject when |
| --- | --- | --- |
| `upstream_url` | Immutable HTTPS URL of the authoritative repository, no query, fragment, or branch | Not on the allowlist, or any branch or moving locator present |
| `mirror_url` | Immutable HTTPS URL of the owned mirror | Owner or URL differs from the recorded value |
| `ref_name` | Locator only: the exact ref (`refs/heads/main` or `refs/tags/release-name`) used for the fetch; it is never an identity | A mutable locator is supplied as ID-09A, ID-09B, ID-10, ID-11, or any artifact identity |
| `tag_object_id` or `ref_object_id` | ID-09A; exact Git object ID with explicit Git object type `tag` for an annotated tag or `commit` for an explicitly permitted signed ref; the signer/attestor is bound to this object | Object type, object bytes, signer, or typed ID is absent or substituted |
| `peeled_commit` | ID-10; exact commit object ID reached by peeling the ref, with the fetched commit bytes and ordered parent set bound | Not equal to the verified mirror commit or its fetched object |
| `tree_id` | ID-11; exact tree object ID of `peeled_commit`, bound to its commit object | Not equal to the tree recorded at fetch |
| `parent_ids` | Ordered parent commit IDs, included in the commit preimage and queue-base binding | Differ from the fetched commit object |
| `signer_evidence` | Upstream tag signer key fingerprint and signature bytes digest where upstream signs the ref; otherwise an explicitly ratified coordinator-signed attestation naming its authority, object type, object ID, and scope | Neither the required upstream signature nor an explicitly ratified attestation is present |
| `ref_advertisement_digest` | ID-09B; `sha256(CONCAT(ASCII("omarchy-ref-advertisement/v1"),0x00,JCS(sorted({remote,ref_name,object_id,peeled_object_id,fetch_clock}))))` over both remotes, with the fetch clock included in the preimage | Advertisement differs between the two remotes for `refs/heads/main`, the clock is absent/stale, or the digest is replaced by an object ID |
| `prior_base` | The previous candidate's `peeled_commit`, for range-diff and rebase reports | Missing when a prior candidate exists |
| `object_set_digest` | Digest of the sorted list of commit, tree, tag, and blob object IDs materialized for the build (section 7.5) | Any object needed by the build is absent from the set |
| `fetch_clock` | TP-07 attested monotonic sample plus UTC representation recorded by the synchronization job at fetch; it is not an identity | Caller wall clock, missing sample, or freshness expiry |

The manifest `source` record for `components.mesa_stack` carries `source_kind = git-repository/v1`, `repository_id` (ID-08), `source_commit` (queue tip, ID-10), `upstream_commit` (peeled mirror base, ID-10), `source_digest` (digest of the sorted provenance object set), and `provenance_report_digest`. The Git tag/ref object identity, ref-advertisement digest, peeled commit, tree, ordered parent set, signer/attestation authority, and fetch clock are separate typed members. A missing member, a mutable locator, a network-only object, or a digest supplied for the wrong Git object type rejects the component.

### 7.2 Minimal downstream patch queue

The downstream queue is a short, ordered series of atomic Git commits above the exact verified mirror commit. The queue is the implementation history; a second manually maintained patch list is not an authority. The manifest records the queue through `patch_lock` (ID-12); the repository preserves the reviewable commits and their disposition.

Each ordered queue entry, both in the `PatchEntry` (`patch_id`, `source_digest`, `patch_digest`, `order`) and in the queue report signed under `ci-conformance`, records:

- Commit ID and parent ID; the first parent of `order = 1` equals `upstream_commit`; each later parent equals the previous entry's commit.
- Patch digest over the canonical `git format-patch` bytes with normalized author date and no signature trailer.
- Scoped subject naming the Mesa subsystem and the AGX or platform reason.
- Exact affected board records and AGX profile IDs, if any.
- Upstream disposition: `proposed`, `accepted`, `waiting`, `intentionally-downstream`, or `scheduled-for-removal`, with the freedesktop review reference where one exists.
- Owner and removal condition or review date.
- Kernel/DTB/firmware ABI assumptions and the ABI tuple members (section 8) that must match.
- Focused tests and the expected evidence type.

The queue report additionally records the queue tip commit and tree, the range-diff digest against `prior_base`, and the changed-path census. Reordering, dropping, duplicating, or editing a patch without a new queue report rejects the queue. The queue must not contain unrelated style rewrites, generated artifacts without their generator input, copied upstream commits with altered identity, broad future-chip fallbacks, or a workaround whose only proof is a desktop that happens to start.

| Class | Policy | Exit condition |
| --- | --- | --- |
| Upstream candidate | Keep the smallest upstreamable change, link its freedesktop review, and test against the authoritative base. | Remove the downstream copy after the upstream commit is in the verified mirror. |
| Compatibility bridge | Carry only when a pinned Omarchy kernel/firmware ABI needs a temporary bridge that upstream cannot yet consume. | Remove when the ABI contract or upstream implementation changes, with a recorded rebase decision. |
| Board or generation enablement | Add profile-driven capability data and compiler/runtime work only for boards with a ratified AGX profile and required evidence. | Replace by upstream support or keep with an explicit board lifecycle and maintenance owner. |
| Release integration | Packaging, manifest, or build metadata that belongs outside Mesa code. | Keep out of `origin/main`; encode through the platform release pipeline. |
| Emergency corrective fix | Use only for a release-blocking regression with a reproducible failure and rollback plan. | Upstream immediately where possible; otherwise expire at the next synchronization checkpoint. |

Queue review rejects a patch when a capability can be expressed by an existing upstream interface, when it widens support without board evidence, when it duplicates canonical platform data, or when its removal condition is missing. "Accepted" means the commit is present in the verified authoritative history and the downstream copy has been removed or proven necessary by a new ABI decision. A rebase may never relax a hard gate to preserve a release date, and shared release refs are never force-pushed.

### 7.3 Subproject, wrap, and fallback closure

The frozen tree contains 42 `subprojects/*.wrap` files and a `subprojects/packagefiles/` directory. Every one of them, whether or not the selected build profile uses it, is locked:

- Each wrap's `source_url`, `source_fallback_url`, `source_filename`, `source_hash`, `patch_url`, `patch_fallback_url`, `patch_filename`, `patch_hash`, `wrapdb_version`, and any `directory` value are recorded, and the wrap file bytes themselves are digest-locked as a `toolchain_lock` entry or `config_inputs` entry.
- Every `packagefiles/` overlay directory is digest-locked as a sorted file-list digest.
- The build passes `--wrap-mode=nodownload`; any subproject the profile requires is admitted only from the offline cache (section 7.5) by matching `source_hash`, and any `force_fallback_for` or `allow-fallback-for` value is an explicit recipe member, not a default.
- System dependencies used instead of a wrap are locked by package name, version, and content digest in `toolchain_lock`.
- A wrap with no hash, a fallback URL not in the cache manifest, a wrap that would download at configure time, or a system dependency not in the lock rejects the build.

### 7.4 Generated-source locks

Mesa generates source at configure and build time from Python and Mako inputs. `meson.build:1176-1226` searches an interpreter list (`python3.16` down to `python`) and requires Mako at least 0.8.0 by probing the host; this auto-detection must not reach a release build. The recipe therefore records:

- Generator tool identities: the exact Python interpreter path and version, Mako version and source digest, and every generator script under `src/` reached by the build, each as a `ToolchainEntry` with `toolchain_digest` and `flags_digest`.
- Generator input digests (ID-13) for every template, XML, and Python input, as `ConfigInput` records with `normalized_content_digest`.
- Generator output digests (ID-14) for every generated file, captured from both builders and compared byte-for-byte.
- Any generator whose output differs between builders, or whose input is not in the lock, rejects the candidate.

### 7.5 Offline cache closure

Before the build starts, a content-addressed cache is admitted containing exactly: the Git object set named by `object_set_digest`, every wrap source and patch archive named in section 7.3 with its `source_hash`, every system dependency package with its content digest, the builder image with its digest, and the generated AGX profile constants (when ratified). The cache manifest is a sorted list of `(cache_key, content_digest, size_bytes)` and its digest is a recipe member. After admission, network access is denied to the build (section 9.2). Any input not in the cache manifest, any cache entry whose bytes do not match its digest, and any attempt to resolve a name over the network rejects the build.

## 8. Kernel, DTB, firmware, and Mesa ABI tuple

### 8.1 Tuple members

The complete ABI tuple is the closed list below. Every member is typed, has one owner, one exact preimage, one source path, one comparison path, one freshness rule, and one rejection contract. One missing, unknown, stale, or unequal member quarantines the candidate on the builder and quarantines the device at admission, in both cases before renderer selection. The ABI-25 through ABI-44 additions are not implied by a broad digest: each is independently present and is also included in the tuple digest.

| ID | Typed member | Owner and exact preimage | Source path | Comparison path | Freshness | Mismatch contract | Closure |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ABI-01 | `board_id` | F-02; `JCS(board_id)` from the verified board record | `$.payload.board_targets[*].board_id`, TP-08 | `$.payload.abi_tuple.ABI-01` | Current manifest and TP-07 expiry | `MESA_E_ABI_ABI01;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-01` | Tuple, package, admission, rollback, F-07 |
| ABI-02 | `agx_profile_id_digest` | F-02; `sha256(JCS({profile_id,profile_digest}))` from ID-25 | `$.payload.gpu_profile` and TP-12 | `$.payload.abi_tuple.ABI-02` | Current profile expiry | `MESA_E_ABI_ABI02;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-02` | Tuple, package, admission, rollback, F-07 |
| ABI-03 | `kernel_release` | K-01; `JCS(kernel.release)` from the `abi/v1` report | `$.payload.components.linux_kernel.abi_report.kernel_release` | `$.runtime.kernel.release` | Per candidate and boot session | `MESA_E_ABI_ABI03;P10;REJECT;exit=78` at `$.runtime.kernel.release` | Tuple, package, admission, rollback, F-07 |
| ABI-04 | `kernel_source_commit` | K-01; Git commit object ID bytes for the verified kernel source commit | `$.payload.components.linux_kernel.source.source_commit` | `$.runtime.kernel.source_commit` | Per candidate | `MESA_E_ABI_ABI04;P10;REJECT;exit=78` at `$.runtime.kernel.source_commit` | Tuple, package, admission, rollback, F-07 |
| ABI-05 | `kernel_config_digest_set` | K-01; `sha256(JCS(sorted(config_input_id,normalized_content_digest)))` | `$.payload.components.linux_kernel.config_inputs[*]` | `$.payload.abi_tuple.ABI-05` | Per candidate | `MESA_E_ABI_ABI05;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-05` | Tuple, package, admission, rollback, F-07 |
| ABI-06 | `asahi_uapi_header_digest` | K-01 and Mesa; `sha256(exact include/drm-uapi/asahi_drm.h bytes)` on each side | `$.payload.abi_report.asahi_uapi_header_digest` | `$.payload.abi_tuple.ABI-06` | Per candidate | `MESA_E_ABI_ABI06;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-06` | Tuple, package, admission, rollback, F-07 |
| ABI-07 | `drm_asahi_uapi_layout_digest` | K-01; `sha256(JCS({struct_layouts,ioctl_numbers,feature_bits,max_clusters}))` | `$.payload.abi_report.drm_asahi_uapi_layout` | `$.payload.abi_tuple.ABI-07` | Per candidate | `MESA_E_ABI_ABI07;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-07` | Tuple, package, admission, rollback, F-07 |
| ABI-08 | `abi_contract_id` | K-01; exact `kernel-abi/v1` contract ID bytes | `$.payload.components.linux_kernel.abi_contract_id` | `$.payload.abi_tuple.ABI-08` | Per candidate | `MESA_E_ABI_ABI08;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-08` | Tuple, package, admission, rollback, F-07 |
| ABI-09 | `dtb_artifact_digest_set` | K-01; `sha256(JCS(sorted(artifact_id,content_digest)))` | `$.payload.components.dtb_set.artifacts[*]` | `$.runtime.device_tree.artifacts` | Per candidate and boot session | `MESA_E_ABI_ABI09;P10;REJECT;exit=78` at `$.runtime.device_tree.artifacts` | Tuple, package, admission, rollback, F-07 |
| ABI-10 | `dt_schema_and_required_property_digest` | K-01; `sha256(JCS({schema_id,schema_version,schema_digest,required_property_ids}))` | `$.payload.components.dtb_set.dt_schema` | `$.runtime.device_tree.schema_digest` | Per candidate | `MESA_E_ABI_ABI10;P10;REJECT;exit=78` at `$.runtime.device_tree.schema_digest` | Tuple, package, admission, rollback, F-07 |
| ABI-11 | `firmware_artifact_identity_set` | Firmware owner; `sha256(JCS(sorted(artifact_id,artifact_version,content_digest)))` | `$.payload.components.firmware_bundle.artifacts[*]` | `$.runtime.firmware.artifacts` | Per candidate and boot session | `MESA_E_ABI_ABI11;P10;REJECT;exit=78` at `$.runtime.firmware.artifacts` | Tuple, package, admission, rollback, F-07 |
| ABI-12 | `firmware_schema_digest` | Firmware owner; `sha256(JCS({schema_id,schema_version,schema_digest}))` | `$.payload.components.firmware_bundle.firmware_schema` | `$.runtime.firmware.schema_digest` | Per candidate | `MESA_E_ABI_ABI12;P10;REJECT;exit=78` at `$.runtime.firmware.schema_digest` | Tuple, package, admission, rollback, F-07 |
| ABI-13 | `firmware_kernel_relation_digest` | Firmware owner; `sha256(JCS({relation_id,owner,left,right,contract_id,evidence_digest}))` | `$.payload.components.firmware_bundle.compatibility_relations[*]` | `$.payload.abi_tuple.ABI-13` | Per candidate | `MESA_E_ABI_ABI13;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-13` | Tuple, package, admission, rollback, F-07 |
| ABI-14 | `mesa_base_and_queue_tip` | Mesa lane; Git object preimages for `upstream_commit` and `source_commit` plus their parent bindings | `$.payload.components.mesa_stack.source` | `$.payload.abi_tuple.ABI-14` | Per candidate | `MESA_E_ABI_ABI14;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-14` | Tuple, package, admission, rollback, F-07 |
| ABI-15 | `mesa_patch_lock_digest` | Mesa lane; `sha256(JCS(sorted(PatchEntry{patch_id,source_digest,patch_digest,order})))` | `$.payload.components.mesa_stack.patch_lock` | `$.payload.abi_tuple.ABI-15` | Per candidate | `MESA_E_ABI_ABI15;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-15` | Tuple, package, admission, rollback, F-07 |
| ABI-16 | `mesa_build_profile_digest` | F-04; `sha256(JCS({recipe_digest,config_inputs,toolchain_lock_digest,cache_manifest_digest,argv_digest}))` | `$.payload.components.mesa_stack.build_profile` | `$.payload.abi_tuple.ABI-16` | Per candidate | `MESA_E_ABI_ABI16;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-16` | Tuple, package, admission, rollback, F-07 |
| ABI-17 | `mesa_generated_artifact_digest_set` | Mesa lane; sorted exact package bytes by artifact ID | `$.payload.components.mesa_stack.artifacts[*]` | `$.payload.abi_tuple.ABI-17` | Per candidate | `MESA_E_ABI_ABI17;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-17` | Tuple, package, admission, rollback, F-07 |
| ABI-18 | `package_identity_set` | P-05; `sha256(JCS(sorted(package_name,architecture,version,content_digest)))` | `$.payload.components.mesa_stack.packages[*]` | `$.runtime.packages` | Install and boot session | `MESA_E_ABI_ABI18;P10;REJECT;exit=78` at `$.runtime.packages` | Tuple, package, admission, rollback, F-07 |
| ABI-19 | `agx_feature_and_property_digest` | F-02 profile owner; `sha256(JCS(sorted(feature_or_property,disposition,expected_value_or_range)))` | `$.payload.gpu_profile.features` and `properties` | `$.runtime.agx.feature_report_digest` | Per candidate and admission | `MESA_E_ABI_ABI19;P10;REJECT;exit=78` at `$.runtime.agx.feature_report_digest` | Tuple, package, admission, rollback, F-07 |
| ABI-20 | `kernel_mesa_relation_digest` | K-01; `sha256(JCS({owner,left,right,contract_id,evidence_digest}))` with owner `linux-kernel` and right `mesa-stack` | `$.payload.components.linux_kernel.compatibility_relations[*]` | `$.payload.abi_tuple.ABI-20` | Per candidate | `MESA_E_ABI_ABI20;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-20` | Tuple, package, admission, rollback, F-07 |
| ABI-21 | `package_architecture_relation` | P-05; `JCS({relation_id,left,right,architecture})`, architecture exactly `aarch64` | `$.payload.components.mesa_stack.package_architecture_relation` | `$.runtime.packages.architecture` | Install and boot session | `MESA_E_ABI_ABI21;P10;REJECT;exit=78` at `$.runtime.packages.architecture` | Tuple, package, admission, rollback, F-07 |
| ABI-22 | `rollback_predecessor_identity` | F-07; `sha256(JCS({previous_component_ids,previous_manifest_ids,artifact_ids,retention_count,rollback_policy_digest,tuple_digest}))` | `$.payload.components.mesa_stack.rollback` | `$.payload.abi_tuple.ABI-22` | Per candidate and until successor health expires | `MESA_E_ABI_ABI22;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-22` | Tuple, package, admission, rollback, F-07 |
| ABI-23 | `opaque_boot_artifact_digest_set` | Human-owned producer via coordinator; exact sorted opaque content digests only | `$.payload.components.boot_stack.artifacts[*].content_digest` | `$.runtime.boot.artifacts` | Per manifest and boot session | `MESA_E_ABI_ABI23;P10;REJECT;exit=78` at `$.runtime.boot.artifacts` | Tuple, package, admission, rollback, F-07 |
| ABI-24 | `schema_set_digest` | F-02; ID-03 preimage over the schema input lock | `$.schema_set_digest` in every document and both locks | `$.payload.abi_tuple.ABI-24` | At every use and expiry | `MESA_E_ABI_ABI24;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-24` | Tuple, package, admission, rollback, F-07 |
| ABI-25 | `libdrm_source_identity_digest` | libdrm owner; `sha256(JCS({repository_id,commit,tree,source_file_digest_set}))` | `$.payload.abi_report.libdrm.source_identity` | `$.payload.abi_tuple.ABI-25` | Per candidate | `MESA_E_ABI_ABI25;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-25` | Tuple, package, admission, rollback, F-07 |
| ABI-26 | `libdrm_package_identity_digest` | P-05; `sha256(JCS({package_name,version,architecture,content_digest}))` for the installed libdrm package | `$.payload.abi_report.libdrm.package_identity` | `$.runtime.libdrm.package_identity` | Install and boot session | `MESA_E_ABI_ABI26;P10;REJECT;exit=78` at `$.runtime.libdrm.package_identity` | Tuple, package, admission, rollback, F-07 |
| ABI-27 | `libdrm_asahi_uapi_wrapper_digest` | libdrm owner; `sha256(exact normalized wrapper/header bytes and generated wrapper options)` | `$.payload.abi_report.libdrm.asahi_uapi_wrapper` | `$.runtime.libdrm.asahi_uapi_wrapper_digest` | Per candidate | `MESA_E_ABI_ABI27;P10;REJECT;exit=78` at `$.runtime.libdrm.asahi_uapi_wrapper_digest` | Tuple, package, admission, rollback, F-07 |
| ABI-28 | `syncobj_uapi_semantics_digest` | K-01 and Mesa; `sha256(JCS({ioctls,flags,handle_lifetime,timeline_rules,wait_reset_submit_semantics}))` | `$.payload.abi_report.syncobj.uapi_semantics` | `$.runtime.syncobj.uapi_semantics_digest` | Per candidate and boot session | `MESA_E_ABI_ABI28;P10;REJECT;exit=78` at `$.runtime.syncobj.uapi_semantics_digest` | Tuple, package, admission, rollback, F-07 |
| ABI-29 | `syncobj_capability_report_digest` | K-01; `sha256(JCS({supported_features,probe_inputs,probe_result,reset_behavior}))` | `$.payload.abi_report.syncobj.capability_report` | `$.runtime.syncobj.capability_report_digest` | Per boot session | `MESA_E_ABI_ABI29;P10;REJECT;exit=78` at `$.runtime.syncobj.capability_report_digest` | Tuple, package, admission, rollback, F-07 |
| ABI-30 | `vulkan_component_identity_digest` | Vulkan owner; `sha256(JCS({loader_id,loader_version,icd_id,icd_version,source_commit,content_digest}))` | `$.payload.abi_report.vulkan` | `$.runtime.vulkan.component_identity` | Install and boot session | `MESA_E_ABI_ABI30;P10;REJECT;exit=78` at `$.runtime.vulkan.component_identity` | Tuple, package, admission, rollback, F-07 |
| ABI-31 | `opengl_component_identity_digest` | OpenGL owner; `sha256(JCS({loader_id,loader_version,driver_id,driver_version,source_commit,content_digest}))` | `$.payload.abi_report.opengl` | `$.runtime.opengl.component_identity` | Install and boot session | `MESA_E_ABI_ABI31;P10;REJECT;exit=78` at `$.runtime.opengl.component_identity` | Tuple, package, admission, rollback, F-07 |
| ABI-32 | `egl_component_identity_digest` | EGL owner; `sha256(JCS({loader_id,loader_version,platforms,source_commit,content_digest}))` | `$.payload.abi_report.egl` | `$.runtime.egl.component_identity` | Install and boot session | `MESA_E_ABI_ABI32;P10;REJECT;exit=78` at `$.runtime.egl.component_identity` | Tuple, package, admission, rollback, F-07 |
| ABI-33 | `wsi_component_identity_digest` | WSI owner; `sha256(JCS({backend_ids,backend_versions,protocol_set,source_commit,content_digest}))` | `$.payload.abi_report.wsi` | `$.runtime.wsi.component_identity` | Per topology and boot session | `MESA_E_ABI_ABI33;P10;REJECT;exit=78` at `$.runtime.wsi.component_identity` | Tuple, package, admission, rollback, F-07 |
| ABI-34 | `compiler_identity_digest` | F-04; `sha256(JCS({compiler_id,version,source_commit,binary_digest}))` | `$.payload.build_report.compiler` | `$.payload.abi_tuple.ABI-34` | Per candidate | `MESA_E_ABI_ABI34;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-34` | Tuple, package, admission, rollback, F-07 |
| ABI-35 | `compiler_flags_digest` | F-04; `sha256(JCS(sorted(argv compiler flags,debug maps,optimization flags)))` | `$.payload.build_report.compiler.flags_digest` | `$.payload.abi_tuple.ABI-35` | Per candidate | `MESA_E_ABI_ABI35;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-35` | Tuple, package, admission, rollback, F-07 |
| ABI-36 | `toolchain_component_identity_digest` | F-04; `sha256(JCS(sorted(ToolchainEntry{name,version,source_digest,binary_digest,flags_digest})))` | `$.payload.components.mesa_stack.toolchain_lock` | `$.payload.abi_tuple.ABI-36` | Per candidate | `MESA_E_ABI_ABI36;P10;REJECT;exit=78` at `$.payload.abi_tuple.ABI-36` | Tuple, package, admission, rollback, F-07 |
| ABI-37 | `cts_result_digest` | Q-02; `sha256(JCS({suite_id,suite_version,selection,attempts,expected,actual,evidence_digest}))` | `$.payload.qualification_reports.cts` | `$.payload.abi_tuple.ABI-37` | Candidate-bound and record expiry | `MESA_E_ABI_ABI37;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-37` | Tuple, package, admission, rollback, F-07 |
| ABI-38 | `deqp_result_digest` | Q-02; `sha256(JCS({suite_id,suite_version,selection,attempts,expected,actual,evidence_digest}))` | `$.payload.qualification_reports.deqp` | `$.payload.abi_tuple.ABI-38` | Candidate-bound and record expiry | `MESA_E_ABI_ABI38;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-38` | Tuple, package, admission, rollback, F-07 |
| ABI-39 | `glslang_validator_result_digest` | Q-02; `sha256(JCS({tool_id,version,argv_digest,shader_set_digest,expected,actual,evidence_digest}))` | `$.payload.qualification_reports.glslangValidator` | `$.payload.abi_tuple.ABI-39` | Candidate-bound and record expiry | `MESA_E_ABI_ABI39;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-39` | Tuple, package, admission, rollback, F-07 |
| ABI-40 | `vkcube_result_digest` | Q-02; `sha256(JCS({tool_id,version,argv_digest,display_topology,expected,actual,evidence_digest}))` | `$.payload.qualification_reports.vkcube` | `$.payload.abi_tuple.ABI-40` | Candidate-bound and record expiry | `MESA_E_ABI_ABI40;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-40` | Tuple, package, admission, rollback, F-07 |
| ABI-41 | `golden_output_digest` | Mesa/Q-02; `sha256(JCS(sorted(test_id,input_digest,output_digest,profile_id)))` | `$.payload.qualification_reports.golden` | `$.payload.abi_tuple.ABI-41` | Candidate-bound and record expiry | `MESA_E_ABI_ABI41;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-41` | Tuple, package, admission, rollback, F-07 |
| ABI-42 | `reset_report_digest` | Q-02; `sha256(JCS({reset_fixture,attempts,engine_results,fence_results,recovery_result,evidence_digest}))` | `$.payload.qualification_reports.reset` | `$.payload.abi_tuple.ABI-42` | Candidate-bound and record expiry | `MESA_E_ABI_ABI42;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-42` | Tuple, package, admission, rollback, F-07 |
| ABI-43 | `compositor_output_digest` | Q-02; `sha256(JCS({compositor_id,display_topology,interaction_matrix,visual_digest,health_digest}))` | `$.payload.qualification_reports.compositor` | `$.payload.abi_tuple.ABI-43` | Candidate-bound and record expiry | `MESA_E_ABI_ABI43;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-43` | Tuple, package, admission, rollback, F-07 |
| ABI-44 | `conformance_report_set_digest` | Q-02; `sha256(JCS(sorted({report_kind,result_digest,attempt_history_digest,tuple_digest,expiry})))` over ABI-37 through ABI-43 | `$.payload.qualification_reports.report_set_digest` | `$.payload.abi_tuple.ABI-44` | Candidate-bound and record expiry | `MESA_E_ABI_ABI44;P11;REJECT;exit=78` at `$.payload.abi_tuple.ABI-44` | Tuple, package, admission, rollback, F-07 |

Forty-four tuple members are defined. The canonical tuple preimage is `CONCAT(ASCII("omarchy-mesa-abi-tuple/v2"),0x00,JCS({ABI-01:tuple.ABI-01,ABI-02:tuple.ABI-02,ABI-03:tuple.ABI-03,ABI-04:tuple.ABI-04,ABI-05:tuple.ABI-05,ABI-06:tuple.ABI-06,ABI-07:tuple.ABI-07,ABI-08:tuple.ABI-08,ABI-09:tuple.ABI-09,ABI-10:tuple.ABI-10,ABI-11:tuple.ABI-11,ABI-12:tuple.ABI-12,ABI-13:tuple.ABI-13,ABI-14:tuple.ABI-14,ABI-15:tuple.ABI-15,ABI-16:tuple.ABI-16,ABI-17:tuple.ABI-17,ABI-18:tuple.ABI-18,ABI-19:tuple.ABI-19,ABI-20:tuple.ABI-20,ABI-21:tuple.ABI-21,ABI-22:tuple.ABI-22,ABI-23:tuple.ABI-23,ABI-24:tuple.ABI-24,ABI-25:tuple.ABI-25,ABI-26:tuple.ABI-26,ABI-27:tuple.ABI-27,ABI-28:tuple.ABI-28,ABI-29:tuple.ABI-29,ABI-30:tuple.ABI-30,ABI-31:tuple.ABI-31,ABI-32:tuple.ABI-32,ABI-33:tuple.ABI-33,ABI-34:tuple.ABI-34,ABI-35:tuple.ABI-35,ABI-36:tuple.ABI-36,ABI-37:tuple.ABI-37,ABI-38:tuple.ABI-38,ABI-39:tuple.ABI-39,ABI-40:tuple.ABI-40,ABI-41:tuple.ABI-41,ABI-42:tuple.ABI-42,ABI-43:tuple.ABI-43,ABI-44:tuple.ABI-44}))`; the tuple digest is SHA-256 of those bytes. The tuple digest and every member are copied into `components.mesa_stack.abi_tuple`, each package record, the admission input, the rollback predecessor/successor records, and the F-07 transaction and public-ledger closure. F-07 recomputes all 44 member preimages and the tuple digest; a broad component digest cannot stand in for a missing member.

### 8.2 Compatibility classes

For each member, compatibility is one of exact match required, explicitly compatible closed set signed by the owning component and covered by tests, or incompatible. There is no implicit latest, same-chip, or backward-compatible value. The design does not use the closed-set option for any member in v1; every row above is exact equality or exact digest equality. A firmware schema change, kernel reset behavior change, UAPI struct change, or compiler/runtime change invalidates the affected candidate until the complete tuple is rebuilt and requalified.

### 8.3 ABI report content

The `abi/v1` report whose digest is ID-18 is produced twice, independently, by K-01 tooling from the kernel tree and by the Mesa builder from the Mesa tree, and the two digests must be equal. The report content is closed: the UAPI header digest (ABI-06), the `drm_asahi_params_global` layout, each ioctl name and number, each feature bit name and value, `DRM_ASAHI_MAX_CLUSTERS`, the `abi_contract_id`, and the kernel release string. The companion `userspace-abi/v2` report separately covers ABI-25 through ABI-36, and the `qualification-output/v2` report covers ABI-37 through ABI-44; each report has its own typed digest and is included in ABI-44. Stable kernel DRM UAPI is distinguished from implementation-private interfaces; Mesa must not claim ABI compatibility because a symbol or device node exists.

### 8.4 Mismatch quarantine

On the builder, a tuple mismatch aborts candidate assembly with the failing member ID and exact comparison path recorded in the build report; no artifact is emitted. On the device, the admission guard evaluates all 44 members against the active kernel, DTB, firmware, libdrm, syncobj semantics, API/WSI components, compiler/toolchain identities, installed packages, and candidate-bound output reports before any DRI or Vulkan ICD is selected; a mismatch leaves the previous tuple active and reports the member ID and path. Downgrading only Mesa is forbidden when any member is incompatible with the active kernel, firmware, userspace, output evidence, or rollback predecessor.

## 9. Reproducible build and package design

### 9.1 Recipe and argv derivation

The build recipe is owned by F-04 and consumed as a pinned input; its digest is ID-21. `BuildRecipe/v1` is a closed authenticated grammar, not a bag of Meson options. Its exact members are `{recipe_id,version,source_commit,queue_lock_digest,paths,options,environment,object_set_digest,cache_manifest_digest,toolchain_lock_digest,argv_digest}`. The authenticated preimage is `CONCAT(ASCII("omarchy-build-recipe/v1"),0x00,JCS({recipe_id,version,source_commit,queue_lock_digest,paths,options,environment,object_set_digest,cache_manifest_digest,toolchain_lock_digest,argv_digest}))`, and `recipe_digest = sha256(preimage)`. Unknown members, duplicate members, omitted required members, aliases, auto-detected values, and values outside the closed choices are rejected at P09.

The immutable path tokens are exactly `SOURCE_DIR`, `BUILD_DIR`, `NATIVE_FILE_FROM_LOCK`, `DESTDIR`, `PREFIX`, and `LIBDIR`. A recipe maps them to the builder workspace by generated binding; a token is not a caller path, contains no `..`, symlink traversal, home-directory expansion, shell metacharacter, or network URL, and cannot be changed after cache admission. `jobs` is an integer in the inclusive range 1 through 256. The closed option vocabulary is: `buildtype` in `{debug,debugoptimized,release,minsize}`, `b_ndebug` in `{true,false}`, `gallium_drivers` in `{asahi,swrast}`, `vulkan_drivers` in `{asahi,swrast}`, `platforms` in `{drm,wayland,x11,surfaceless}`, `egl`, `gles2`, `opengl`, `gbm`, `glx`, `llvm`, `shared_llvm`, `shader_cache`, `gallium_rusticl`, `split_debug`, and `xmlconfig` each in `{true,false}`, `video_codecs` in `{all,none}`, `gallium_va` in `{true,false}`, `tools` in `{none,asahi}`, `build_tests` in `{true,false}`, and `allow_fallback_for` in `{none,asahi}`. `prefix` and `libdir` are the immutable `PREFIX` and `LIBDIR` tokens. No member has an `auto` or implicit value.

The exact argv arrays are:

~~~text
setup_argv = ["meson", "setup", BUILD_DIR, SOURCE_DIR,
  "--wrap-mode=nodownload",
  "--native-file", NATIVE_FILE_FROM_LOCK,
  "--buildtype=" + recipe.buildtype,
  "-Db_ndebug=" + recipe.b_ndebug,
  "-Dgallium-drivers=" + recipe.gallium_drivers,
  "-Dvulkan-drivers=" + recipe.vulkan_drivers,
  "-Dplatforms=" + recipe.platforms,
  "-Degl=" + recipe.egl, "-Dgles2=" + recipe.gles2, "-Dopengl=" + recipe.opengl,
  "-Dgbm=" + recipe.gbm, "-Dglx=" + recipe.glx,
  "-Dllvm=" + recipe.llvm, "-Dshared-llvm=" + recipe.shared_llvm,
  "-Dshader-cache=" + recipe.shader_cache,
  "-Dvideo-codecs=" + recipe.video_codecs, "-Dgallium-va=" + recipe.gallium_va,
  "-Dgallium-rusticl=" + recipe.gallium_rusticl,
  "-Dtools=" + recipe.tools, "-Dbuild-tests=" + recipe.build_tests,
  "-Dsplit-debug=" + recipe.split_debug,
  "-Dxmlconfig=" + recipe.xmlconfig,
  "-Dallow-fallback-for=" + recipe.allow_fallback_for,
  "-Dprefix=" + recipe.prefix, "-Dlibdir=" + recipe.libdir]
build_argv   = ["ninja", "-C", BUILD_DIR, "-j", recipe.jobs, "-v"]
install_argv = ["meson", "install", "-C", BUILD_DIR, "--destdir", DESTDIR, "--no-rebuild"]
~~~

`argv_digest` is `sha256(CONCAT(ASCII("omarchy-build-argv/v1"),0x00,JCS({setup_argv,build_argv,install_argv})))` after token resolution and before execution; the arrays, not a shell rendering, are authenticated and recorded. The native file is generated from the toolchain lock and names the compiler, linker, `ar`, `strip`, `pkg-config` path, and sysroot by content digest. The recipe also authenticates `object_set_digest = sha256(CONCAT(ASCII("omarchy-git-object-set/v1"),0x00,JCS(sorted({object_type,object_id,content_digest,size_bytes}))))`, `cache_manifest_digest = sha256(CONCAT(ASCII("omarchy-offline-cache/v1"),0x00,JCS(sorted({cache_key,content_digest,size_bytes}))))`, and `toolchain_lock_digest = sha256(CONCAT(ASCII("omarchy-toolchain-lock/v1"),0x00,JCS(sorted({name,version,source_digest,binary_digest,flags_digest}))))`. A cache or object may be used only when its bytes recompute the named preimage.

The environment closure is the exact sorted set `{SOURCE_DATE_EPOCH, TZ=UTC, LC_ALL=C.UTF-8, LANG=C.UTF-8, PYTHONHASHSEED=0, UMASK=022, HOME_CACHE=EMPTY_CONTENT_ADDRESS, PATH=LOCKED_TOOLCHAIN_BIN_DIRS}` plus the exact `MESON_*`, `CFLAGS`, `CXXFLAGS`, `LDFLAGS`, and `PKG_CONFIG_PATH` absence assertions. The environment digest is `sha256(CONCAT(ASCII("omarchy-build-environment/v1"),0x00,JCS(sorted(name,value_or_absent))))` and is a recipe member. Operator environment is never merged. The builder executes each argv array directly with `execve` and records its exit status. If a pipe is required for human-readable capture, it must run with explicit `pipefail` and record the status of every component plus the aggregate status; a zero status from `tee` cannot hide a failed producer. Network policy is offline after cache admission and any resolver, socket, or wrap fetch is an exact P09 rejection.

### 9.2 Hermetic environment

- Builder image digest, source checkout at the queue tip, and the offline cache (section 7.5) are the only inputs.
- Network is denied after cache admission by the builder sandbox; a resolver call, socket connect, or wrap download attempt is a build failure, not a warning.
- Environment is fixed to `SOURCE_DATE_EPOCH` from the queue tip commit time, `TZ=UTC`, `LC_ALL=C.UTF-8`, `LANG=C.UTF-8`, `PYTHONHASHSEED=0`, `UMASK=022`, an empty content-addressed `HOME_CACHE`, and no `MESON_*`, `CFLAGS`, `CXXFLAGS`, `LDFLAGS`, `PKG_CONFIG_PATH`, or `PATH` inherited from the operator.
- Paths are normalized with fixed source and build directories and `-ffile-prefix-map` and `-fdebug-prefix-map` entries recorded in the recipe.
- Build caches are disabled or content-addressed and included in provenance.
- No developer GPU, untracked local patch, or mutable home-directory state is readable by the build.

### 9.3 Two-build comparison

Every candidate is built by two isolated builders from the same source, queue, recipe, cache, and toolchain digests. The following must be byte-identical: every installed file after the declared deterministic packaging step, every generated source file (ID-14), build IDs, debug and source artifacts where shipped, the SBOM, the provenance attestation body, the compiler invocation transcript, and the license inventory. Intentionally non-identical items are limited to the builder attestation wrapper and signature envelopes, each named in the recipe. Any other difference is a release-blocking failure investigated as a source, toolchain, generated-code, environment, or packaging defect. One builder's output is never selected because it boots.

### 9.4 Complete artifact set

A candidate build emits exactly: runtime packages, development packages, debug packages, source package, test tools package (`agxdecode` and selected Mesa tools when `tools` includes `asahi`), the SBOM, the source provenance report (section 7.1), the queue report (section 7.2), the build transcript, the kernel ABI report (section 8.3), the userspace ABI report carrying ABI-25 through ABI-36, the `compatibility/v1` report carrying ABI-19, the CTS, deqp, glslangValidator, and vkcube reports, the golden output report, the reset report, the compositor output report, the conformance report-set digest, the two-build comparison report, the license inventory for F-06, and a detached signature per artifact. Every artifact is a `ComponentArtifact` of kind `mesa-package/v1` or a `ReportEntry` of kind `build/v1`, `abi/v1`, `userspace-abi/v2`, `qualification-output/v2`, or `compatibility/v1`. A missing member rejects the candidate.

### 9.5 Signatures, repository, channel, and install transaction

- Every artifact carries a detached signature verified against the F-03 role and key named by `signature_policy_id`; Mesa holds no signing key.
- Package repository indexes are signed under F-03 roles; `Optional TrustAll`, `skipinteg`, unsigned databases, and network transport success are never authority.
- The package repository namespace is bound to the manifest `channel`; a package in an `edge` namespace that is also reachable from a `stable` namespace without an F-07 promotion is a repository-integrity failure.
- Install is a transaction owned by P-05 and the installer slices: the new Mesa packages are written to the inactive slot, every `content_digest` is verified after write, the ABI tuple (section 8) is evaluated, and only then is the slot selected once; a boot-health failure returns to the predecessor per the manifest rollback records.
- The Mesa driver is not installed into an active search path outside this transaction.

### 9.6 Deterministic builder rejection classes

The candidate builder emits exactly one section 3.4 error record. The closed builder code registry is the exact set of fixture codes in section 20; the legacy class names below are explanatory labels only and cannot appear as a result without a code, path, phase, decision, and process result. The mapping to the F-04/F-05 vocabulary is itself a required DEP-03/DEP-04 artifact and remains BLOCKED until ratification.

| Closed condition family | Exact phase | Required path form | Decision and process result |
| --- | --- | --- | --- |
| Source object, ref, advertisement, or queue identity failure | P08 | `$.provenance.*` or `$.payload.components.mesa_stack.source.*` | `REJECT;exit=78` |
| Recipe grammar, immutable path, option, environment, cache, or toolchain failure | P09 | `$.build.recipe.*`, `$.cache.*`, or `$.build.environment.*` | `REJECT;exit=78` |
| Network attempt after cache admission | P09 | `$.build.network.*` | `REJECT;exit=78` |
| ABI tuple member or report mismatch | P10 | `$.payload.abi_tuple.ABI-NN` or exact `$.runtime.*` member path | `REJECT;exit=78` |
| Missing or stale build/conformance artifact | P11 | `$.payload.qualification_reports.*` or `$.payload.artifacts.*` | `REJECT;exit=78` |
| Candidate or promotion closure failure | P13 or P14 | `$.candidate.*` or `$.promotion.*` | `NO_PUBLISH;publish=NONE;exit=78` |
| Unratified F-02/F-03 dependency or unavailable executable authority | P01, P02, or P11 | `$.dependency.*` or `$.fixtures.*` | `HOLD;exit=79` |

The old broad labels `MESA_BUILD_SOURCE_UNVERIFIED`, `MESA_BUILD_QUEUE_INVALID`, `MESA_BUILD_INPUT_UNLOCKED`, `MESA_BUILD_NETWORK_DENIED`, `MESA_BUILD_ABI_MISMATCH`, `MESA_BUILD_NONREPRODUCIBLE`, `MESA_BUILD_ARTIFACT_INCOMPLETE`, and `MESA_BUILD_PROFILE_UNRATIFIED` are not accepted as standalone outcomes. Each future implementation must resolve to one closed code in section 20.

## 10. Shader, compiler, and GPU-generation work

### 10.1 Compiler contract

1. Ingest the API shader representation and capture the exact source or binary input.
2. Run the pinned Mesa/NIR lowering and optimization pipeline.
3. Apply only generation- and capability-specific lowering permitted by the ratified AGX profile for the admitted board.
4. Generate AGX intermediate representation and machine code with the pinned compiler/toolchain.
5. Validate register allocation, instruction encoding, resource limits, synchronization, barriers, interpolation, texture/image operations, and API-visible results.
6. Capture stable intermediate and final representations where deterministic output is promised, with the compiler options and profile identity.

Unknown GPU generations, instruction capabilities, firmware interfaces, resource limits, or shader feature combinations fail closed. They do not fall back to the nearest generation's encoding or silently disable a required API feature. Optional features remain explicitly optional in the capability report and never become a false success.

### 10.2 Golden and compiler tests

Golden tests are versioned inputs and expected outputs for NIR lowering, AGX IR, instruction selection, encoding, disassembly, and API-visible behavior where each representation is stable enough to promise. Every golden update records the reason, upstream or queue commit, profile, toolchain, and an independent semantic test. A changed golden file without a reason and review is a hard failure. The suite includes Mesa unit and NIR tests selected by changed path, shader-db comparison, API shader tests for the profile's required paths, negative tests for unsupported generation, feature, malformed shader, invalid resource use, and ABI mismatch, cache invalidation tests across queue, ABI, compiler, and profile changes, and determinism tests compiling the same corpus twice in clean build directories. Performance work may not trade away correctness, reset safety, or a required capability.

### 10.3 Generation intake lanes

| Goal | Scope | Entry condition | Exit evidence before F-07 promotion | Status |
| --- | --- | --- | --- | --- |
| G-01 | Mirror, provenance, queue, build, ABI, profile, and test contracts | Ratified F-02 including the AGX profile extension, F-04 builder, F-06 compliance | Implemented gates, executable fixtures (section 20), reproducible candidate pipeline, no drift | BLOCKED on F-02 rejection |
| G-02 | M1/M2 graphics reference tuple | G-01 DONE, K-02 kernel/DT package set, ratified profiles for every M1/M2 board | Per-board physical evidence, compiler/API tests, compositor matrix, reset results, package/reproducibility evidence, Q-04 records | TODO |
| G-03 | M3 base/Pro/Max/Ultra graphics and display dependencies | G-02 DONE, K-03, ratified M3 profiles | Same evidence per exact M3 board without inheriting M1/M2 assumptions, Q-05 records | TODO |
| G-04 | M4 base/Pro/Max graphics and display dependencies | G-03 DONE, K-04, ratified M4 profiles | Same evidence per exact M4 board, Q-06 records | TODO |
| G-05 | A18 Pro, M5 family, and M6 graphics dependencies | G-04 DONE, K-05 (and K-06 for M6), shipping hardware, ratified profiles | Same evidence per exact board, Q-07 and Q-08 records; announced hardware remains intake-only | TODO |

M6 remains a target until shipping hardware is acquired and qualified; an M6 intake record and preparatory compiler work are permitted, an M6 support label is not.

## 11. Conformance, golden, performance, and reset evidence

Every result attaches to a `qualification-record/v1` test result or a lower-level `ci-conformance`-signed report with: board ID and profile ID, physical lab asset pseudonym, macOS and firmware baseline digests, the complete ABI tuple digest set, suite and harness revision, environment, expected and actual result, timestamps from TP-07, operator or automation identity through TP-05, immutable raw evidence digests with privacy class, failure classification, reproduction status, quarantine state, and residual risk. Static checks, compilation, mocked device trees, VMs, virtual GPUs, and a successful desktop are lower-level evidence; none can create a physical qualification record or change a lifecycle state.

| Family | Minimum design coverage | Release rule |
| --- | --- | --- |
| Upstream/unit | Mesa Meson tests, NIR/compiler tests, API loader/runtime tests, changed-path tests | All required tests pass or have a coordinator-approved residual that prevents promotion |
| Conformance | OpenGL, Vulkan, EGL, shader, and compute conformance suites for the profile's required feature set | Required failures block candidate activation and promotion; a skipped test needs a reason and owner |
| Golden | Compiler IR, machine code, disassembly, and semantic shader corpus | Unexpected drift blocks the candidate until explained and reviewed |
| Performance | Fixed workloads, frame-time distributions, compile latency, memory, power, thermal behavior, regression against the last qualified tuple | Regression budget exceeded only through a recorded decision; unsafe thermal or power behavior always blocks |
| Reset/recovery | GPU hang/reset injection, engine recovery, fence and context behavior, compositor survival, repeated reset cycles, boot-health reporting | Any unrecovered hang, silent corruption, false success, or unsafe repeated-reset behavior blocks |
| Packaging | Install/upgrade/downgrade, dependency closure, signatures, SBOM/provenance, artifact digest, file ownership | Missing or mutable metadata blocks publication |

Tests distinguish an expected unsupported feature from a regression; neither is a success without its exact evidence contract. Reset and hang tests run only on disposable or quarantined lab targets with a rehearsed recovery path and never against a user's installation.

## 12. Hyprland/Quickshell regression matrix

| Axis | Required values or evidence |
| --- | --- |
| Hardware | Every candidate board record and applicable RAM/display class; reference coverage does not substitute for every board |
| Platform tuple | The complete section 8 tuple by member ID |
| Hyprland | Exact commit/package, renderer/backend options, config revision, protocol versions, logs |
| Quickshell | Exact commit/package, shell configuration revision, QML/runtime versions, logs |
| Displays | Internal panel, each supported external DP/HDMI/Thunderbolt path, hotplug, mixed DPI, multi-monitor, rotation/scaling, board-specific panel features from the registry |
| Interaction | Login, workspace/animation, window resize, fullscreen, screenshots/recording, clipboard/drag, lock/unlock, idle, suspend/resume, monitor reconfiguration, shell reload |
| Accessibility | Reduced-motion, high-contrast, and scaled rendering paths produce correct output; owned by `omarchy-mac` (P-02, P-04) with lab execution by Q-02 |
| Stress | Shader compilation during interaction, memory pressure, long compositor soak, repeated display hotplug, video/3D overlap, injected GPU reset where safe |
| Observables | Frame-time distribution, dropped frames, render errors, GPU hangs/resets, visual diffs, shell crashes/restarts, protocol errors, power/thermal telemetry, user-visible recovery |
| Evidence | Immutable capture/log/trace digests, exact expected result, pass/fail, environment, residuals |

A pass on an internal display cannot stand in for an external topology, and a compositor launch cannot stand in for a soak, reset, or shell-reload test. A regression in Hyprland or Quickshell is triaged against the same tuple and is never hidden by weakening Mesa or disabling a required feature without an explicit capability outcome.

## 13. Physical qualification closure

### 13.1 Q-row bindings

| Mesa goal | Qualification row | Binding |
| --- | --- | --- |
| G-02 | Q-04 | Every M1/M2 board with `gpu` present has a signed `qualification-record/v1` whose profile contains the AGX rows; Q-04 cannot pass without G-02 evidence and G-02 cannot exit without Q-04 records |
| G-03 | Q-05 | Same for every M3 board |
| G-04 | Q-06 | Same for every M4 board |
| G-05 | Q-07 and Q-08 | Same for every A18 Pro and M5 board (Q-07) and every M6 board (Q-08) |

### 13.2 Board/profile set equality

Three sets must be equal for a manifest to be eligible for F-07: the manifest `board_targets`, the set of boards whose registry record has `gpu` capability `present` and a bound ratified AGX profile, and the set of boards named by verified qualification records bound to that manifest with `outcome = pass`. A board in one set and not the others is a `CROSS_DOCUMENT_MISMATCH` (PROVISIONAL) and the manifest is not eligible. Family extrapolation is forbidden: a passing M3 Max record proves nothing about M3 Pro, a laptop record proves nothing about a desktop with the same SoC, and a die/bin, port-count, display, RAM, storage, or GPU-core-count difference creates a distinct profile requiring its own records.

### 13.3 Thresholds from PROGRAM policy

| ID | Threshold | Value | Source |
| --- | --- | --- | --- |
| QT-01 | Independently serialized physical units per materially distinct board/profile | 2 | PROGRAM section 9 |
| QT-02 | Clean installs per unit per exact profile | 3 | PROGRAM section 9 |
| QT-03 | Encryption paths per profile where applicable | 2 (encrypted and unencrypted) | PROGRAM section 9 |
| QT-04C | Cold boot cycles for each laptop board/profile | At least 50 | PROGRAM section 9, split minimum required by this correction |
| QT-04W | Warm boot cycles for each laptop board/profile | At least 50 | PROGRAM section 9, split minimum required by this correction |
| QT-05 | Attach/detach cycles per applicable port or device class | 5 | PROGRAM section 9 |
| QT-06 | Complete update/rollback cycles per profile | 10 | PROGRAM section 9 |
| QT-07 | Independent human operators per qualitative row | 2 | PROGRAM section 9 |
| QT-08 | Unexpected accepts permitted across the hostile fixture catalog | 0 | Section 20 |
| QT-09 | Retries permitted to be dropped, merged, or relabeled in an attempt history | 0 | Section 13.4 |

Ten thresholds are bound. QT-04C and QT-04W are independent minima; 25 cold plus 25 warm is an undercount for both and is not 50. A higher subsystem safety standard overrides these floors; none may be lowered by this lane.

### 13.4 Attempt histories and anti-laundering

Every attempt of every test on every unit is an immutable evidence entry with its own `content_digest`, timestamp, unit pseudonym, and outcome. QT-04C has a distinct test-result ID and immutable attempt-history digest from QT-04W; each history has at least 50 cold or warm attempts respectively and cannot borrow the other class. A retry is a new entry that references the prior attempt; the prior attempt is never deleted, overwritten, merged, or reclassified. The qualification record's `test_results` entry for a row reports the final status and lists every attempt's evidence ID; a record that lists fewer attempts than the lab controller recorded, or a pass whose attempt list omits a failed attempt, fails the record. Flake is a failure class with an owner, never a filter.

### 13.5 Human-observed and quantitative evidence

Every quantitative AGX row has an approved unit, procedure, fixture and calibration requirement, pass/fail threshold, measurement uncertainty, repetition count, and safety stop before execution. Every qualitative row (visual artifacts, panel quality, tearing, flicker, external-display behavior) has a bounded rubric, required capture, QT-07 operators, a disagreement/escalation rule, and a named acceptance authority. Software fallback (a software rasterizer, a virtual GPU, a headless render, a compositor that starts without acceleration) never proves a hardware capability and is recorded as `not-run` for the hardware row.

### 13.6 Signed receipts, accessibility, and privacy ownership

Qualification records are signed under the `qualification-lab` role; the operator and lab resolve through TP-05; no Mesa CI signature can produce a qualification record. Accessibility rows for the compositor and shell are owned by `omarchy-mac` (P-02, P-04) and executed by the lab under Q-02. Privacy is owned by Q-02 with the F-02 projection classes: public exports omit serials, unit identifiers, usernames, paths, network identifiers, and raw evidence; raw evidence is readable only through TP-11.

## 14. Lifecycle states for this lane

| State | Meaning |
| --- | --- |
| DETECTED | Identity recognized by the canonical registry; no graphics qualification implied |
| BRINGUP | Work permitted on a pinned tuple with a ratified profile; required tests or hardware evidence missing |
| EXPERIMENTAL | Some controlled evidence exists; failures, exclusions, or residuals remain |
| DAILY_DRIVER | Coordinator has accepted a narrower operational evidence set; still not FULL |
| FULL | Assigned only by the canonical program through F-07 after every applicable physical capability and cross-repository gate passes |

This document never assigns a lifecycle state. No physical evidence is fabricated from a VM, emulator, virtual GPU, static source check, or recognized GPU string.

## 15. Candidate emission and F-07 promotion

Mesa emits candidate artifacts and evidence only: the section 9.4 artifact set, signed under F-03 roles, described by `components.mesa_stack` records inside a manifest that F-05 assembles. Mesa does not assemble manifests, does not select a channel, does not write to any package namespace, does not copy digests between namespaces, and cannot mark a board, release, or slice as anything. There is no Mesa-local promotion record, promotion command, promotion role, or promotion key. F-07 is the only stable-promotion writer and, once its interface is ratified, recomputes the full closure (PROGRAM sections 12.1, 13, 17) before publishing an immutable candidate digest.

The future-ratified F-07 transaction input is a closed `F07PromotionTransaction/v1` record with exactly `transaction_id`, `state`, `candidate_id`, `candidate_digest`, `board_id_set_digest`, `artifact_set_digest`, `manifest_id`, `manifest_digest`, `generation`, `lineage_digest`, `rollback_digest`, `f06_compliance_bundle_digest`, `public_ledger_projection_digest`, `health_evidence_digest`, `freshness`, and `recovery_attempt`. Its authenticated preimage is `CONCAT(ASCII("omarchy-f07-promotion/v1"),0x00,JCS({transaction_id,state,candidate_id,candidate_digest,board_id_set_digest,artifact_set_digest,manifest_id,manifest_digest,generation,lineage_digest,rollback_digest,f06_compliance_bundle_digest,public_ledger_projection_digest,health_evidence_digest,freshness,recovery_attempt}))`. The state enum is exactly `PRECOMMIT`, `PUBLISH_MARKED`, `PUBLISHED`, `CLEANUP_PENDING`, `RETRY_PENDING`, or `ABORTED`. The precommit digest binds the candidate, board set, all 44 ABI members and tuple digest, artifact set, manifest identity, generation, lineage, rollback predecessor, F-06 compliance bundle, public-ledger projection, health evidence, and freshness. The atomic publish marker contains the same digest and is written only by F-07 in the stable namespace; a marker with any changed member is invalid.

Recovery is also F-07-only and typed. A `PRECOMMIT` without its matching atomic publish marker leaves the stable namespace unchanged, records `CLEANUP_PENDING`, deletes only the transaction's staging set, and records a cleanup digest; cleanup failure becomes `RETRY_PENDING` with a retry count and next-attempt freshness, never an implied publish. A `PUBLISH_MARKED` record is accepted only when the stable manifest, package index, artifact set, public-ledger projection, and marker all recompute the same candidate, board, artifact, manifest, generation, lineage, rollback, F-06, tuple, health, and freshness digests. Any missing, stale, mismatched, or replayed input emits the exact P14/P16 code from section 20, exits 78, writes no stable marker, and preserves the last-known-good predecessor. F-07 must never infer health from presence, copy partial state, or retry with a relaxed contract.

The binding that F-07 verifies, expressed only in future-ratified F-02 terms (PROVISIONAL), is: candidate ID and digest (ID-23, ID-24) equal the assembled manifest identity; `board_targets` equals the section 13.2 set; every `components.mesa_stack.artifacts[*].content_digest` and `packages[*].content_digest` equals the built and signed artifact; the manifest `DocumentIdLineage` names the predecessor stable manifest as `predecessor_id` with `generation` exactly one greater; `components.mesa_stack.rollback.previous_manifest_ids` names that predecessor and `artifact_ids` names its artifacts; all 44 ABI members and the tuple digest are present; the F-06 compliance bundle and public-ledger projection digests match; freshness and health evidence are current; and every board's boot-health profile in `components.boot_stack.boot_check_profile` lists the checks that observe renderer admission under the existing `display-ready/v1` and `userspace-ready/v1` classes. If a renderer-admission check cannot be expressed with those classes, that is an F-02 extension request, not a Mesa-local check class.

Interrupted promotion, stale health, missing F-06 approval, and rollback-lineage mismatch are explicit future fixtures FX-91 through FX-94. A consumer that observes `channel = stable` without a matching F-07 marker and attestation emits its exact code, exits 78, and does not select it. Failed health after publication returns to the predecessor only through the platform rollback contract; Mesa neither overrides the decision nor reports success. DEP-06 in section 18 records the unresolved F-06/F-07 transaction, compliance-bundle, and public-ledger schema as a named BLOCKED dependency. No F-07 transaction or recovery implementation exists in this docs-only correction.

## 16. Rollback, quarantine, and incident handling

Updates write a complete new platform tuple to inactive versioned storage, verify signatures and hashes, evaluate the section 8 tuple, and select it once. Boot-health and first-session checks record the selected tuple and success marker. A failed boot, compositor health check, required graphics test, or reset recovery returns to the last-known-good tuple according to the platform rollback contract. Rollback is atomic across Mesa, kernel, firmware, boot artifacts, and required userspace. The prior tuple remains available until the successor has passed promotion and the board's retention policy permits garbage collection.

An incident response stops promotion, quarantines affected artifacts and board/profile records, preserves every typed identity and evidence digest, classifies the cause (source drift, provenance, ABI mismatch, profile mismatch, compiler correctness, runtime corruption, reset failure, compositor regression, performance/thermal regression, packaging, signature, or lab fault), reproduces on a disposable target where safe, publishes a correction or rollback decision with requalification scope, and never marks a failed rollback or unknown recovery outcome as success. The rollback exercise itself is a required release test: install the predecessor, update to the candidate, inject or reproduce the failure, observe automatic fallback, verify macOS and unrelated platform state are untouched, and record the final active manifest ID and digest.

## 17. Future-chip intake

New Apple announcements enter intake within one business day, but an announcement does not change the supported set. Intake is: create or update the exact board record through Q-00 and the registry; acquire physical hardware before any physical, reset, performance, or conformance claim; obtain the kernel/DTB/firmware inputs and the opaque boot envelope through the canonical manifest; request an AGX profile extension row (REQ-AGX-01 through REQ-AGX-12) for the exact board, which raises `max_gpu_generation` only when ratified; run identity, tuple, and profile admission with unknown or ambiguous identity failing closed; identify generation-specific shader, memory, synchronization, display, and reset work without inferring capabilities from the closest prior generation; build reproducibly on two builders; run every section 10 through 13 matrix; record per-board evidence and residuals; and publish only the lifecycle state supported by evidence through F-07. The intake record lists remaining work so that "not yet qualified" is observable and no branch, package repository, or marketing page silently expands support.

## 18. PROGRAM dependency and handoff ledger

Every row is mechanically typed. `artifact_id` is not a description, `content_or_payload_digest` has the exact preimage shown, freshness is checked against TP-07, scope is closed, the named consumer is the only consumer that can close the row, and the rejection field is exactly `code;path;phase;decision;process_result`. The F-02 candidate is explicitly rejected, F-03 is not ratified, and the F-06/F-07 artifacts remain BLOCKED.

| ID | Slice | Producer | Artifact/document ID | Content or payload digest preimage | Freshness/expiry | Exact scope | Consumer | Owner | Due before | Rejection code;path;phase;decision;process result |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DEP-01 | F-02 | `omarchy-apple-platform` | `F02-SCHEMASET-V1` and `F02-GENERATED-BINDING-V1` | `sha256(CONCAT(ASCII("omarchy-f02-artifact/v1"),0x00,JCS({artifact_id,payload,generated_binding_metadata})))` | Must be ratified and unexpired at every use; ID-03 byte-equal to local lock | All payloads, generated binding, AGX profile extension | Builder and admission guard | F-02 owner and coordinator | G-01 | `MESA_E_DEP01_F02_UNRATIFIED;$.dependency.F02;P02;HOLD;exit=79` |
| DEP-02 | F-03 | `omarchy-apple-platform` | `F03-TRUST-CONTEXT-V1` | `sha256(CONCAT(ASCII("omarchy-f03-trust/v1"),0x00,JCS({key_set_digest,revocation_epoch,authority_bindings,expires_at})))` | `expires_at` and revocation epoch checked for every signature | All roles and keys used by this lane | Both consumers | F-03 owner | G-01 | `MESA_E_DEP02_F03_TRUST_CONTEXT;$.dependency.F03;P02;REJECT;exit=78` |
| DEP-03 | F-04 | `omarchy-apple-platform` | `F04-BUILD-RECIPE-V1` | `sha256(CONCAT(ASCII("omarchy-build-recipe/v1"),0x00,JCS(complete BuildRecipe/v1)))` | Candidate-bound and unexpired before cache admission and each build | Sections 7.3 to 9, all 44 ABI build inputs | Candidate builder | F-04 owner | G-01 | `MESA_E_DEP03_F04_RECIPE;$.dependency.F04;P09;REJECT;exit=78` |
| DEP-04 | F-05 | `omarchy-apple-platform` | `F05-CANDIDATE-V1` | `sha256(CONCAT(ASCII("omarchy-f05-candidate/v1"),0x00,JCS({candidate_id,candidate_digest,tuple_digest,artifact_set_digest})))` | One candidate transaction and TP-07 expiry | Sections 8, 9, 15 and exact board target set | F-07 candidate closure | F-05 owner | G-02 | `MESA_E_DEP04_F05_CANDIDATE;$.dependency.F05;P13;NO_PUBLISH;publish=NONE;exit=78` |
| DEP-05 | F-06 | `omarchy-apple-platform` | `F06-COMPLIANCE-BUNDLE-V1` | `sha256(CONCAT(ASCII("omarchy-f06-compliance/v1"),0x00,JCS({candidate_digest,license_inventory_digest,notice_digest,source_offer_digest,approval})))` | Per candidate; approval and expiry current at promotion | Every Mesa artifact and license/notice/source-offer result | F-07 promotion terminal | F-06 owner | G-01 and stable promotion | `MESA_E_DEP05_F06_COMPLIANCE;$.dependency.F06;P14;NO_PUBLISH;publish=NONE;exit=78` |
| DEP-06 | F-07 | `omarchy-apple-platform` | `F07-PROMOTION-TRANSACTION-V1` | `sha256(CONCAT(ASCII("omarchy-f07-promotion/v1"),0x00,JCS(complete F07PromotionTransaction/v1)))` | Per promotion; freshness, health, and recovery attempt checked atomically | F-07 precommit, marker, cleanup, retry, stable namespace, ledger | F-07 sole stable writer | F-07 owner | Any stable promotion | `MESA_E_DEP06_F07_UNRESOLVED;$.dependency.F07;P14;HOLD;exit=79` |
| DEP-07 | P-03 | `omarchy-mac` | `P03-ARM-PARITY-V1` | `sha256(CONCAT(ASCII("omarchy-p03-parity/v1"),0x00,JCS({revision,scope,queue,disposition})))` | Per promotion and current to TP-07 | Product integration parity for the candidate board set | F-07 closure | P-03 owner | Any stable promotion | `MESA_E_DEP07_P03_STALE;$.dependency.P03;P14;NO_PUBLISH;publish=NONE;exit=78` |
| DEP-08 | K-02 | `linux-omarchy` | `K02-M1M2-ABI-V1` | `sha256(CONCAT(ASCII("omarchy-k02-abi/v1"),0x00,JCS({board_set,tuple_member_digest_set})))`, where `tuple_member_digest_set = JCS(sorted({ABI-03,ABI-04,ABI-05,ABI-06,ABI-07,ABI-08,ABI-09,ABI-10,ABI-11,ABI-12,ABI-13,ABI-20}))` | Per candidate and boot session | Every exact M1/M2 board | ABI builder and admission guard | K-02 owner | G-02 | `MESA_E_DEP08_K02_ABI;$.dependency.K02;P10;REJECT;exit=78` |
| DEP-09 | K-03 | `linux-omarchy` | `K03-M3-ABI-V1` | `sha256(CONCAT(ASCII("omarchy-k03-abi/v1"),0x00,JCS({board_id,tuple_member_digest_set})))`, where `tuple_member_digest_set = JCS(sorted({ABI-03,ABI-04,ABI-05,ABI-06,ABI-07,ABI-08,ABI-09,ABI-10,ABI-11,ABI-12,ABI-13,ABI-20}))` | Per candidate and boot session | Every exact M3 board | ABI builder and admission guard | K-03 owner | G-03 | `MESA_E_DEP09_K03_ABI;$.dependency.K03;P10;REJECT;exit=78` |
| DEP-10 | K-04 | `linux-omarchy` | `K04-M4-ABI-V1` | `sha256(CONCAT(ASCII("omarchy-k04-abi/v1"),0x00,JCS({board_id,tuple_member_digest_set})))`, where `tuple_member_digest_set = JCS(sorted({ABI-03,ABI-04,ABI-05,ABI-06,ABI-07,ABI-08,ABI-09,ABI-10,ABI-11,ABI-12,ABI-13,ABI-20}))` | Per candidate and boot session | Every exact M4 board | ABI builder and admission guard | K-04 owner | G-04 | `MESA_E_DEP10_K04_ABI;$.dependency.K04;P10;REJECT;exit=78` |
| DEP-11 | K-05 | `linux-omarchy` | `K05-A18M5-ABI-V1` | `sha256(CONCAT(ASCII("omarchy-k05-abi/v1"),0x00,JCS({board_id,tuple_member_digest_set})))`, where `tuple_member_digest_set = JCS(sorted({ABI-03,ABI-04,ABI-05,ABI-06,ABI-07,ABI-08,ABI-09,ABI-10,ABI-11,ABI-12,ABI-13,ABI-20}))` | Per candidate and boot session | Every exact A18 Pro and M5 board | ABI builder and admission guard | K-05 owner | G-05 | `MESA_E_DEP11_K05_ABI;$.dependency.K05;P10;REJECT;exit=78` |
| DEP-12 | K-06 | `linux-omarchy` | `K06-M6-ABI-V1` | `sha256(CONCAT(ASCII("omarchy-k06-abi/v1"),0x00,JCS({board_id,tuple_member_digest_set})))`, where `tuple_member_digest_set = JCS(sorted({ABI-03,ABI-04,ABI-05,ABI-06,ABI-07,ABI-08,ABI-09,ABI-10,ABI-11,ABI-12,ABI-13,ABI-20}))` | Per candidate and boot session after hardware exists | Every exact M6 board | ABI builder and admission guard | K-06 owner | G-05 M6 and Q-08 | `MESA_E_DEP12_K06_ABI;$.dependency.K06;P10;REJECT;exit=78` |
| DEP-13 | Q-00 | `omarchy-apple-platform` | `Q00-BOARD-INTAKE-V1` | `sha256(CONCAT(ASCII("omarchy-q00-intake/v1"),0x00,JCS({dataset_digest,contradiction_ledger_digest,revision})))` | Current registry revision and TP-07 expiry | Every board and chip ID named by this lane | F-02 profile generator and admission guard | Q-00 owner | G-01 profile ratification | `MESA_E_DEP13_Q00_INTAKE;$.dependency.Q00;P07;REJECT;exit=78` |
| DEP-14 | Q-01 | `omarchy-apple-platform` | `Q01-BOARD-INVENTORY-V1` | `sha256(CONCAT(ASCII("omarchy-q01-inventory/v1"),0x00,JCS({board_inventory_digest,capability_criteria_digest,profile_rows_digest})))` | Per registry revision and expiry | All qualification boards and AGX profile rows | Qualification record validator | Q-01 owner | G-02 | `MESA_E_DEP14_Q01_INVENTORY;$.dependency.Q01;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-15 | Q-02 | `omarchy-apple-platform` | `Q02-EVIDENCE-SERVICE-V1` | `sha256(CONCAT(ASCII("omarchy-q02-evidence/v1"),0x00,JCS({entry_id,content_digest,privacy_class,redaction_digest})))` | Per evidence entry and TP-07 expiry | All physical, conformance, golden, reset, and compositor evidence | Evidence ingestion and F-07 ledger projection | Q-02 owner | G-02 | `MESA_E_DEP15_Q02_EVIDENCE;$.dependency.Q02;P11;REJECT;exit=78` |
| DEP-16 | Q-03 | Hardware lab | `Q03-UNIT-INVENTORY-V1` | `sha256(CONCAT(ASCII("omarchy-q03-units/v1"),0x00,JCS({profile_id,unit_pseudonyms,recovery_certificates_digest})))` | Per profile and before each qualification run | At least two independently serialized units per exact profile | Lab controller | Lab owner | G-02 | `MESA_E_DEP16_Q03_UNITS;$.dependency.Q03;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-17 | Q-04 | Hardware lab | `Q04-M1M2-QUALIFICATION-V1` | `sha256(CONCAT(ASCII("omarchy-q04-record/v1"),0x00,JCS({document_id,schema,payload_type,payload_version,schema_set_digest,payload})))` | Record `expires_at` and candidate-bound | Every exact M1/M2 board | F-07 qualification closure | Lab owner | F-07 for M1/M2 | `MESA_E_DEP17_Q04_RECORD;$.dependency.Q04;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-18 | Q-05 | Hardware lab | `Q05-M3-QUALIFICATION-V1` | `sha256(CONCAT(ASCII("omarchy-q05-record/v1"),0x00,JCS({document_id,schema,payload_type,payload_version,schema_set_digest,payload})))` | Record `expires_at` and candidate-bound | Every exact M3 board | F-07 qualification closure | Lab owner | F-07 for M3 | `MESA_E_DEP18_Q05_RECORD;$.dependency.Q05;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-19 | Q-06 | Hardware lab | `Q06-M4-QUALIFICATION-V1` | `sha256(CONCAT(ASCII("omarchy-q06-record/v1"),0x00,JCS({document_id,schema,payload_type,payload_version,schema_set_digest,payload})))` | Record `expires_at` and candidate-bound | Every exact M4 board | F-07 qualification closure | Lab owner | F-07 for M4 | `MESA_E_DEP19_Q06_RECORD;$.dependency.Q06;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-20 | Q-07 | Hardware lab | `Q07-A18M5-QUALIFICATION-V1` | `sha256(CONCAT(ASCII("omarchy-q07-record/v1"),0x00,JCS({document_id,schema,payload_type,payload_version,schema_set_digest,payload})))` | Record `expires_at` and candidate-bound | Every exact A18 Pro and M5 board | F-07 qualification closure | Lab owner | F-07 for A18/M5 | `MESA_E_DEP20_Q07_RECORD;$.dependency.Q07;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-21 | Q-08 | Hardware lab | `Q08-M6-QUALIFICATION-V1` | `sha256(CONCAT(ASCII("omarchy-q08-record/v1"),0x00,JCS({document_id,schema,payload_type,payload_version,schema_set_digest,payload})))` | Record `expires_at` and candidate-bound after hardware exists | Every exact M6 board | F-07 qualification closure | Lab owner | F-07 for M6 | `MESA_E_DEP21_Q08_RECORD;$.dependency.Q08;P12;RECORD_FAIL;record=FAIL;exit=78` |
| DEP-22 | B-02, B-05 to B-08 | Human boot-artifact owner via coordinator | `BOOT-ARTIFACT-SET-V1` | `sha256(CONCAT(ASCII("omarchy-boot-artifact-set/v1"),0x00,JCS(sorted({artifact_id,content_digest,board_id}))))`; opaque bytes are not opened here | Per manifest and boot session | Exact board cohort and ABI-23 only | TP-13, ABI tuple validator, F-07 | Human owner and coordinator | Q-04 to Q-08 respectively | `MESA_E_DEP22_BOOT_SET;$.dependency.boot_artifacts;P10;NO_PUBLISH;publish=NONE;exit=78` |

Twenty-two dependency rows are bound. DEP-01, DEP-02, DEP-05, and DEP-06 are unresolved or rejected at the pinned authorities; no dependent gate may convert them to PASS.

## 19. Gate ledger and acceptance contract

| Gate | Required evidence | Hard failure examples | Status at design time |
| --- | --- | --- | --- |
| G-01 mirror and provenance | Owned-repository check, exact commit equality, complete object set, signed or attested ref, provenance record (section 7.1) | Drift, missing object, unattested ref, mutable locator | BLOCKED on DEP-01 |
| G-01 queue | Ordered commits with parents, patch lock, queue report, range-diff, disposition, removal condition | Reordered or edited patch, stale base, hidden conflict edit | BLOCKED on DEP-01 |
| G-01 build and ABI | Recipe-derived argv, hermetic environment, two-build byte equality, complete artifact set, section 8 tuple, ABI report equality with K-01 | Network access, `auto` option, byte mismatch, missing tuple member | BLOCKED on DEP-01, DEP-03 |
| G-01 profile | Ratified AGX profile extension and generated binding (TP-12) | Any board admitted from source tables, generation extrapolation, virtual device accepted | BLOCKED on DEP-01 |
| G-01 fixtures | Every section 20 fixture executable with 0 unexpected accepts | Any fixture NOT EXECUTABLE | BLOCKED on DEP-01 |
| G-02 M1/M2 | Per-board physical records (Q-04), compiler/API, conformance, golden, performance, reset, compositor, packaging, rollback | Any required failure, missing unit, family extrapolation, false success | TODO |
| G-03 M3 | Same per exact M3 board (Q-05) | M3 inferred from M1/M2, missing topology or reset evidence | TODO |
| G-04 M4 | Same per exact M4 board (Q-06) | M4 inferred from M3, missing firmware/ABI declaration | TODO |
| G-05 A18/M5/M6 | Same per exact board (Q-07, Q-08), shipping hardware | Announcement treated as support, skipped required feature | TODO |

Stable promotion is structurally blocked unless every G-01 gate passes and the applicable G-02 through G-05 records contain every required result, and even then only F-07 promotes.

## 20. Deterministic negative fixture requirements

Each fixture has one canonical input, exactly one mutation, one exact code, one exact path, one total-order phase, one closed decision, one process result, and one consumer. The code registry in this table is closed for this lane; an implementation may not substitute a broad class, free text, a missing code, or a state-only outcome. Every execution is currently `NOT IMPLEMENTED`; no fixture runner, verifier, builder, admission guard, lab controller, or F-07 terminal exists in this docs-only correction. QT-08 requires zero unexpected accepts only after all rows execute from pinned inputs.

| ID | Class | Single mutation | Exact code | Exact path | Phase | Decision | Process result | Consumer | Execution status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FX-01 | trust replay | Reuse an accepted manifest envelope's replay identity | `MESA_E_FX01_REPLAY_ID` | `$.replay_id` | P06 | REJECT | `exit=78` | F-02 verifier via TP-02 | NOT IMPLEMENTED |
| FX-02 | trust transplant | Sign a `platform-manifest/v1` payload under the `qualification-result` context | `MESA_E_FX02_CONTEXT` | `$.context` | P04 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-03 | trust transplant | Sign a qualification record with the `manifest-release` role | `MESA_E_FX03_SIGNER_ROLE` | `$.signatures[0].signer_role` | P04 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-04 | trust transplant | Present a valid manifest with an `ExpectedContext.board_id` for a different board | `MESA_E_FX04_BOARD_CONTEXT` | `$.payload.board_targets` | P07 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-05 | trust expiry | Manifest `expires_at` before TP-07 time | `MESA_E_FX05_EXPIRED` | `$.payload.expires_at` | P05 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-06 | trust seam | Pass a parsed, unverified manifest to the admission guard | `MESA_E_FX06_UNTRUSTED_VALUE` | `$` | P02 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-07 | digest substitution | Put a manifest `document_id` where `manifest_digest` is expected in a qualification binding | `MESA_E_FX07_DIGEST_TYPE` | `$.payload.manifest.manifest_digest` | P03 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-08 | digest substitution | Put the queue tip commit in `upstream_commit` and the base in `source_commit` | `MESA_E_FX08_QUEUE_COMMIT_ROLE` | `$.payload.components.mesa_stack.source` | P08 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-09 | digest substitution | Use a tree ID as `source_commit` | `MESA_E_FX09_COMMIT_OBJECT_TYPE` | `$.provenance.peeled_commit` | P08 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-10 | digest substitution | Use a patch digest as an artifact `content_digest` | `MESA_E_FX10_ARTIFACT_DIGEST_TYPE` | `$.payload.artifacts[0].content_digest` | P07 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-11 | digest substitution | Use the generated-output lock digest as `schema_set_digest` | `MESA_E_FX11_SCHEMA_DIGEST_TYPE` | `$.schema_set_digest` | P03 | REJECT | `exit=78` | TP-10 negotiation | NOT IMPLEMENTED |
| FX-12 | digest substitution | Present the kernel ABI report digest of a different kernel commit as ID-18 | `MESA_E_FX12_RELATION_DIGEST` | `$.payload.abi_tuple.ABI-20` | P10 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-13 | digest substitution | Reference a qualification record by digest inside a manifest | `MESA_E_FX13_UNKNOWN_MEMBER` | `$.payload.qualification_bindings[0].record_digest` | P03 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-14 | digest substitution | Use an `edge` manifest digest as the F-07 candidate digest for a different manifest ID | `MESA_E_FX14_CANDIDATE_BINDING` | `$.candidate.candidate_digest` | P13 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 promotion terminal | NOT IMPLEMENTED |
| FX-15 | future generation | DRM params report `gpu_generation = 15` with a ratified generation 14 profile | `MESA_E_FX15_FUTURE_GENERATION` | `$.runtime.agx.gpu_generation` | P15 | REJECT | `exit=78` | Renderer admission guard and built driver | NOT IMPLEMENTED |
| FX-16 | below minimum | DRM params report `gpu_generation = 12` | `MESA_E_FX16_BELOW_MINIMUM` | `$.runtime.agx.gpu_generation` | P15 | REJECT | `exit=78` | Renderer admission guard and built driver | NOT IMPLEMENTED |
| FX-17 | unknown chip | DRM params report `chip_id = 0x9999` | `MESA_E_FX17_UNKNOWN_CHIP` | `$.runtime.agx.chip_id` | P15 | REJECT | `exit=78` | Renderer admission guard and `agxdecode` | NOT IMPLEMENTED |
| FX-18 | virtual device | DRM driver name `virtio_gpu` | `MESA_E_FX18_VIRTUAL_DEVICE` | `$.runtime.drm.driver_name` | P15 | REJECT | `exit=78` | Renderer admission guard and built driver | NOT IMPLEMENTED |
| FX-19 | profile mismatch | Registry `chip_id_u32` differs from DRM `chip_id` by one | `MESA_E_FX19_CHIP_PROFILE_MISMATCH` | `$.payload.identity_match.macos.chip_id_u32` | P15 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-20 | profile mismatch | `num_clusters_total` differs from the profile | `MESA_E_FX20_CLUSTER_PROFILE_MISMATCH` | `$.runtime.agx.num_clusters_total` | P15 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-21 | profile mismatch | Driver advertises a Vulkan feature the profile marks `unsupported` | `MESA_E_FX21_FEATURE_PROFILE_MISMATCH` | `$.runtime.agx.feature_report_digest` | P15 | REJECT | `exit=78` | Conformance harness against the built driver | NOT IMPLEMENTED |
| FX-22 | profile unbound | Board with `gpu` present and no bound profile | `MESA_E_FX22_PROFILE_UNBOUND` | `$.payload.gpu_profile.profile_id` | P15 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-23 | ambiguous identity | Two registry boards share `chip_id_u32` and the observation matches both | `MESA_E_FX23_AMBIGUOUS_BOARD` | `$.payload.boards[1].identity_match` | P07 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-24 | mutable source | Provenance `ref_name` is a branch with no `peeled_commit` | `MESA_E_FX24_MUTABLE_REF` | `$.provenance.ref_name` | P08 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-25 | mutable source | `source.branch` added to the manifest component | `MESA_E_FX25_UNKNOWN_SOURCE_MEMBER` | `$.payload.components.mesa_stack.source.branch` | P03 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-26 | mutable source | Ref advertisement differs between upstream and mirror | `MESA_E_FX26_ADVERTISEMENT_DRIFT` | `$.provenance.ref_advertisement_digest` | P08 | REJECT | `exit=78` | Synchronization job | NOT IMPLEMENTED |
| FX-27 | reordered patch | Swap `order` of two queue entries without a new queue report | `MESA_E_FX27_PATCH_ORDER` | `$.payload.components.mesa_stack.patch_lock.entries[1].order` | P08 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-28 | edited patch | Change one byte of a patch while keeping `patch_digest` | `MESA_E_FX28_PATCH_DIGEST` | `$.payload.components.mesa_stack.patch_lock.entries[0].patch_digest` | P08 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-29 | dropped patch | Remove the last queue entry while keeping `source_commit` | `MESA_E_FX29_QUEUE_TIP` | `$.payload.components.mesa_stack.source.source_commit` | P08 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-30 | missing subproject lock | Remove one of the 42 wrap digests from the lock | `MESA_E_FX30_WRAP_LOCK` | `$.build.cache.wraps[0].content_digest` | P09 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-31 | wrap download | Wrap with a `source_url` and no cache entry, `--wrap-mode=nodownload` | `MESA_E_FX31_WRAP_CACHE` | `$.cache.entries[0].wrap_digest` | P09 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-32 | missing generated lock | Mako template changed without a `config_inputs` digest update | `MESA_E_FX32_GENERATED_INPUT` | `$.payload.components.mesa_stack.config_inputs[0].normalized_content_digest` | P09 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-33 | generator drift | Generated file differs between builders | `MESA_E_FX33_GENERATED_OUTPUT` | `$.build.comparison.generated_outputs[0].digest` | P09 | REJECT | `exit=78` | Two-build comparison | NOT IMPLEMENTED |
| FX-34 | ABI one-field | ABI-03 kernel release string differs | `MESA_E_FX34_ABI03` | `$.payload.abi_tuple.ABI-03` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-35 | ABI one-field | ABI-04 kernel commit differs | `MESA_E_FX35_ABI04` | `$.payload.abi_tuple.ABI-04` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-36 | ABI one-field | ABI-05 one config input digest differs | `MESA_E_FX36_ABI05` | `$.payload.abi_tuple.ABI-05` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-37 | ABI one-field | ABI-06 UAPI header digest differs | `MESA_E_FX37_ABI06` | `$.payload.abi_tuple.ABI-06` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-38 | ABI one-field | ABI-07 one `drm_asahi_params_global` field renamed in the report | `MESA_E_FX38_ABI07` | `$.payload.abi_tuple.ABI-07` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-39 | ABI one-field | ABI-08 `abi_contract_id` differs | `MESA_E_FX39_ABI08` | `$.payload.abi_tuple.ABI-08` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-40 | ABI one-field | ABI-09 one DTB artifact digest differs | `MESA_E_FX40_ABI09` | `$.payload.abi_tuple.ABI-09` | P10 | REJECT | `exit=78` | Admission guard | NOT IMPLEMENTED |
| FX-41 | ABI one-field | ABI-10 `dt_schema.schema_digest` differs | `MESA_E_FX41_ABI10` | `$.payload.abi_tuple.ABI-10` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-42 | ABI one-field | ABI-11 firmware artifact version differs | `MESA_E_FX42_ABI11` | `$.payload.abi_tuple.ABI-11` | P10 | REJECT | `exit=78` | Admission guard | NOT IMPLEMENTED |
| FX-43 | ABI one-field | ABI-12 firmware schema version differs | `MESA_E_FX43_ABI12` | `$.payload.abi_tuple.ABI-12` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-44 | ABI one-field | ABI-13 firmware/kernel relation `evidence_digest` differs | `MESA_E_FX44_ABI13` | `$.payload.abi_tuple.ABI-13` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-45 | ABI one-field | ABI-14 `upstream_commit` differs from the verified mirror | `MESA_E_FX45_ABI14` | `$.payload.abi_tuple.ABI-14` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-46 | ABI one-field | ABI-15 patch lock digest differs | `MESA_E_FX46_ABI15` | `$.payload.abi_tuple.ABI-15` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-47 | ABI one-field | ABI-16 recipe digest differs | `MESA_E_FX47_ABI16` | `$.payload.abi_tuple.ABI-16` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-48 | ABI one-field | ABI-17 one artifact digest differs from the built file | `MESA_E_FX48_ABI17` | `$.payload.abi_tuple.ABI-17` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-49 | ABI one-field | ABI-18 one package version differs | `MESA_E_FX49_ABI18` | `$.payload.abi_tuple.ABI-18` | P10 | REJECT | `exit=78` | Admission guard | NOT IMPLEMENTED |
| FX-50 | ABI one-field | ABI-19 feature-set digest differs | `MESA_E_FX50_ABI19` | `$.payload.abi_tuple.ABI-19` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-51 | ABI one-field | ABI-20 relation owned by `mesa-stack` instead of `linux-kernel` | `MESA_E_FX51_ABI20` | `$.payload.abi_tuple.ABI-20` | P10 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-52 | ABI one-field | ABI-21 package architecture `arm64` where `aarch64` is bound | `MESA_E_FX52_ABI21` | `$.payload.abi_tuple.ABI-21` | P10 | REJECT | `exit=78` | Builder | NOT IMPLEMENTED |
| FX-53 | ABI one-field | ABI-22 `previous_manifest_ids` names a manifest that never verified | `MESA_E_FX53_ABI22` | `$.payload.abi_tuple.ABI-22` | P10 | REJECT | `exit=78` | Builder and admission guard | NOT IMPLEMENTED |
| FX-54 | ABI one-field | ABI-23 one opaque boot artifact digest differs | `MESA_E_FX54_ABI23` | `$.payload.abi_tuple.ABI-23` | P10 | REJECT | `exit=78` | Admission guard | NOT IMPLEMENTED |
| FX-55 | ABI one-field | ABI-24 schema-set digest differs in one document | `MESA_E_FX55_ABI24` | `$.payload.abi_tuple.ABI-24` | P10 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-56 | build drift | One installed file differs between builders | `MESA_E_FX56_INSTALLED_OUTPUT` | `$.build.comparison.installed_files[0].digest` | P09 | REJECT | `exit=78` | Two-build comparison | NOT IMPLEMENTED |
| FX-57 | build recipe | Option left at `auto` in the recipe | `MESA_E_FX57_AUTO_OPTION` | `$.build.recipe.options.gallium_drivers` | P09 | REJECT | `exit=78` | BuildRecipe/v1 validator | NOT IMPLEMENTED |
| FX-58 | network access | Resolver call after cache admission | `MESA_E_FX58_RESOLVER_CALL` | `$.build.network.resolver` | P09 | REJECT | `exit=78` | F-04 builder sandbox | NOT IMPLEMENTED |
| FX-59 | network access | Meson attempts a wrap download | `MESA_E_FX59_WRAP_NETWORK` | `$.build.network.wrap_download` | P09 | REJECT | `exit=78` | F-04 builder sandbox | NOT IMPLEMENTED |
| FX-60 | environment | Operator `CFLAGS` present in the build environment | `MESA_E_FX60_OPERATOR_ENV` | `$.build.environment.CFLAGS` | P09 | REJECT | `exit=78` | F-04 builder sandbox | NOT IMPLEMENTED |
| FX-61 | incomplete artifacts | SBOM missing | `MESA_E_FX61_SBOM_MISSING` | `$.payload.artifacts.sbom` | P11 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-62 | incomplete artifacts | One detached signature missing | `MESA_E_FX62_SIGNATURE_MISSING` | `$.payload.artifacts[0].signature` | P11 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-63 | incomplete artifacts | ABI report missing while packages exist | `MESA_E_FX63_ABI_REPORT_MISSING` | `$.payload.abi_report` | P11 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-64 | incomplete artifacts | Two-build comparison report missing | `MESA_E_FX64_COMPARISON_MISSING` | `$.payload.build_reports.two_build_comparison` | P11 | REJECT | `exit=78` | F-04 builder | NOT IMPLEMENTED |
| FX-65 | package mismatch | Package `content_digest` differs from the repository index | `MESA_E_FX65_PACKAGE_INDEX_DIGEST` | `$.payload.package_index.packages[0].content_digest` | P11 | REJECT | `exit=78` | P-05 install transaction | NOT IMPLEMENTED |
| FX-66 | channel mismatch | Package reachable from a `stable` namespace without an F-07 attestation | `MESA_E_FX66_STABLE_NAMESPACE` | `$.promotion.stable_namespace` | P14 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 and repository audit | NOT IMPLEMENTED |
| FX-67 | channel mismatch | Manifest `channel = stable` presented without a verifiable F-07 attestation | `MESA_E_FX67_STABLE_ATTESTATION` | `$.payload.channel` | P14 | NO_PUBLISH | `publish=NONE;exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-68 | package mismatch | Package signed by a key without the `signature_policy_id` role | `MESA_E_FX68_SIGNATURE_ROLE` | `$.signatures[0].key_id` | P04 | REJECT | `exit=78` | F-03 verifier | NOT IMPLEMENTED |
| FX-69 | qualification undercount | One unit per profile where QT-01 requires two | `MESA_E_FX69_UNIT_COUNT` | `$.qualification.thresholds.QT-01` | P12 | RECORD_FAIL | `record=FAIL;exit=78` | Q-02 lab controller | NOT IMPLEMENTED |
| FX-70 | qualification undercount | Nine update/rollback cycles where QT-06 requires ten | `MESA_E_FX70_CYCLE_COUNT` | `$.qualification.thresholds.QT-06` | P12 | RECORD_FAIL | `record=FAIL;exit=78` | Q-02 lab controller | NOT IMPLEMENTED |
| FX-71 | retry laundering | Pass whose attempt list omits an earlier failed attempt | `MESA_E_FX71_ATTEMPT_HISTORY` | `$.payload.test_results[0].attempt_history_digest` | P12 | RECORD_FAIL | `record=FAIL;exit=78` | Q-02 lab controller | NOT IMPLEMENTED |
| FX-72 | retry laundering | Failed attempt evidence entry deleted | `MESA_E_FX72_EVIDENCE_REFERENCE` | `$.payload.test_results[0].evidence_ids[0]` | P07 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-73 | family extrapolation | M3 Pro record cited for an M3 Max board target | `MESA_E_FX73_BOARD_RECORD` | `$.payload.board.board_id` | P07 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-74 | family extrapolation | Manifest `board_targets` contains a board with no qualification binding | `MESA_E_FX74_BINDING_SET` | `$.payload.qualification_bindings` | P07 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-75 | software fallback | Software rasterizer result recorded as a hardware `gpu` pass | `MESA_E_FX75_SOFTWARE_FALLBACK` | `$.payload.test_results[0].gpu_execution` | P12 | RECORD_FAIL | `record=FAIL;exit=78` | Q-02 lab controller | NOT IMPLEMENTED |
| FX-76 | alternate promotion | Mesa CI writes a package into the `stable` namespace | `MESA_E_FX76_NON_F07_WRITER` | `$.promotion.writer_role` | P14 | NO_PUBLISH | `publish=NONE;exit=78` | Repository audit and F-07 | NOT IMPLEMENTED |
| FX-77 | self-promotion | Manifest with `channel = stable` signed by `ci-conformance` | `MESA_E_FX77_SELF_SIGNED_STABLE` | `$.signatures[0].signer_role` | P04 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-78 | alternate promotion | F-07 invoked with an artifact set differing from the candidate manifest by one digest | `MESA_E_FX78_ARTIFACT_CLOSURE` | `$.promotion.artifact_set_digest` | P13 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 promotion terminal | NOT IMPLEMENTED |
| FX-79 | self-promotion | Manifest lineage `generation` skips a value | `MESA_E_FX79_LINEAGE_GENERATION` | `$.payload.lineage.generation` | P07 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-80 | compatible string unknown | Runtime compatible is `apple,agx-unknown` and all other DRM/profile fields match | `MESA_E_FX80_AGX_COMPAT_UNKNOWN` | `$.payload.observation.device_tree.compatible` | P15 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-81 | compatible string future | Runtime compatible is a syntactically valid future `apple,agx-*` value absent from the ratified allowlist | `MESA_E_FX81_AGX_COMPAT_FUTURE` | `$.payload.observation.device_tree.compatible` | P15 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-82 | compatible string cross-board | Runtime compatible is allowlisted for another board/profile while the local board and DTB fields otherwise match | `MESA_E_FX82_AGX_COMPAT_PROFILE` | `$.payload.gpu_profile.runtime_compatible_allowlist` | P15 | REJECT | `exit=78` | Renderer admission guard | NOT IMPLEMENTED |
| FX-83 | libdrm mismatch | Substitute another libdrm source/package identity while retaining the Mesa UAPI header digest | `MESA_E_FX83_LIBDRM_IDENTITY` | `$.runtime.libdrm.source_identity` | P10 | REJECT | `exit=78` | ABI builder and admission guard | NOT IMPLEMENTED |
| FX-84 | syncobj mismatch | Change syncobj timeline, wait, reset, or handle-lifetime semantics while retaining the DRM header digest | `MESA_E_FX84_SYNCOBJ_SEMANTICS` | `$.runtime.syncobj.uapi_semantics_digest` | P10 | REJECT | `exit=78` | ABI builder and admission guard | NOT IMPLEMENTED |
| FX-85 | stale report reuse | Reuse a prior candidate's conformance report with a current artifact and unchanged report digest | `MESA_E_FX85_STALE_REPORT` | `$.payload.qualification_reports.report_set_digest` | P11 | REJECT | `exit=78` | Q-02 evidence validator | NOT IMPLEMENTED |
| FX-86 | hidden pipe status | Run `producer` piped to `tee` with producer exit 1 and aggregate status 0 | `MESA_E_FX86_PIPE_STATUS` | `$.build.commands[0].components[0].exit_status` | P09 | REJECT | `exit=78` | F-04 command runner | NOT IMPLEMENTED |
| FX-87 | incomplete conformance | CTS report omits a required result or evidence digest | `MESA_E_FX87_CTS_INCOMPLETE` | `$.payload.qualification_reports.cts` | P11 | REJECT | `exit=78` | Q-02 evidence validator | NOT IMPLEMENTED |
| FX-88 | incomplete conformance | deqp report omits a required result or evidence digest | `MESA_E_FX88_DEQP_INCOMPLETE` | `$.payload.qualification_reports.deqp` | P11 | REJECT | `exit=78` | Q-02 evidence validator | NOT IMPLEMENTED |
| FX-89 | incomplete conformance | glslangValidator report omits a required invocation or evidence digest | `MESA_E_FX89_GLSLANG_INCOMPLETE` | `$.payload.qualification_reports.glslangValidator` | P11 | REJECT | `exit=78` | Q-02 evidence validator | NOT IMPLEMENTED |
| FX-90 | incomplete conformance | vkcube report omits a required topology, result, or evidence digest | `MESA_E_FX90_VKCUBE_INCOMPLETE` | `$.payload.qualification_reports.vkcube` | P11 | REJECT | `exit=78` | Q-02 evidence validator | NOT IMPLEMENTED |
| FX-91 | interrupted promotion | Precommit exists but the atomic publish marker is absent after interruption | `MESA_E_FX91_INTERRUPTED_PROMOTION` | `$.promotion.atomic_publish_marker` | P14 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 recovery terminal | NOT IMPLEMENTED |
| FX-92 | stale health | Promotion health evidence is expired or belongs to the predecessor rather than the candidate | `MESA_E_FX92_STALE_HEALTH` | `$.promotion.health_evidence_digest` | P16 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 recovery terminal | NOT IMPLEMENTED |
| FX-93 | missing F-06 approval | Promotion omits the current F-06 compliance bundle approval | `MESA_E_FX93_F06_APPROVAL` | `$.promotion.f06_compliance_bundle_digest` | P14 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 promotion terminal | NOT IMPLEMENTED |
| FX-94 | rollback-lineage mismatch | Rollback predecessor has a different lineage digest or generation than the candidate transaction | `MESA_E_FX94_ROLLBACK_LINEAGE` | `$.promotion.rollback.lineage_digest` | P14 | NO_PUBLISH | `publish=NONE;exit=78` | F-07 promotion terminal | NOT IMPLEMENTED |
| FX-95 | cold boot undercount | Record 49 cold boots and 50 warm boots for one laptop board/profile | `MESA_E_FX95_COLD_BOOT_UNDERCOUNT` | `$.payload.test_results.QT-04C.attempt_history_digest` | P12 | RECORD_FAIL | `record=FAIL;exit=78` | Q-02 lab controller | NOT IMPLEMENTED |
| FX-96 | warm boot undercount | Record 50 cold boots and 49 warm boots for one laptop board/profile | `MESA_E_FX96_WARM_BOOT_UNDERCOUNT` | `$.payload.test_results.QT-04W.attempt_history_digest` | P12 | RECORD_FAIL | `record=FAIL;exit=78` | Q-02 lab controller | NOT IMPLEMENTED |
| FX-97 | simultaneous fault ordering | Add an unknown member and wrong signer while permuting object member order; the unknown-member phase must win identically | `MESA_E_FX97_PHASE_ORDER` | `$.payload.unknown_member` | P03 | REJECT | `exit=78` | F-02 verifier | NOT IMPLEMENTED |
| FX-98 | absent tool | Remove the locked `meson` or `ninja` binary while retaining its toolchain entry | `MESA_E_FX98_ABSENT_TOOL` | `$.build.toolchain.entries[0].binary_digest` | P09 | REJECT | `exit=78` | F-04 recipe validator | NOT IMPLEMENTED |
| FX-99 | stale cache | Present an expired offline cache entry with matching historical bytes and digest | `MESA_E_FX99_STALE_CACHE` | `$.cache.entries[0].freshness` | P09 | REJECT | `exit=78` | F-04 cache admission | NOT IMPLEMENTED |

Ninety-nine hostile fixtures are required: the original FX-01 through FX-79 rows are preserved and FX-80 through FX-99 add the mandatory compatible-string, libdrm, syncobj, stale-report, hidden-pipe, absent-tool, option, network, stale-cache, incomplete-report, promotion, health, F-06, rollback-lineage, cold-boot, warm-boot, and simultaneous-fault cases. Their current census is 0 PASS / 99 NOT IMPLEMENTED. A future executable run must report 0 unexpected accepts; this document does not claim that result.

## 21. Residuals and NOT IMPLEMENTED empirical risks

Every row is an open residual with an owner, a named consumer, an acceptance artifact/digest, a consumer-visible rejection behavior, and a gate it must be closed before. None is a design accomplishment.

| ID | Residual | Evidence | Consumer | Acceptance artifact/digest | Owner | Due before | Consumer-visible rejection behavior |
| --- | --- | --- | --- | --- | --- | --- | --- |
| RES-01 | `gpu_generation >= 14` extrapolation admits any future generation | `src/asahi/lib/agx_device.c:662-670`; no upper bound; `agx_device.c:544` lower bound is assertion-only | Renderer admission guard | `F02-AGX-PROFILE-V1`; `sha256(JCS(profile))` | Mesa lane after REQ-AGX-09 | G-02 | `MESA_E_RES01_GENERATION_BOUND;$.runtime.agx.gpu_generation;P15;REJECT;exit=78` |
| RES-02 | Unknown chip IDs decode as generation 13 G | `src/asahi/lib/decode.c:951-959` shared `default:` arm; `decode.c:974` default call | Renderer admission guard and `agxdecode` | `MESA-AGX-GENERATED-BINDING-V1`; ID-26 output digest | Mesa lane | G-02 | `MESA_E_RES02_UNKNOWN_CHIP;$.runtime.agx.chip_id;P15;REJECT;exit=78` |
| RES-03 | Vulkan features and extensions advertised largely unconditionally | `src/asahi/vulkan/hk_physical_device.c:234-270` and `48-232` | Vulkan capability consumer and qualification harness | `ABI-19`; `sha256(JCS(sorted(feature,disposition,value)))` | Mesa lane | G-02 | `MESA_E_RES03_FEATURE_DISPOSITION;$.runtime.agx.feature_report_digest;P15;REJECT;exit=78` |
| RES-04 | `virtio_gpu` accepted as an Asahi native context | `src/asahi/lib/agx_device.c:519-527`, `46-47`, `313-317`; the cited range `1222-1229` does not exist in the 960-line file | Renderer admission guard | `MESA-AGX-ADMISSION-GUARD-V1`; guard binary content digest | Mesa lane | G-02 | `MESA_E_RES04_VIRTUAL_DEVICE;$.runtime.drm.driver_name;P15;REJECT;exit=78` |
| RES-05 | 79 hostile cases have 0 executable rejections; this revision adds cases FX-80 through FX-99 and keeps every fixture NOT EXECUTABLE | Coordinator review and section 20 | F-02 verifier, F-04 builder, admission guard, Q-02 controller, and F-07 terminal | `MESA-FIXTURE-CATALOG-V2`; SHA-256 of the normalized section 20 catalog | F-02, F-04, F-05, F-07, Q-02 owners with Mesa lane | G-01 | `MESA_E_RES05_FIXTURE_NOT_EXECUTABLE;$.fixtures.execution_status;P11;HOLD;exit=79` |
| RES-06 | No target validator, schema, fixture runner, generated binding, consumer guard, target CI, build pipeline, or physical evidence exists | PROGRAM section 12 and this document | Coordinator gate owner for the consuming slice | `MESA-G01-CLOSURE-V1`; digest of all required executable artifacts | F-02 through F-07 owners; Mesa lane for the guard | G-01 | `MESA_E_RES06_MISSING_IMPLEMENTATION;$.gate.G-01.required_artifacts;P13;HOLD;exit=79` |
| RES-07 | Meson, Ninja, Vulkan CTS, `glslangValidator`, `vkcube`, and `deqp-runner` were unavailable; no build directory exists | Coordinator review; no build performed by this lane | F-04 builder and Q-02 evidence controller | `F04-TOOLCHAIN-LOCK-V1`; `ID-22` plus tool binary digests | F-04 owner | G-01 | `MESA_E_RES07_TOOLING_BLOCK;$.build.toolchain.entries;P09;HOLD;exit=79` |
| RES-08 | Device UUID derived from generation, variant, and revision only | `src/asahi/lib/agx_device.c:895-901` | Renderer admission guard | `ABI-02`; `sha256(JCS({profile_id,profile_digest,board_id}))` | Mesa lane | G-02 | `MESA_E_RES08_UUID_IDENTITY;$.runtime.agx.device_uuid;P15;REJECT;exit=78` |
| RES-09 | Python interpreter and Mako detection at configure time | `meson.build:1176-1226` | F-04 recipe validator | `F04-BUILD-RECIPE-V1`; ID-21 | F-04 recipe owner with Mesa lane | G-01 | `MESA_E_RES09_TOOL_AUTODETECT;$.build.recipe.toolchain;P09;REJECT;exit=78` |
| RES-10 | AGX profile extension not ratified; TP-12 has no producer | Section 5.2 | F-02 generator and renderer admission guard | `F02-AGX-PROFILE-V1`; ID-25 and ID-26 | F-02 owner and coordinator | G-01 | `MESA_E_RES10_PROFILE_UNRATIFIED;$.dependency.F02.gpu_profile;P02;HOLD;exit=79` |
| RES-11 | F-06/F-07 handoff schema unresolved and its artifacts are blocked | DEP-06 | F-07 promotion terminal | `F06-COMPLIANCE-BUNDLE-V1` and `F07-PROMOTION-TRANSACTION-V1`; their future payload digests | F-06 and F-07 owners | Any stable promotion | `MESA_E_RES11_PROMOTION_HANDOFF;$.dependency.F07;P14;HOLD;exit=79` |
| RES-12 | Builder rejection codes are not yet reconciled with the future F-04/F-05 vocabulary | Section 9.6 | F-04 builder and F-05 candidate assembler | `F04-ERROR-VOCABULARY-V1`; digest of the ratified code mapping | F-04 and F-05 owners | G-01 | `MESA_E_RES12_ERROR_VOCABULARY;$.build.error.code;P09;HOLD;exit=79` |
| RES-13 | Upstream ref signing evidence is not observed in the frozen blob-filtered source; the current mirror also has provenance drift | Section 7.1 `signer_evidence` and section 6.3 | Synchronization job and F-07 provenance closure | `MESA-SOURCE-PROVENANCE-V2`; ID-09A and ID-09B digests | Mesa lane synchronization job | G-01 | `MESA_E_RES13_PROVENANCE_HOLD;$.provenance.ref_advertisement_digest;P08;HOLD;exit=79` |
| RES-14 | No physical board, unit, or qualification record exists for any AGX profile | Section 13 | Q-02 lab controller and F-07 qualification closure | `Q04-Q08-QUALIFICATION-SET-V1`; sorted record payload digests | Q-03 through Q-08 owners | G-02 | `MESA_E_RES14_NO_PHYSICAL_EVIDENCE;$.qualification.test_results;P12;RECORD_FAIL;record=FAIL;exit=78` |

Fourteen residuals are open. Their acceptance artifacts and consumers are explicit, but none has closed evidence at this design tip.

## 22. Reporting format

Each implementation or review report must include: repository, branch, exact base and tip commit IDs, and verified authoritative commit; changed-file census and confirmation that no graphics code or canonical program file was modified by the design lane; gate-by-gate PASS, FAIL, BLOCKED, or NOT RUN output with commands and evidence digests; failure census with counts and causes; mirror drift status, queue disposition, section 8 tuple by member ID, builder identities, artifact digests, and rollback predecessor; physical boards and units actually tested, never inferred coverage; deviations, unsupported features, open upstream work, recovery limits, and residual IDs from section 21; and a statement that nothing is DONE unless the coordinator has independently integrated and promoted the required slice.

At design time the residuals are deliberate: no gate is implemented by this documentation change; no physical board was qualified by this lane; no conformance result is claimed; no generation was promoted; F-02 remains rejected and every dependency on it is provisional; the canonical platform schemas remain the source of truth; and the opaque boot artifact boundary remains fenced under the coordinator ruling.
