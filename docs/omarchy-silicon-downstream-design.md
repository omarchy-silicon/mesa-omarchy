# Omarchy Silicon Mesa/AGX downstream design

Status: DESIGN NOTE, correction round 1. G-01 through G-05 are BLOCKED or TODO; nothing in this document is DONE, implemented, built, qualified, compatible, supported, or release-ready.

Owner: Mesa/AGX downstream design lane (DESIGN-SWE)

Repository: `omarchy-silicon/mesa-omarchy`

Branch: `factory/design-mesa-release`

Frozen source evidence base: freedesktop.org Mesa `main` and `origin/main` both observed at `d870cef8b7c8a4a11edc669669c9f18ae402314a` on 2026-09-02 (`VERSION` file reads `26.3.0-devel`). Every `src/`, `include/`, `subprojects/`, and `meson.options` citation below refers to that tree and was read without modification.

This document is a design contract for the Mesa component of the Omarchy Silicon platform program. It does not modify Mesa graphics code, establish hardware support, claim conformance, or promote any board or release. A build, recognized GPU, booting compositor, or passing subset of tests is evidence for a gate only; it is never a support claim by itself.

## 0. Correction status and reading rules

This revision answers the coordinator gate on tip `6221cee01fecb4f452bd56d4cb6ebc8db1391724` (nine blocking findings, hostile census 0 PASS / 64 NOT EXECUTABLE). It is a design-only correction. No schema, validator, fixture, generated binding, consumer guard, CI job, build directory, package, or physical evidence was created by this change.

The authoritative program input is `PROGRAM.md` in `omarchy-apple-platform` at ratified commit `58302d148f0e8b855578f9aa518ff1c5eb48c515`. The only F-02 text available to this lane is the rejected candidate at `omarchy-apple-platform` commit `c315c7e79928d0041deb582bed79a61074361b21`, file `docs/design/platform-schema.md`. PROGRAM section 16 and its decision log freeze that tip after three rejected correction rounds pending an owner exception. Therefore:

- Every `Trusted<T>`, `AuthorityRoleBinding`, `ExpectedContext`, JSON path, failure code, grammar name, and role name cited below is marked PROVISIONAL. It is a dependency on a future ratified F-02/F-03 contract, not a statement that the rejected candidate is authority.
- This lane does not copy, restate, or extend that candidate's schema text as a local schema. Where this lane needs something the candidate lacks, it records an F-02-owned extension requirement and marks the consuming gate BLOCKED.
- If ratified F-02/F-03 rename or reshape any cited item, the generated bindings are the only source of the new names; this document is then corrected in a new round. No Mesa code may be written against the provisional names.
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

### 3.3 Anti-transplant, expiry, replay, and schema-set equality

For every payload the consumer requires, in F-02 phase order, all of the following before any field is read: envelope field equality with the payload common fields; signature preimage recomputation over the closed anti-transplant object (`document_id`, `schema`, `payload_type`, `payload_version`, `schema_set_digest`, `domain`, `context`); signer role and key resolved through TP-05; `issued_at`/`expires_at` evaluated against TP-07; replay identity checked against the durable reservation; and byte equality of `schema_set_digest` with the consumer's own input lock via TP-10. A manifest signed under another domain or context, a qualification record whose `manifest_digest` differs from the verified manifest payload digest, a registry whose `board_registry_digest` differs from the manifest's `board_registry_digest`, or any document whose `schema_set_digest` differs from the local lock is rejected before its content is used.

### 3.4 Deterministic failure behavior

| Failure observed at the seam | PROVISIONAL F-02 code | Consumer behavior in this lane |
| --- | --- | --- |
| Envelope, payload, or signature transplant across type, domain, context, role, or key | `SIGNATURE_CONTEXT_MISMATCH` or `TRUST_FAILURE` | No `Trusted<T>`; builder job exits non-zero; admission guard leaves the last-known-good graphics tuple active and emits a redacted error naming the JSON path only |
| Expired document or replayed nonce/replay ID | `EXPIRY_OR_REPLAY_FAILURE`, `MANIFEST_EXPIRY_FAILURE`, `REGISTRY_EXPIRY_FAILURE`, `QUALIFICATION_EXPIRY_FAILURE` | Same as above; no cached earlier verification is reused |
| Schema-set digest differs from the local input lock | `CROSS_DOCUMENT_MISMATCH` at `$.schema_set_digest` | Reject; no compatible-looking schema set is substituted |
| Binding, parser, API, or output digest not equal to `consumer_schema_set` | `BINDING_INTEGRITY_FAILURE` | Reject before parsing any payload |
| Manifest projection differs from component tree | `MANIFEST_AUTHORITY_CONFLICT` | Reject the manifest as a whole |
| Board not in `board_targets`, or observation predicates differ from the registry | `CROSS_DOCUMENT_MISMATCH` | No renderer selection; board is reported as not targeted |
| Qualification record references another manifest or a non-target board | `CROSS_DOCUMENT_MISMATCH` at `$.payload.manifest_digest` or `$.payload.board.board_id` | Reject; candidate cannot be assembled |
| Any local record (observation, clock, policy, capabilities) supplied raw | `TRUST_BOUNDARY_FAILURE` | Hold; no partial value is returned |

Every failure is total. There is no warning-success, degraded mode, software-renderer substitution, or retry with relaxed context.

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
| ID-08 | source repository identity | F-02 `RepositoryId` plus the immutable HTTPS URL recorded in the provenance record (section 7.1); never a branch | `components.mesa_stack.source.repository_id`, provenance `upstream_url` | ID-09, ID-10 |
| ID-09 | ref or tag object identity | Git object ID of the annotated tag or the ref advertisement entry, plus signer evidence | provenance `tag_object_id`, `ref_advertisement_digest` | ID-10, ID-11 |
| ID-10 | peeled commit identity (`git_commit`) | Git commit object ID, 40 or 64 lowercase hexadecimal | `components.mesa_stack.source.upstream_commit` (verified mirror base), `source.source_commit` (queue tip) | ID-09, ID-11, ID-12 |
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
| ID-25 | AGX profile ID and digest | F-02-owned extension (section 5); profile identity and digest over the closed profile record | BLOCKED, extension not ratified | ID-07, ID-17 |

Twenty-five identity types are defined. Recomputation rule: any digest in this table is recomputed by the consumer from its named preimage bytes; a digest that cannot be recomputed because the preimage is absent, mutable, or network-only is treated as mismatched.

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

Ten extension requirements are recorded. Until the coordinator ratifies the extension, G-01 is BLOCKED (section 19) and TP-12 has no producer.

### 5.3 Board-to-profile disposition at admission

At renderer admission the guard holds TP-01, TP-02, TP-08, TP-09, TP-10, and TP-12. It reads `drm_asahi_params_global` from the physical device, compares every REQ-AGX-02 and REQ-AGX-07 field to the generated profile constants, checks the driver name per REQ-AGX-05, checks the generation bounds per REQ-AGX-03, and checks that the observed board (TP-08) is the board bound to the profile (REQ-AGX-01). Every comparison is equality or closed-set membership. The first mismatch quarantines the device in its REQ-AGX-08 class; the guard then leaves the last-known-good tuple active and records the class and path. There is no nearest-generation fallback, no G13G default, no software-rasterizer substitution reported as graphics success, and no feature advertisement beyond the profile.

### 5.4 Downstream build binding

A future downstream patch (G-02 or later, never G-01) may compile the generated profile constants into `src/asahi/lib/agx_device.c` and `src/asahi/vulkan/hk_physical_device.c` so that chip mapping, decode parameters, and feature advertisement are table-driven from the ratified profile and reject everything else. That patch is admissible only when REQ-AGX-09 is ratified, the constants file is a digest-locked config input (ID-13), and the hostile fixtures FX-15 through FX-22 execute against the built driver. Until then, the unconditional code paths in section 5.1 remain residuals and no board is admitted.

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

The snapshot observation `d870cef8b7c8a4a11edc669669c9f18ae402314a` on 2026-09-02 is an observed synchronization input, not a claim that future mirror state, conformance, or hardware support is complete.

### 6.2 Synchronization procedure

1. Fetch authoritative refs and the GitHub mirror with pruning, accepting only the allowlisted immutable URLs, and record the ref advertisement digest (ID-09).
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

The gate applies to edge, RC, and stable. A drift warning is not an acceptable status, and a successful build cannot override this gate. A coordinator-approved mirror repair is a new auditable event recording old and new commit IDs, reason, impact, reruns, and retained failed evidence. The design never auto-merges upstream into a diverged mirror branch.

## 7. Source, patch, dependency, and generated-source provenance

### 7.1 Source provenance record

The Mesa component provenance report referenced by `components.mesa_stack.source.provenance_report_digest` is a closed record produced by the synchronization job and signed under `ci-conformance`. Its required members are:

| Member | Content | Reject when |
| --- | --- | --- |
| `upstream_url` | Immutable HTTPS URL of the authoritative repository, no query, fragment, or branch | Not on the allowlist, or any branch or moving locator present |
| `mirror_url` | Immutable HTTPS URL of the owned mirror | Owner or URL differs from the recorded value |
| `ref_name` and `tag_object_id` | The exact ref (`refs/heads/main` or `refs/tags/<name>`) and, for tags, the annotated tag object ID (ID-09) | Ref is lightweight where a signed tag is required, or the tag object is absent |
| `peeled_commit` | Commit object ID reached by peeling the ref (ID-10) | Not equal to the mirror commit |
| `tree_id` | Tree object ID of the peeled commit (ID-11) | Not equal to the tree recorded at fetch |
| `parent_ids` | Ordered parent commit IDs | Differ from the fetched commit object |
| `signer_evidence` | Upstream tag signer key fingerprint and signature bytes digest where upstream signs the ref; otherwise a coordinator-signed mirror attestation of the peeled commit under the `ci-conformance` role | Neither an upstream signature nor a coordinator attestation is present |
| `ref_advertisement_digest` | Digest of the canonicalized `ls-remote` advertisement from both remotes at fetch time | Advertisement differs between the two remotes for `refs/heads/main` |
| `prior_base` | The previous candidate's `peeled_commit`, for range-diff and rebase reports | Missing when a prior candidate exists |
| `object_set_digest` | Digest of the sorted list of commit, tree, tag, and blob object IDs materialized for the build (section 7.5) | Any object needed by the build is absent from the set |
| `fetch_clock` | TP-07 time of the fetch | Caller clock or absent |

The manifest `source` record for `components.mesa_stack` carries `source_kind = git-repository/v1`, `repository_id` (ID-08), `source_commit` (queue tip, ID-10), `upstream_commit` (peeled mirror base, ID-10), `source_digest` (digest of the sorted provenance object set), and `provenance_report_digest`. A missing member, a mutable locator, or a network-only object rejects the component.

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

The complete ABI tuple is the closed list below. Every member is a typed identity from section 4 or a value copied from a verified manifest component. One missing, unknown, or unequal member quarantines the candidate on the builder and quarantines the device at admission, in both cases before renderer selection.

| ID | Member | Source path (PROVISIONAL) | Comparison |
| --- | --- | --- | --- |
| ABI-01 | Board ID | `board_targets[*]` and TP-08 | Exact equality |
| ABI-02 | AGX profile ID and digest | TP-12 (ID-25) | Exact equality |
| ABI-03 | Kernel release string | `components.linux_kernel` report of kind `abi/v1` | Exact equality with the running kernel's release string |
| ABI-04 | Kernel source commit | `components.linux_kernel.source.source_commit` | Exact equality |
| ABI-05 | Kernel configuration digest | `components.linux_kernel.config_inputs[*].normalized_content_digest` | Exact equality per input |
| ABI-06 | UAPI header digest | Digest of `include/drm-uapi/asahi_drm.h` as built into Mesa, recorded in the ABI report; must equal the kernel's exported header digest in the K-01 ABI report | Exact equality |
| ABI-07 | DRM UAPI schema | The ABI report's closed description of `drm_asahi_params_global` (field names, widths, order), every `DRM_ASAHI_*` ioctl number, every feature bit, and `DRM_ASAHI_MAX_CLUSTERS = 64` | Exact equality of the report digest (ID-18) |
| ABI-08 | Driver interface revision | `components.linux_kernel.abi_contract_id` and the `kernel-abi/v1` relation `contract_id` | Exact equality |
| ABI-09 | DTB set artifacts | `components.dtb_set.artifacts[*].content_digest` | Exact equality per artifact |
| ABI-10 | DTB required-property set | `components.dtb_set.dt_schema` (`schema_id`, `schema_version`, `schema_digest`) and its `dt-schema/v1` report listing the property IDs the kernel needs to expose the GPU DRM device; property names are owned by K-01 and are not restated here | Exact equality of schema digest and report digest |
| ABI-11 | Firmware bundle identity | `components.firmware_bundle.artifacts[*]` (`artifact_id`, `artifact_version`, `content_digest`) | Exact equality |
| ABI-12 | Firmware schema | `components.firmware_bundle.firmware_schema` (`schema_id`, `schema_version`, `schema_digest`) and the top-level projection | Exact equality |
| ABI-13 | Firmware/kernel pairing | `firmware-schema/v1` relation owned by `firmware-bundle` with right operand `linux-kernel`, `contract_id`, `evidence_digest` | Exact equality |
| ABI-14 | Mesa upstream base and queue tip | `components.mesa_stack.source.upstream_commit`, `source.source_commit` | Exact equality |
| ABI-15 | Mesa patch lock | `components.mesa_stack.patch_lock.lock_digest` and ordered entries | Exact equality |
| ABI-16 | Mesa build profile | `components.mesa_stack.recipe_digest`, `config_inputs`, `toolchain_lock.lock_digest` | Exact equality |
| ABI-17 | Mesa generated artifacts | `components.mesa_stack.artifacts[*]` of kind `mesa-package/v1` | Exact equality per artifact |
| ABI-18 | Package set | `components.mesa_stack.packages[*]` (`package_name`, `architecture`, `version`, `content_digest`) | Exact equality |
| ABI-19 | Feature set | Profile per-feature disposition digest (REQ-AGX-06, REQ-AGX-07) recorded in the Mesa `compatibility/v1` report | Exact equality |
| ABI-20 | Kernel/Mesa relation | `kernel-abi/v1` relation owned by `linux-kernel` with right operand `mesa-stack`, `contract_id` (ID-17), `evidence_digest` (ID-18) | Exact equality, and Mesa must independently recompute ID-18 |
| ABI-21 | Package architecture relation | `package-architecture/v1` relation between `mesa-stack` and the userspace package set consumer, `aarch64` only | Exact equality |
| ABI-22 | Rollback predecessor | `components.mesa_stack.rollback` (`previous_component_ids`, `previous_manifest_ids`, `artifact_ids`, `retention_count`, `rollback_policy_digest`) | Predecessor must exist, verify, and itself have passed this tuple check |
| ABI-23 | Opaque boot artifacts | `components.boot_stack.artifacts[*].content_digest` (TP-13) | Exact equality of digests only |
| ABI-24 | Schema-set digest | ID-03 in every document and both locks | Byte equality with the local input lock |

Twenty-four tuple members are defined.

### 8.2 Compatibility classes

For each member, compatibility is one of exact match required, explicitly compatible closed set signed by the owning component and covered by tests, or incompatible. There is no implicit latest, same-chip, or backward-compatible value. The design does not use the closed-set option for any member in v1; every row above is exact equality or exact digest equality. A firmware schema change, kernel reset behavior change, UAPI struct change, or compiler/runtime change invalidates the affected candidate until the complete tuple is rebuilt and requalified.

### 8.3 ABI report content

The `abi/v1` report whose digest is ID-18 is produced twice, independently, by K-01 tooling from the kernel tree and by the Mesa builder from the Mesa tree, and the two digests must be equal. The report content is closed: the UAPI header digest (ABI-06), the `drm_asahi_params_global` layout, each ioctl name and number, each feature bit name and value, `DRM_ASAHI_MAX_CLUSTERS`, the `abi_contract_id`, and the kernel release string. Stable kernel DRM UAPI is distinguished from implementation-private interfaces; Mesa must not claim ABI compatibility because a symbol or device node exists.

### 8.4 Mismatch quarantine

On the builder, a tuple mismatch aborts candidate assembly with the failing member ID and path recorded in the build report; no artifact is emitted. On the device, the admission guard evaluates the tuple against the active kernel, DTB, firmware, and installed packages before any DRI or Vulkan ICD is selected; a mismatch leaves the previous tuple active and reports the member ID and path. Downgrading only Mesa is forbidden when ABI-14 through ABI-19 are incompatible with the active kernel or firmware.

## 9. Reproducible build and package design

### 9.1 Recipe and argv derivation

The build recipe is owned by F-04 and consumed as a pinned input; its digest is ID-21. Argument vectors are derived from the recipe as arrays, never from shell strings, environment variables, or interpolated templates. The shape is fixed; the values come only from the recipe:

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

Every option name above exists in the frozen `meson.options` or is a Meson built-in. The recipe must set every option it lists explicitly; an option left at `auto` is a recipe error, because `auto` invokes configure-time host detection. The resolved options and dependency versions from the Meson introspection output are part of the build transcript and are compared across builders. The native file is generated from the toolchain lock and names the compiler, linker, `ar`, `strip`, `pkg-config` path, and sysroot by content digest.

### 9.2 Hermetic environment

- Builder image digest, source checkout at the queue tip, and the offline cache (section 7.5) are the only inputs.
- Network is denied after cache admission by the builder sandbox; a resolver call, socket connect, or wrap download attempt is a build failure, not a warning.
- Environment is fixed to `SOURCE_DATE_EPOCH` from the queue tip commit time, `TZ=UTC`, `LC_ALL=C.UTF-8`, `LANG=C.UTF-8`, `PYTHONHASHSEED=0`, a fixed `umask`, an empty `HOME` cache, and no `MESON_*`, `CFLAGS`, `LDFLAGS`, or `PATH` inherited from the operator.
- Paths are normalized with fixed source and build directories and `-ffile-prefix-map` and `-fdebug-prefix-map` entries recorded in the recipe.
- Build caches are disabled or content-addressed and included in provenance.
- No developer GPU, untracked local patch, or mutable home-directory state is readable by the build.

### 9.3 Two-build comparison

Every candidate is built by two isolated builders from the same source, queue, recipe, cache, and toolchain digests. The following must be byte-identical: every installed file after the declared deterministic packaging step, every generated source file (ID-14), build IDs, debug and source artifacts where shipped, the SBOM, the provenance attestation body, the compiler invocation transcript, and the license inventory. Intentionally non-identical items are limited to the builder attestation wrapper and signature envelopes, each named in the recipe. Any other difference is a release-blocking failure investigated as a source, toolchain, generated-code, environment, or packaging defect. One builder's output is never selected because it boots.

### 9.4 Complete artifact set

A candidate build emits exactly: runtime packages, development packages, debug packages, source package, test tools package (`agxdecode` and selected Mesa tools when `tools` includes `asahi`), the SBOM, the source provenance report (section 7.1), the queue report (section 7.2), the build transcript, the ABI report (section 8.3), the `compatibility/v1` report carrying the feature-set digest (ABI-19), the two-build comparison report, the license inventory for F-06, and a detached signature per artifact. Every artifact is a `ComponentArtifact` of kind `mesa-package/v1` or a `ReportEntry` of kind `build/v1`, `abi/v1`, or `compatibility/v1`. A missing member rejects the candidate.

### 9.5 Signatures, repository, channel, and install transaction

- Every artifact carries a detached signature verified against the F-03 role and key named by `signature_policy_id`; Mesa holds no signing key.
- Package repository indexes are signed under F-03 roles; `Optional TrustAll`, `skipinteg`, unsigned databases, and network transport success are never authority.
- The package repository namespace is bound to the manifest `channel`; a package in an `edge` namespace that is also reachable from a `stable` namespace without an F-07 promotion is a repository-integrity failure.
- Install is a transaction owned by P-05 and the installer slices: the new Mesa packages are written to the inactive slot, every `content_digest` is verified after write, the ABI tuple (section 8) is evaluated, and only then is the slot selected once; a boot-health failure returns to the predecessor per the manifest rollback records.
- The Mesa driver is not installed into an active search path outside this transaction.

### 9.6 Deterministic builder rejection classes

The candidate builder exits with exactly one class. These are process exit classifications for the build job; they are not authenticated payload failure codes, never appear inside an F-02 document, and must be mapped one-to-one onto the F-04/F-05 rejection vocabulary when that vocabulary is ratified.

| Class | Condition |
| --- | --- |
| `MESA_BUILD_SOURCE_UNVERIFIED` | Mirror drift, missing object, unsigned or unattested ref, or provenance member missing |
| `MESA_BUILD_QUEUE_INVALID` | Patch base, order, parent, digest, or report mismatch |
| `MESA_BUILD_INPUT_UNLOCKED` | Wrap, system dependency, generator, or profile constant not in the lock or cache |
| `MESA_BUILD_NETWORK_DENIED` | Any network attempt after cache admission |
| `MESA_BUILD_ABI_MISMATCH` | Any section 8 member unequal or absent |
| `MESA_BUILD_NONREPRODUCIBLE` | Two-build comparison difference outside the named wrappers |
| `MESA_BUILD_ARTIFACT_INCOMPLETE` | Any section 9.4 member missing or unsigned |
| `MESA_BUILD_PROFILE_UNRATIFIED` | TP-12 absent; the build cannot claim any board |

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

Tests distinguish an expected unsupported feature from a regression; neither is a generic pass. Reset and hang tests run only on disposable or quarantined lab targets with a rehearsed recovery path and never against a user's installation.

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
| QT-04 | Cold/warm boot cycles for laptop profiles | 50 | PROGRAM section 9 |
| QT-05 | Attach/detach cycles per applicable port or device class | 5 | PROGRAM section 9 |
| QT-06 | Complete update/rollback cycles per profile | 10 | PROGRAM section 9 |
| QT-07 | Independent human operators per qualitative row | 2 | PROGRAM section 9 |
| QT-08 | Unexpected accepts permitted across the hostile fixture catalog | 0 | Section 20 |
| QT-09 | Retries permitted to be dropped, merged, or relabeled in an attempt history | 0 | Section 13.4 |

Nine thresholds are bound. A higher subsystem safety standard overrides these floors; none may be lowered by this lane.

### 13.4 Attempt histories and anti-laundering

Every attempt of every test on every unit is an immutable evidence entry with its own `content_digest`, timestamp, unit pseudonym, and outcome. A retry is a new entry that references the prior attempt; the prior attempt is never deleted, overwritten, merged, or reclassified. The qualification record's `test_results` entry for a row reports the final status and lists every attempt's evidence ID; a record that lists fewer attempts than the lab controller recorded, or a pass whose attempt list omits a failed attempt, fails the record. Flake is a failure class with an owner, never a filter.

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

Mesa emits candidate artifacts and evidence only: the section 9.4 artifact set, signed under F-03 roles, described by `components.mesa_stack` records inside a manifest that F-05 assembles. Mesa does not assemble manifests, does not select a channel, does not write to any package namespace, does not copy digests between namespaces, and cannot mark a board, release, or slice as anything. There is no Mesa-local promotion record, promotion command, promotion role, or promotion key. F-07 is the only stable-promotion writer and recomputes the full closure (PROGRAM sections 12.1, 13, 17) before copying an immutable candidate digest.

The binding that F-07 verifies, expressed only in ratified F-02 terms (PROVISIONAL), is: candidate ID and digest (ID-23, ID-24) equal the assembled manifest identity; `board_targets` equals the section 13.2 set; every `components.mesa_stack.artifacts[*].content_digest` and `packages[*].content_digest` equals the built and signed artifact; the manifest `DocumentIdLineage` names the predecessor stable manifest as `predecessor_id` with `generation` exactly one greater; `components.mesa_stack.rollback.previous_manifest_ids` names that predecessor and `artifact_ids` names its artifacts; and every board's boot-health profile in `components.boot_stack.boot_check_profile` lists the checks that observe renderer admission under the existing `display-ready/v1` and `userspace-ready/v1` classes. If a renderer-admission check cannot be expressed with those classes, that is an F-02 extension request, not a Mesa-local check class.

Interrupted promotion: promotion is an atomic digest copy by F-07; an interrupted copy leaves no stable manifest, and a consumer that observes a manifest with `channel = stable` but cannot verify the F-07 promotion attestation for that exact manifest digest treats it as not stable and does not select it. Failed health after promotion: `boot-health/v1` fallback (`decision = recover`, `target_slot` in the rollback set) returns to the predecessor tuple; Mesa neither overrides the decision nor reports success; the incident path in section 16 applies and F-07 closure must be recomputed before any re-promotion.

DEP-06 in section 18 records the unresolved F-06/F-07 handoff schema (promotion attestation shape, compliance bundle binding, public-ledger projection) as a named BLOCKED dependency.

## 16. Rollback, quarantine, and incident handling

Updates write a complete new platform tuple to inactive versioned storage, verify signatures and hashes, evaluate the section 8 tuple, and select it once. Boot-health and first-session checks record the selected tuple and success marker. A failed boot, compositor health check, required graphics test, or reset recovery returns to the last-known-good tuple according to the platform rollback contract. Rollback is atomic across Mesa, kernel, firmware, boot artifacts, and required userspace. The prior tuple remains available until the successor has passed promotion and the board's retention policy permits garbage collection.

An incident response stops promotion, quarantines affected artifacts and board/profile records, preserves every typed identity and evidence digest, classifies the cause (source drift, provenance, ABI mismatch, profile mismatch, compiler correctness, runtime corruption, reset failure, compositor regression, performance/thermal regression, packaging, signature, or lab fault), reproduces on a disposable target where safe, publishes a correction or rollback decision with requalification scope, and never marks a failed rollback or unknown recovery outcome as success. The rollback exercise itself is a required release test: install the predecessor, update to the candidate, inject or reproduce the failure, observe automatic fallback, verify macOS and unrelated platform state are untouched, and record the final active manifest ID and digest.

## 17. Future-chip intake

New Apple announcements enter intake within one business day, but an announcement does not change the supported set. Intake is: create or update the exact board record through Q-00 and the registry; acquire physical hardware before any physical, reset, performance, or conformance claim; obtain the kernel/DTB/firmware inputs and the opaque boot envelope through the canonical manifest; request an AGX profile extension row (REQ-AGX-01 through REQ-AGX-10) for the exact board, which raises `max_gpu_generation` only when ratified; run identity, tuple, and profile admission with unknown or ambiguous identity failing closed; identify generation-specific shader, memory, synchronization, display, and reset work without inferring capabilities from the closest prior generation; build reproducibly on two builders; run every section 10 through 13 matrix; record per-board evidence and residuals; and publish only the lifecycle state supported by evidence through F-07. The intake record lists remaining work so that "not yet qualified" is observable and no branch, package repository, or marketing page silently expands support.

## 18. PROGRAM dependency and handoff ledger

Each row names the producer, the consumer in this lane, the exact artifact identity consumed, freshness, scope, rejection behavior, residual owner, and the gate the dependency must be satisfied before. F-02 is explicitly rejected and blocks G-01 admission.

| ID | Slice | Producer | Consumer here | Artifact identity and digest consumed | Freshness | Scope | Rejection behavior | Residual owner | Due before |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| DEP-01 | F-02 | `omarchy-apple-platform` | Builder and admission guard | Ratified schema set (ID-03), generated Python binding and output lock, AGX profile extension (REQ-AGX-01 to REQ-AGX-10, TP-12) | Byte equality of ID-03 with the local lock at every use | All eight payload types plus the extension | Any use before ratification is rejected; the rejected candidate `c315c7e79928d0041deb582bed79a61074361b21` is not authority | F-02 owner and coordinator | G-01, REJECTED and BLOCKING |
| DEP-02 | F-03 | `omarchy-apple-platform` | Both consumers | `Trusted<TrustContext>` with `key_set_digest`, `revocation_epoch`, role bindings for `manifest-release`, `board-admission`, `qualification-lab`, `ci-conformance`, `evidence-reader` | `expires_at` and revocation epoch checked at every verification | Every signature this lane verifies or requests | No `Trusted<T>`; no evidence can be signed | F-03 owner | G-01 |
| DEP-03 | F-04 | `omarchy-apple-platform` | Builder | Builder image digest, recipe (ID-21), toolchain lock (ID-22), immutable artifact store, two-builder comparison harness | Recipe digest equality per build | Sections 7.3 to 7.5 and 9 | `MESA_BUILD_INPUT_UNLOCKED` or `MESA_BUILD_NONREPRODUCIBLE`; no artifact emitted | F-04 owner | G-01 |
| DEP-04 | F-05 | `omarchy-apple-platform` | Builder output, candidate identity | Candidate assembler, tuple/ABI validator, rejection code vocabulary, candidate ID/digest (ID-23, ID-24) | Per candidate | Sections 8, 9.6, 15 | Mesa outputs remain unassembled evidence; no candidate exists | F-05 owner | G-02 |
| DEP-05 | F-06 | `omarchy-apple-platform` | Builder | Per-artifact license/notice/source-offer inventory schema and policy result | Per candidate | Section 9.4 license inventory | Candidate assembly blocked | F-06 owner | G-01 |
| DEP-06 | F-07 | `omarchy-apple-platform` | Section 15 | Promotion attestation shape, compliance bundle binding, public-ledger projection, sole stable-write terminal | Per promotion | Section 15 | Manifest with `channel = stable` lacking a verifiable F-07 attestation is treated as not stable | F-07 owner | Any stable promotion; BLOCKED, schema unresolved |
| DEP-07 | P-03 | `omarchy-mac` | Section 15 closure | ARM parity census and tracked porting queue, as an F-07 prerequisite | Per promotion | Product integration parity | F-07 refuses promotion; Mesa cannot compensate | P-03 owner | Any stable promotion |
| DEP-08 | K-02 | `linux-omarchy` | Sections 8, 10.3 | M1/M2 kernel/DT package set: ABI-03 to ABI-10, ABI-13, ABI-20 | Per candidate | G-02 boards | `MESA_BUILD_ABI_MISMATCH`; G-02 cannot enter | K-02 owner | G-02 |
| DEP-09 | K-03 | `linux-omarchy` | Sections 8, 10.3 | Same members for every exact M3 board | Per candidate | G-03 boards | Same | K-03 owner | G-03 |
| DEP-10 | K-04 | `linux-omarchy` | Sections 8, 10.3 | Same members for every exact M4 board | Per candidate | G-04 boards | Same | K-04 owner | G-04 |
| DEP-11 | K-05 | `linux-omarchy` | Sections 8, 10.3 | Same members for A18 Pro and M5 boards | Per candidate | G-05 boards | Same | K-05 owner | G-05 |
| DEP-12 | K-06 | `linux-omarchy` | Sections 8, 10.3 | Same members for M6 boards after shipping hardware exists | Per candidate | G-05 M6 subset | Same; M6 remains intake-only | K-06 owner | G-05 M6 subset and Q-08 |
| DEP-13 | Q-00 | `omarchy-apple-platform` | Sections 5.2, 13.2 | Cited, digest-addressed intake dataset and contradiction ledger from which board records and chip IDs (REQ-AGX-04) derive | Dataset digest per registry revision | Every board this lane may name | No board or chip ID is admitted from research rows | Q-00 owner | G-01 profile ratification |
| DEP-14 | Q-01 | `omarchy-apple-platform` | Section 13 | Board inventory, capability criteria, and qualification-record profile rows for AGX | Per registry revision | All Q-rows | No AGX qualification row can be recorded | Q-01 owner | G-02 |
| DEP-15 | Q-02 | `omarchy-apple-platform` | Sections 13.4 to 13.6 | Lab controller, evidence ingestion, redaction, public ledger generation | Per evidence entry | All physical evidence | Evidence cannot be ingested or read (TP-11) | Q-02 owner | G-02 |
| DEP-16 | Q-03 | hardware lab | Section 13.3 | Inventory and recovery certification of QT-01 units per profile | Per profile | Every profile | Profile has no units; no records possible | Lab owner | G-02 |
| DEP-17 | Q-04 | hardware lab | Section 13.1 | Signed M1/M2 qualification records (ID-19, ID-20) | `expires_at` per record | G-02 boards | G-02 cannot exit | Lab owner | F-07 for M1/M2 |
| DEP-18 | Q-05 | hardware lab | Section 13.1 | Signed M3 records | Same | G-03 boards | G-03 cannot exit | Lab owner | F-07 for M3 |
| DEP-19 | Q-06 | hardware lab | Section 13.1 | Signed M4 records | Same | G-04 boards | G-04 cannot exit | Lab owner | F-07 for M4 |
| DEP-20 | Q-07 | hardware lab | Section 13.1 | Signed A18 Pro and M5 records | Same | G-05 boards | G-05 cannot exit | Lab owner | F-07 for A18/M5 |
| DEP-21 | Q-08 | hardware lab | Section 13.1 | Signed M6 records | Same | G-05 M6 subset | G-05 M6 subset cannot exit | Lab owner | F-07 for M6 |
| DEP-22 | B-02, B-05 to B-08 | human boot-artifact owner via coordinator | TP-13, ABI-23 | Opaque boot artifact content digests inside the manifest | Per manifest | Every board cohort | Tuple incomplete; no candidate | Human owner and coordinator | Q-04 to Q-08 respectively |

Twenty-two dependency rows are bound.

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

Each fixture has one canonical input and exactly one mutation, one expected rejection, one path, and an executing owner. Expected codes are PROVISIONAL F-02 names where the rejection occurs at the F-02 seam, and section 9.6 builder classes or a named gate where it occurs in this lane. None of these fixtures exists yet; all are NOT IMPLEMENTED. The catalog is the minimum; a target validator must reject every row with zero unexpected accepts (QT-08).

| ID | Class | Single mutation | Expected rejection | Path or member | Executor |
| --- | --- | --- | --- | --- | --- |
| FX-01 | trust replay | Reuse an accepted manifest envelope's replay identity | `EXPIRY_OR_REPLAY_FAILURE` | `$.replay_id` | F-02 verifier via TP-02 |
| FX-02 | trust transplant | Sign a `platform-manifest/v1` payload under the `qualification-result` context | `SIGNATURE_CONTEXT_MISMATCH` | `$.context` | F-02 verifier |
| FX-03 | trust transplant | Sign a qualification record with the `manifest-release` role | `SIGNATURE_CONTEXT_MISMATCH` | `$.signatures[0].signer_role` | F-02 verifier |
| FX-04 | trust transplant | Present a valid manifest with an `ExpectedContext.board_id` for a different board | `CROSS_DOCUMENT_MISMATCH` | `$.payload.board_targets` | Admission guard |
| FX-05 | trust expiry | Manifest `expires_at` before TP-07 time | `MANIFEST_EXPIRY_FAILURE` | `$.payload.expires_at` | F-02 verifier |
| FX-06 | trust seam | Pass a parsed, unverified manifest to the admission guard | `TRUST_BOUNDARY_FAILURE` | `$` | Admission guard |
| FX-07 | digest substitution | Put a manifest `document_id` where `manifest_digest` is expected in a qualification binding | `PARSE_SCHEMA_FAILURE` | `$.payload.manifest.manifest_digest` | F-02 verifier |
| FX-08 | digest substitution | Put the queue tip commit in `upstream_commit` and the base in `source_commit` | `MESA_BUILD_QUEUE_INVALID` | `components.mesa_stack.source` | Builder |
| FX-09 | digest substitution | Use a tree ID as `source_commit` | `MESA_BUILD_SOURCE_UNVERIFIED` | provenance `peeled_commit` | Builder |
| FX-10 | digest substitution | Use a patch digest as an artifact `content_digest` | `MANIFEST_AUTHORITY_CONFLICT` | `$.payload.artifacts[0].content_digest` | F-02 verifier |
| FX-11 | digest substitution | Use the generated-output lock digest as `schema_set_digest` | `CROSS_DOCUMENT_MISMATCH` | `$.schema_set_digest` | TP-10 negotiation |
| FX-12 | digest substitution | Present the kernel ABI report digest of a different kernel commit as ID-18 | `MESA_BUILD_ABI_MISMATCH` | ABI-20 | Builder |
| FX-13 | digest substitution | Reference a qualification record by digest inside a manifest | `UNKNOWN_FIELD` | `$.payload.qualification_bindings[0].record_digest` | F-02 verifier |
| FX-14 | digest substitution | Use an `edge` manifest digest as the F-07 candidate digest for a different manifest ID | F-07 closure rejection | ID-23, ID-24 | F-07 |
| FX-15 | future generation | DRM params report `gpu_generation = 15` with a ratified generation 14 profile | REQ-AGX-08 `future-generation` | `drm_asahi_params_global.gpu_generation` | Admission guard and built driver |
| FX-16 | below minimum | DRM params report `gpu_generation = 12` | REQ-AGX-08 `below-minimum-generation` | `gpu_generation` | Admission guard and built driver |
| FX-17 | unknown chip | DRM params report `chip_id = 0x9999` | REQ-AGX-08 `unknown-chip` | `chip_id` | Admission guard and `agxdecode` |
| FX-18 | virtual device | DRM driver name `virtio_gpu` | REQ-AGX-08 `virtual-or-foreign` | driver name | Admission guard and built driver |
| FX-19 | profile mismatch | Registry `chip_id_u32` differs from DRM `chip_id` by one | REQ-AGX-08 `profile-mismatch` | `identity_match.macos.chip_id_u32` | Admission guard |
| FX-20 | profile mismatch | `num_clusters_total` differs from the profile | REQ-AGX-08 `profile-mismatch` | `num_clusters_total` | Admission guard |
| FX-21 | profile mismatch | Driver advertises a Vulkan feature the profile marks `unsupported` | REQ-AGX-08 `profile-mismatch` | ABI-19 | Conformance harness against the built driver |
| FX-22 | profile unbound | Board with `gpu` present and no bound profile | REQ-AGX-08 `profile-unbound` | REQ-AGX-01 | Admission guard |
| FX-23 | ambiguous identity | Two registry boards share `chip_id_u32` and the observation matches both | `AMBIGUOUS_IDENTITY` | `$.payload.boards[1].identity_match` | F-02 verifier |
| FX-24 | mutable source | Provenance `ref_name` is a branch with no `peeled_commit` | `MESA_BUILD_SOURCE_UNVERIFIED` | provenance `ref_name` | Builder |
| FX-25 | mutable source | `source.branch` added to the manifest component | `UNKNOWN_FIELD` | `$.payload.components.mesa_stack.source.branch` | F-02 verifier |
| FX-26 | mutable source | Ref advertisement differs between upstream and mirror | `MESA_BUILD_SOURCE_UNVERIFIED` | `ref_advertisement_digest` | Synchronization job |
| FX-27 | reordered patch | Swap `order` of two queue entries without a new queue report | `MESA_BUILD_QUEUE_INVALID` | `patch_lock.entries[1].order` | Builder |
| FX-28 | edited patch | Change one byte of a patch while keeping `patch_digest` | `MESA_BUILD_QUEUE_INVALID` | `patch_lock.entries[0].patch_digest` | Builder |
| FX-29 | dropped patch | Remove the last queue entry while keeping `source_commit` | `MESA_BUILD_QUEUE_INVALID` | `source.source_commit` | Builder |
| FX-30 | missing subproject lock | Remove one of the 42 wrap digests from the lock | `MESA_BUILD_INPUT_UNLOCKED` | section 7.3 | Builder |
| FX-31 | wrap download | Wrap with a `source_url` and no cache entry, `--wrap-mode=nodownload` | `MESA_BUILD_INPUT_UNLOCKED` | section 7.3 | Builder |
| FX-32 | missing generated lock | Mako template changed without a `config_inputs` digest update | `MESA_BUILD_INPUT_UNLOCKED` | ID-13 | Builder |
| FX-33 | generator drift | Generated file differs between builders | `MESA_BUILD_NONREPRODUCIBLE` | ID-14 | Two-build comparison |
| FX-34 | ABI one-field | ABI-03 kernel release string differs | `MESA_BUILD_ABI_MISMATCH` | ABI-03 | Builder and admission guard |
| FX-35 | ABI one-field | ABI-04 kernel commit differs | `MESA_BUILD_ABI_MISMATCH` | ABI-04 | Builder and admission guard |
| FX-36 | ABI one-field | ABI-05 one config input digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-05 | Builder and admission guard |
| FX-37 | ABI one-field | ABI-06 UAPI header digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-06 | Builder and admission guard |
| FX-38 | ABI one-field | ABI-07 one `drm_asahi_params_global` field renamed in the report | `MESA_BUILD_ABI_MISMATCH` | ABI-07 | Builder |
| FX-39 | ABI one-field | ABI-08 `abi_contract_id` differs | `MESA_BUILD_ABI_MISMATCH` | ABI-08 | Builder and admission guard |
| FX-40 | ABI one-field | ABI-09 one DTB artifact digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-09 | Admission guard |
| FX-41 | ABI one-field | ABI-10 `dt_schema.schema_digest` differs | `MESA_BUILD_ABI_MISMATCH` | ABI-10 | Builder |
| FX-42 | ABI one-field | ABI-11 firmware artifact version differs | `MESA_BUILD_ABI_MISMATCH` | ABI-11 | Admission guard |
| FX-43 | ABI one-field | ABI-12 firmware schema version differs | `MESA_BUILD_ABI_MISMATCH` | ABI-12 | Builder and admission guard |
| FX-44 | ABI one-field | ABI-13 firmware/kernel relation `evidence_digest` differs | `MESA_BUILD_ABI_MISMATCH` | ABI-13 | Builder |
| FX-45 | ABI one-field | ABI-14 `upstream_commit` differs from the verified mirror | `MESA_BUILD_ABI_MISMATCH` | ABI-14 | Builder |
| FX-46 | ABI one-field | ABI-15 patch lock digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-15 | Builder |
| FX-47 | ABI one-field | ABI-16 recipe digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-16 | Builder |
| FX-48 | ABI one-field | ABI-17 one artifact digest differs from the built file | `MESA_BUILD_ARTIFACT_INCOMPLETE` | ABI-17 | Builder |
| FX-49 | ABI one-field | ABI-18 one package version differs | `MESA_BUILD_ABI_MISMATCH` | ABI-18 | Admission guard |
| FX-50 | ABI one-field | ABI-19 feature-set digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-19 | Builder and admission guard |
| FX-51 | ABI one-field | ABI-20 relation owned by `mesa-stack` instead of `linux-kernel` | `MANIFEST_AUTHORITY_CONFLICT` | ABI-20 | F-02 verifier |
| FX-52 | ABI one-field | ABI-21 package architecture `arm64` where `aarch64` is bound | `MESA_BUILD_ABI_MISMATCH` | ABI-21 | Builder |
| FX-53 | ABI one-field | ABI-22 `previous_manifest_ids` names a manifest that never verified | `MESA_BUILD_ABI_MISMATCH` | ABI-22 | Builder and admission guard |
| FX-54 | ABI one-field | ABI-23 one opaque boot artifact digest differs | `MESA_BUILD_ABI_MISMATCH` | ABI-23 | Admission guard |
| FX-55 | ABI one-field | ABI-24 schema-set digest differs in one document | `CROSS_DOCUMENT_MISMATCH` | ABI-24 | F-02 verifier |
| FX-56 | build drift | One installed file differs between builders | `MESA_BUILD_NONREPRODUCIBLE` | section 9.3 | Two-build comparison |
| FX-57 | build drift | Option left at `auto` in the recipe | recipe error, no build | section 9.1 | Recipe validator |
| FX-58 | network access | Resolver call after cache admission | `MESA_BUILD_NETWORK_DENIED` | section 9.2 | Builder sandbox |
| FX-59 | network access | Meson attempts a wrap download | `MESA_BUILD_NETWORK_DENIED` | section 7.3 | Builder sandbox |
| FX-60 | environment | Operator `CFLAGS` present in the build environment | `MESA_BUILD_NONREPRODUCIBLE` | section 9.2 | Builder sandbox |
| FX-61 | incomplete artifacts | SBOM missing | `MESA_BUILD_ARTIFACT_INCOMPLETE` | section 9.4 | Builder |
| FX-62 | incomplete artifacts | One detached signature missing | `MESA_BUILD_ARTIFACT_INCOMPLETE` | section 9.5 | Builder |
| FX-63 | incomplete artifacts | ABI report missing while packages exist | `MESA_BUILD_ARTIFACT_INCOMPLETE` | section 8.3 | Builder |
| FX-64 | incomplete artifacts | Two-build comparison report missing | `MESA_BUILD_ARTIFACT_INCOMPLETE` | section 9.3 | Builder |
| FX-65 | package mismatch | Package `content_digest` differs from the repository index | Install transaction rejection | section 9.5 | P-05 install transaction |
| FX-66 | channel mismatch | Package reachable from a `stable` namespace without an F-07 attestation | Repository-integrity failure | section 9.5 | F-07 and repository audit |
| FX-67 | channel mismatch | Manifest `channel = stable` presented without a verifiable F-07 attestation | Treated as not stable, not selected | section 15 | Admission guard |
| FX-68 | package mismatch | Package signed by a key without the `signature_policy_id` role | `TRUST_FAILURE` | `$.signatures[0].key_id` | F-03 verifier |
| FX-69 | qualification undercount | One unit per profile where QT-01 requires two | Record `outcome = fail` | QT-01 | Q-02 lab controller |
| FX-70 | qualification undercount | Nine update/rollback cycles where QT-06 requires ten | Record `outcome = fail` | QT-06 | Q-02 lab controller |
| FX-71 | retry laundering | Pass whose attempt list omits an earlier failed attempt | Record `outcome = fail` | section 13.4 | Q-02 lab controller |
| FX-72 | retry laundering | Failed attempt evidence entry deleted | `CROSS_DOCUMENT_MISMATCH` | `$.payload.test_results[0].evidence_ids[0]` | F-02 verifier |
| FX-73 | family extrapolation | M3 Pro record cited for an M3 Max board target | `CROSS_DOCUMENT_MISMATCH` | `$.payload.board.board_id` | F-02 verifier |
| FX-74 | family extrapolation | Manifest `board_targets` contains a board with no qualification binding | `CROSS_DOCUMENT_MISMATCH` | `$.payload.qualification_bindings` | F-02 verifier |
| FX-75 | software fallback | Software rasterizer result recorded as a hardware `gpu` pass | Record `outcome = fail` | section 13.5 | Q-02 lab controller |
| FX-76 | alternate promotion | Mesa CI writes a package into the `stable` namespace | Repository-integrity failure and incident | section 15 | Repository audit and F-07 |
| FX-77 | self-promotion | Manifest with `channel = stable` signed by `ci-conformance` | `SIGNATURE_CONTEXT_MISMATCH` | `$.signatures[0].signer_role` | F-02 verifier |
| FX-78 | alternate promotion | F-07 invoked with an artifact set differing from the candidate manifest by one digest | F-07 closure rejection | ID-24 | F-07 |
| FX-79 | self-promotion | Manifest lineage `generation` skips a value | `DOCUMENT_ID_FORK` | document lineage record | F-02 verifier |

Seventy-nine hostile fixtures are required. Executing them requires the ratified schemas, generated bindings, builder, and lab controller, none of which exist; their current census is 0 PASS / 79 NOT EXECUTABLE.

## 21. Residuals and NOT IMPLEMENTED empirical risks

Every row is an open residual with an owner and a gate it must be closed before. None is a design accomplishment.

| ID | Residual | Evidence | Owner | Due before |
| --- | --- | --- | --- | --- |
| RES-01 | `gpu_generation >= 14` extrapolation admits any future generation | `src/asahi/lib/agx_device.c:662-670`; no upper bound; `agx_device.c:544` lower bound is assertion-only | Mesa lane (downstream patch after REQ-AGX-09) | G-02 |
| RES-02 | Unknown chip IDs decode as generation 13 G | `src/asahi/lib/decode.c:951-959` shared `default:` arm; `decode.c:974` default call | Mesa lane | G-02 |
| RES-03 | Vulkan features and extensions advertised largely unconditionally | `src/asahi/vulkan/hk_physical_device.c:234-270` and `48-232` | Mesa lane | G-02 |
| RES-04 | `virtio_gpu` accepted as an Asahi native context | `src/asahi/lib/agx_device.c:519-527`, `46-47`, `313-317`; the cited range `1222-1229` does not exist in the 960-line file | Mesa lane | G-02 |
| RES-05 | 64 coordinator hostile cases had 0 executable rejections; this revision defines 79 fixtures, all NOT EXECUTABLE | Coordinator review and section 20 | F-02, F-04, F-05, Q-02 owners with Mesa lane | G-01 |
| RES-06 | No target validator, schema, fixture, generated binding, consumer guard, target CI, build pipeline, or physical evidence exists | PROGRAM section 12 and this document | F-02 through F-07 owners; Mesa lane for the guard | G-01 |
| RES-07 | Meson, Ninja, Vulkan CTS, `glslangValidator`, `vkcube`, and `deqp-runner` were unavailable; no build directory exists | Coordinator review; no build performed by this lane | F-04 owner | G-01 |
| RES-08 | Device UUID derived from generation, variant, and revision only | `src/asahi/lib/agx_device.c:895-901` | Mesa lane | G-02 |
| RES-09 | Python interpreter and Mako detection at configure time | `meson.build:1176-1226` | F-04 recipe owner with Mesa lane | G-01 |
| RES-10 | AGX profile extension not ratified; TP-12 has no producer | Section 5.2 | F-02 owner and coordinator | G-01 |
| RES-11 | F-06/F-07 handoff schema unresolved | DEP-06 | F-06 and F-07 owners | Any stable promotion |
| RES-12 | Builder rejection classes not reconciled with F-04/F-05 vocabulary | Section 9.6 | F-04 and F-05 owners | G-01 |
| RES-13 | Upstream ref signing evidence not observed: the blob-filtered worktree carries no tags, so whether upstream signs release refs is unverified here | Section 7.1 `signer_evidence` | Mesa lane synchronization job | G-01 |
| RES-14 | No physical board, unit, or qualification record exists for any AGX profile | Section 13 | Q-03 through Q-08 owners | G-02 |

Fourteen residuals are open.

## 22. Reporting format

Each implementation or review report must include: repository, branch, exact base and tip commit IDs, and verified authoritative commit; changed-file census and confirmation that no graphics code or canonical program file was modified by the design lane; gate-by-gate PASS, FAIL, BLOCKED, or NOT RUN output with commands and evidence digests; failure census with counts and causes; mirror drift status, queue disposition, section 8 tuple by member ID, builder identities, artifact digests, and rollback predecessor; physical boards and units actually tested, never inferred coverage; deviations, unsupported features, open upstream work, recovery limits, and residual IDs from section 21; and a statement that nothing is DONE unless the coordinator has independently integrated and promoted the required slice.

At design time the residuals are deliberate: no gate is implemented by this documentation change; no physical board was qualified by this lane; no conformance result is claimed; no generation was promoted; F-02 remains rejected and every dependency on it is provisional; the canonical platform schemas remain the source of truth; and the opaque boot artifact boundary remains fenced under the coordinator ruling.
