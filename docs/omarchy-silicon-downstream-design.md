# Omarchy Silicon Mesa/AGX downstream design

Status: DESIGN NOTE — G-01 through G-05 are TODO; nothing in this document is DONE.

Owner: Mesa/AGX downstream design lane

Repository: `omarchy-silicon/mesa-omarchy`

Branch: `factory/design-mesa-release`

This document is a design contract for the Mesa component of the Omarchy Silicon platform program. It does not modify Mesa graphics code, establish hardware support, claim conformance, or promote any board or release. A build, recognized GPU, booting compositor, or passing subset of tests is evidence for a gate only; it is never a support claim by itself.

## 1. Authority, ownership, and boundaries

The authoritative Mesa history is freedesktop.org Mesa at `https://gitlab.freedesktop.org/mesa/mesa`. `https://github.com/omarchy-silicon/mesa-omarchy` is the Omarchy-owned mirror and downstream integration point. The GitHub repository may be a standalone owned mirror rather than a GitHub-native fork; GitHub ownership, repository URL, default branch, and upstream history must all be checked explicitly.

`origin/main` is the pure mirror branch. It must represent the exact authoritative Mesa `main` tip after each accepted synchronization. Omarchy changes are carried on named downstream branches or release refs above a recorded upstream base. A downstream commit must never be silently placed on the mirror branch.

The canonical `omarchy-apple-platform` repository owns `board-registry/v1`, `platform-manifest/v1`, and `qualification-record/v1`. This repository consumes those interfaces and records references to them; it must not publish a second board map, capability vocabulary, qualification schema, or release authority. The platform manifest is the only authority that assembles Mesa with the kernel, firmware, boot artifacts, and userspace.

The kernel and firmware are external component boundaries. Mesa declares the exact kernel source/configuration/device-tree and firmware schema/digest required by a candidate; it does not infer compatibility from a product name, a SoC family, `uname`, or a successful probe.

The coordinator has declared `m1n1-omarchy` an opaque human-produced artifact boundary because that repository contains an AGENTS.md prohibiting AI/LLM use. This lane does not inspect, clone, analyze, edit, test, or make claims about that repository or its contents. Any required m1n1 identity or compatibility fact must arrive through the canonical signed platform manifest and qualification record.

## 2. Design principles

- Preserve upstream Mesa history and keep the Omarchy patch queue minimal, reviewable, and removable.
- Fail closed on mirror drift, source ambiguity, ABI mismatch, unsupported generation, missing evidence, signature failure, or non-reproducible output.
- Tie every result to exact source, patch queue, build recipe, kernel/firmware tuple, board identity, and evidence IDs.
- Keep generation support board-specific. A shared SoC label is diagnostic metadata, not qualification.
- Separate source synchronization, implementation, build, conformance, compositor, physical, packaging, and release gates.
- Promote an immutable artifact digest from edge to RC to stable; never rebuild a moving branch during promotion.
- Keep the last-known-good complete platform tuple available for rollback for the supported lifetime of each board.
- Report failures, exclusions, and residuals alongside passing results. No warning may be converted into a success state.

## 3. G-01 authoritative mirror synchronization

### 3.1 Mirror invariants

The mirror is healthy only if all of the following hold:

- The GitHub repository is owned by `omarchy-silicon`, its canonical URL is the one recorded in the platform manifest, and its default branch is `main`.
- The authoritative source URL is exactly `https://gitlab.freedesktop.org/mesa/mesa` or a coordinator-approved immutable equivalent.
- `origin/main` and authoritative `main` resolve to the same commit object. Equality is required; merely having the authoritative commit as an ancestor is insufficient for the pure mirror branch.
- The mirror retains the authoritative commit history needed to reproduce the selected source. A shallow or object-missing checkout is rejected for release assembly.
- The mirror branch has no Omarchy-only commits, generated release edits, vendored graphics code, or merge-only synchronization commits.
- The source commit, tree, parent set, and repository URL recorded in the candidate manifest agree with the verified remote values.
- Any downstream patch branch declares the exact mirror commit from which it was based and the ordered downstream commit IDs or patch digest.

The current design snapshot was prepared after comparing GitHub `origin/main` and freedesktop.org `main`: both resolved to `d870cef8b7c8a4a11edc669669c9f18ae402314a` on 2026-09-02. This is an observed synchronization input, not a claim that future mirror state, conformance, or hardware support is complete.

### 3.2 Synchronization procedure

The scheduled synchronization job and every release candidate perform the same read-only checks before accepting source:

1. Fetch authoritative refs and the GitHub mirror with pruning, without accepting unpinned branch URLs or arbitrary remote responses.
2. Resolve both `refs/heads/main` values and record their full 40-character object IDs, commit metadata, tree ID, and parent IDs.
3. Verify repository ownership and default-branch metadata through the GitHub API or equivalent authenticated owner check.
4. Verify that the authoritative commit is present locally and that the mirror has no missing objects needed for the candidate.
5. Require exact equality of `upstream_main_sha` and `mirror_main_sha`.
6. Verify that the candidate downstream base is that exact SHA and that the patch queue applies cleanly in order.
7. Write the source identity, check time, tool version, and raw command result into the release evidence referenced by `platform-manifest/v1`.

The verification must be independently reproducible from a blob-filtered clone. Blob filtering is an approved storage optimization; it is not permission to omit commit, tree, tag, or release evidence required by the candidate. A build may fetch missing blobs only from the allowlisted source and must record the resulting object set.

### 3.3 Mirror drift is a hard gate

Mirror drift blocks all downstream release activity. The following are hard failures:

| Condition | Required result |
|---|---|
| `origin/main` SHA differs from authoritative `main` SHA | Stop synchronization, block patch rebase and candidate assembly, quarantine all channels, and open a coordinator ruling. |
| `origin/main` contains extra Omarchy commits | Reject the mirror branch; move work to a downstream ref only through an auditable correction. Never auto-reset or hide the commits. |
| The authoritative remote, owner, default branch, or repository identity changes unexpectedly | Stop before accepting new source and require explicit owner/coordinator review. |
| The authoritative main branch is force-rewritten or the recorded base is no longer an ancestor | Stop promotion and invalidate affected source attestations until history is reconciled. |
| Required commit/tree/tag/blob objects cannot be fetched from the allowlisted source | Reject the candidate as non-reproducible; do not substitute a local cache or another fork. |
| Patch queue base does not equal the verified mirror SHA | Reject the queue; rebase or regenerate it and rerun all dependent tests. |
| Source identity is absent, mutable, unsigned where signature is required, or inconsistent across manifest/build/package metadata | Reject before build or package publication. |
| Drift is discovered after build or packaging | Mark artifacts revoked/quarantined, prevent promotion, preserve evidence, and assemble a new candidate from a clean verified base. |

The hard gate applies to edge, RC, and stable. Edge is an experimental release channel, not an exception to source integrity. A drift warning is not an acceptable release status, and a successful build cannot override this gate.

The coordinator may approve a mirror repair, but the repair is a new auditable event. It must record the old and new SHAs, reason, impact on patch queue and artifacts, rerun source and reproducibility checks, and retain the failed evidence. The design never auto-merges upstream into a diverged mirror branch.

### 3.4 Source-check pseudocode

The implementation belongs in the platform release tooling, not in this documentation file. Its contract is equivalent to:

```text
authoritative = resolve(allowed_freedesktop_url, "refs/heads/main")
mirror = resolve(owned_github_url, "refs/heads/main")
require github_owner == "omarchy-silicon"
require github_default_branch == "main"
require authoritative.sha == mirror.sha
require candidate.upstream_main_sha == authoritative.sha
require candidate.patch_base_sha == mirror.sha
require source_objects_complete(candidate)
```

Any failed `require` returns a hard failure with the offending values. It must not fall back to a nearby commit, a cached result, a compatible-looking fork, or a chip-family default.

## 4. Minimal downstream patch queue

### 4.1 Queue shape

The downstream queue is a short, ordered series of atomic Git commits above an exact `origin/main` SHA. The queue is the implementation history; a second manually maintained patch list is not an authority. The platform manifest records the base SHA and an ordered queue digest, while the repository preserves the reviewable commits and their upstream disposition.

Each downstream commit must include:

- A scoped subject identifying the Mesa subsystem and AGX or platform reason.
- The exact affected generation or board records, if any.
- Why upstream Mesa cannot accept the change as-is, or the upstream issue/MR/reference when it can.
- Kernel/firmware ABI assumptions and the manifest fields that must match.
- Focused tests and the expected evidence type.
- Upstream disposition: proposed, accepted, waiting, intentionally downstream, or scheduled for removal.
- An owner and review date or removal condition for temporary work.

The queue must not contain unrelated style rewrites, generated artifacts without their generator input, copied upstream commits with altered identity, broad “future chip” fallbacks, or a workaround whose only proof is a desktop that happens to start.

### 4.2 Patch classification

| Class | Policy | Exit condition |
|---|---|---|
| Upstream candidate | Keep the smallest upstreamable change, link its freedesktop review, and test against the authoritative base. | Remove the downstream copy after the upstream commit is in the verified mirror. |
| Compatibility bridge | Carry only when a pinned Omarchy kernel/firmware ABI needs a temporary bridge that upstream cannot yet consume. | Remove when the ABI contract or upstream implementation changes, with a recorded rebase decision. |
| Board or generation enablement | Add explicit capability data and compiler/runtime work only for identified hardware and required evidence. | Replace by upstream support or keep with an explicit board lifecycle and maintenance owner. |
| Release integration | Packaging, manifest, or build metadata that belongs outside Mesa code. | Keep out of `origin/main`; encode through the platform release pipeline. |
| Emergency corrective fix | Use only for a release-blocking regression with a reproducible failure and rollback plan. | Upstream immediately where possible; otherwise expire at the next synchronization checkpoint. |

Queue review rejects a patch when a capability can be expressed by an existing upstream interface, when it widens support without board evidence, when it duplicates canonical platform data, or when its removal condition is missing.

### 4.3 Queue admission and removal

Before a queue branch is accepted, the author produces a clean application from the verified mirror base, a patch digest, the changed-path census, the ABI declaration, and focused test results. A reviewer attempts to apply the queue to the previous and next eligible upstream bases where available. A queue that cannot be rebased cleanly is blocked rather than resolved by ad hoc conflict edits.

When upstream absorbs a change, removal is a separate small commit or rebase event with the upstream commit ID, changed behavior, and rerun test evidence. Dead workarounds are not retained “for safety.” When upstream changes behavior without absorbing the queue, the compatibility owner must either update the bridge with a new evidence record or remove the feature from the candidate.

## 5. Upstreaming and rebase policy

The synchronization cadence is continuous for source monitoring and at every candidate build. A release does not wait for upstream acceptance, but it never treats a downstream fork as the authority for the next mirror tip.

The normal sequence is:

1. Verify the new freedesktop.org SHA and update the pure mirror branch.
2. Rebase the downstream queue in commit order onto that exact SHA in an isolated worktree.
3. Resolve only conflicts attributable to the queue, retaining upstream semantics unless the compatibility declaration proves a temporary bridge is still required.
4. Rerun source, ABI, compiler, conformance, compositor, performance, reset, reproducibility, packaging, and rollback gates that the rebase can affect.
5. Publish a rebase report naming removed, changed, retained, and newly blocked patches.
6. Submit upstream candidates with the smallest self-contained series, current tests, and the exact upstream base.
7. Do not force-push shared release refs. A failed rebase creates a new reviewable correction ref and leaves the prior candidate available for rollback.

The queue owner may not mark a patch upstreamed based on an open review alone. “Accepted” means the commit is present in the verified authoritative history and the downstream copy has been removed or proven necessary by a new ABI decision. A rebase may never relax a hard gate to preserve a release date.

## 6. Kernel/firmware ABI declaration

Mesa release metadata must name the complete graphics contract consumed by the candidate. It must reference canonical schema objects instead of copying their definitions. The minimum declaration is:

| Contract item | Required declaration |
|---|---|
| Board identity | Exact board ID, SoC ID, device-tree compatibility record, lifecycle state, and qualification profile from `board-registry/v1`. |
| Kernel | Exact linux source SHA, configuration digest, device-tree artifact digest, DRM UAPI/feature contract, and required driver interface revision. |
| Firmware | Exact firmware schema, artifact digest, provenance, and any required firmware/kernel pairing. |
| Mesa | Authoritative upstream base SHA, ordered downstream queue digest, generated-source inputs, and Mesa package version. |
| Compiler/toolchain | LLVM/compiler, SPIR-V tools, Python/Mako, Meson/Ninja, linker, and builder image/toolchain digests. |
| Userspace integration | Exact libdrm, Wayland protocol, Hyprland, Quickshell, loader, and relevant runtime package identities. |
| Release relationship | Platform-manifest ID, rollback-compatible predecessor IDs, signing identity, channel, and expiry/revocation metadata. |

The declaration must distinguish stable kernel DRM UAPI from implementation-private interfaces. Mesa must not claim ABI compatibility merely because a symbol or device node exists. A missing, unknown, or incompatible field blocks installation and candidate activation before the graphics stack is selected.

For each ABI item, compatibility is one of:

- Exact match required.
- Explicitly compatible range, signed by the canonical schema owner and covered by tests.
- Incompatible; the candidate is rejected.

There is no implicit “latest,” “same chip,” or “backward compatible” value. A firmware schema change, kernel reset behavior change, or compiler/runtime mismatch invalidates the affected candidate until the complete tuple is rebuilt and requalified.

This lane does not inspect or validate the opaque m1n1 artifact boundary described in Section 1. The platform manifest must still carry whatever exact identity is required to assemble the boot/kernel/firmware/Mesa tuple, and the qualification record must cite evidence for the assembled tuple before a release can be promoted.

## 7. Reproducible Meson builds

### 7.1 Build recipe

The canonical build recipe is owned by the platform release pipeline. Mesa consumes the recipe as a pinned input; local developer defaults must not become release behavior. The recipe records:

- Mesa upstream SHA and downstream queue digest.
- Host architecture, target architecture, CPU feature policy, and board/generation capability selection.
- Meson, Ninja, compiler, linker, LLVM, SPIR-V, Python, Mako, Rust where enabled, and system dependency versions.
- Builder image or isolated environment digest, package repository snapshot, and network-disabled build mode after source acquisition.
- Meson options, enabled graphics drivers, enabled APIs, debug assertions, optimization mode, sanitizers, and generated-source commands.
- Subproject/wrap source identities and hashes, including whether a system dependency is permitted.
- `SOURCE_DATE_EPOCH`, locale, timezone, file ordering, archive metadata, and any reproducibility-sensitive environment variables.
- Staging layout, package split, install paths, build logs, compiler invocations, and artifact manifest.

The recipe must use a clean source checkout and a fresh build directory. It must not read untracked local patches, a mutable home-directory cache, a developer's GPU, or an unpinned remote during compilation. Build caches are either disabled or content-addressed and included in the provenance record.

### 7.2 Reproducibility checks

Every release candidate is built independently by two isolated builders from the same source, queue, recipe, and dependency digests. The checks require:

- Identical installed file lists, modes, ownership policy, and generated metadata.
- Byte-identical package payloads after the declared deterministic packaging step.
- Identical debug/source-map policy and reproducible build IDs where those artifacts are shipped.
- Matching SBOM, source-provenance, compiler-invocation, and license inventories.
- A documented explanation for any intentionally non-identical local cache, timestamp, signature envelope, or builder attestation wrapper.

A mismatch is a release-blocking failure. It is investigated as a source, toolchain, generated-code, environment, or packaging defect. The candidate is not “close enough,” and one builder's output is not selected merely because it boots.

The recipe must provide a deterministic Meson configure and build transcript. The transcript records the resolved options and dependency versions, not only the command line typed by the operator. Any configure-time auto-detection that can alter the artifact must be converted to an explicit manifest input or cause the build to fail.

## 8. Shader, compiler, and GPU-generation work

### 8.1 Compiler contract

AGX shader work follows a staged contract:

1. Ingest the API shader representation and capture the exact source or binary input.
2. Run the pinned Mesa/NIR lowering and optimization pipeline.
3. Apply only generation- and capability-specific lowering supported by the board record.
4. Generate AGX intermediate representation and machine code with the pinned compiler/toolchain.
5. Validate register allocation, instruction encoding, resource limits, synchronization, barriers, interpolation, texture/image operations, and API-visible results.
6. Capture stable intermediate and final representations where deterministic output is promised, together with the compiler options and generation identity.

Unknown GPU generations, instruction capabilities, firmware interfaces, resource limits, or shader feature combinations fail closed. They do not fall back to the nearest generation's encoding or silently disable a required API feature. Optional features remain explicitly optional in the capability report and do not become a false success.

### 8.2 Golden and compiler tests

Golden tests are versioned inputs and expected outputs for NIR lowering, AGX IR, instruction selection, encoding, disassembly, and API-visible shader behavior where each representation is stable enough to promise. Every golden update records the reason, upstream or queue commit, generation, toolchain, and an independent semantic test. A changed golden file without a corresponding reason and review is a hard failure.

The compiler suite includes:

- Mesa unit tests and NIR/compiler tests selected by the changed path.
- Shader-db or equivalent corpus comparison for instruction count, register pressure, spills, code size, and compile time.
- API shader tests for required OpenGL/Vulkan/compute paths declared by the board profile.
- Negative tests for unsupported generation, unsupported feature, malformed shader, invalid resource use, and ABI mismatch.
- Cache invalidation tests across Mesa queue, kernel/firmware ABI, compiler, and board-generation changes.
- Determinism tests that compile the same corpus twice in clean build directories and compare promised outputs.

Performance improvements may not trade away correctness, reset safety, or a required capability. A new optimization needs a measured baseline, workload definition, variance policy, and rollback switch or artifact boundary when practical.

### 8.3 Generation intake lanes

G-02 through G-05 are staged work packages, not support claims:

| Goal | Scope | Entry condition | Exit evidence before platform promotion |
|---|---|---|---|
| G-01 | Mirror, queue, build, ABI, and test contracts | Verified ownership and exact upstream synchronization | Approved design, implemented gates, reproducible candidate pipeline, and no drift. |
| G-02 | M1/M2 graphics reference tuple | Exact M1/M2 board and kernel/firmware records are available through canonical interfaces | Physical reference-board evidence, compiler/API tests, compositor matrix, reset results, package/reproducibility evidence, and coordinator ruling. |
| G-03 | M3 base/Pro/Max/Ultra graphics and display dependencies | Each exact M3 board enters lifecycle intake with a pinned tuple | Per-board compiler and API results, display/compositor/reset evidence, physical qualification records, and release rollback evidence. |
| G-04 | M4 base/Pro/Max graphics and display dependencies | Each exact M4 board enters lifecycle intake with a pinned tuple | Same gates as G-03 without inheriting unsupported M3 assumptions. |
| G-05 | A18 Pro, M5 family, and future M6 graphics dependencies | Exact board records and shipping hardware are available; announced hardware remains intake-only | Per-board physical evidence, generation-specific compiler work, complete tuple, reproducible package, rollback, and coordinator promotion. |

The canonical program treats M6 as a target until shipping hardware is acquired and qualified. This Mesa design therefore permits an M6 intake record and preparatory compiler work but does not permit an M6 support label, release claim, or conformance statement from a marketing announcement or emulation alone.

## 9. Conformance, golden, performance, and reset evidence

### 9.1 Evidence record

Every result is attached to a `qualification-record/v1` or lower-level test record with:

- Exact board ID and SoC ID, RAM/storage class where relevant, and physical lab asset ID.
- macOS and firmware baseline, kernel/DT identity, Mesa upstream SHA, downstream queue digest, build recipe digest, and platform-manifest SHA.
- Test suite/version, test selection, command or harness revision, environment, expected result, actual result, timestamps, and operator/automation identity.
- Immutable raw log, capture, trace, image, measurement, or report references with privacy redaction.
- Failure classification, reproduction status, linked incident, quarantine state, and residual risk.

Static checks, compilation, mocked device trees, VMs, and a successful desktop are lower-level evidence. None can create a physical qualification record or change a board lifecycle state to FULL.

### 9.2 Required test families

| Family | Minimum design coverage | Release rule |
|---|---|---|
| Upstream/unit | Mesa Meson tests, NIR/compiler tests, API loader/runtime tests, and changed-path tests | All required tests pass or have an explicit coordinator-approved residual that prevents promotion to FULL. |
| Conformance | Applicable OpenGL, Vulkan, EGL/GLX, shader, and compute conformance suites for the board profile | Required failures block candidate activation and stable promotion. A skipped test needs a reason and lifecycle owner. |
| Golden | Compiler IR, machine-code/disassembly, and semantic shader corpus with reviewed updates | Unexpected output drift blocks the candidate until explained and reviewed. |
| Performance | Fixed workloads, frame-time distributions, shader compile latency, memory, power, thermal behavior, and regression comparison with the last qualified tuple | A defined regression budget is exceeded only through a recorded decision; unsafe thermal or power behavior is always a block. |
| Reset/recovery | GPU hang/reset injection, engine recovery, fence and context behavior, compositor survival, repeated reset cycles, and boot-health reporting | Any unrecovered hang, silent corruption, false success, or unsafe repeated-reset behavior blocks release. |
| Packaging | Install/upgrade/downgrade, dependency closure, signatures, SBOM/provenance, artifact digest, and file ownership | Missing or mutable metadata blocks publication. |

Tests must distinguish an expected unsupported feature from a regression. Neither may be represented as a generic pass. A required failure causes the board or candidate to remain in its narrower lifecycle state.

### 9.3 Reset test safety

Reset and hang tests run only on disposable or explicitly quarantined lab targets with a rehearsed recovery path. The harness records the injected fault, kernel/firmware/Mesa tuple, reset attempts, GPU and compositor logs, visual result, data-integrity result, and whether the system returned to a known-good state. It must not run destructive experiments against a user's installation or silently consume a failed board as a pass.

## 10. Hyprland/Quickshell regression matrix

Graphics qualification includes the production compositor and shell integration used by Omarchy. The matrix is keyed by exact build identities, not just package names:

| Axis | Required values or evidence |
|---|---|
| Hardware | Every candidate board record and applicable RAM/display class; reference coverage does not substitute for every board. |
| Platform tuple | Platform-manifest SHA, kernel/DT, firmware schema/digest, Mesa source/queue/build/package digests. |
| Hyprland | Exact commit/package, renderer/backend options, config revision, protocol versions, and logs. |
| Quickshell | Exact commit/package, shell configuration revision, QML/runtime versions, and logs. |
| Displays | Internal panel, each supported external DP/HDMI/Thunderbolt path, hotplug, mixed DPI, multi-monitor, rotation/scaling, and board-specific panel features. |
| Interaction | Login, workspace/animation, window resize, fullscreen, screenshots/recording, clipboard/drag, lock/unlock, idle, suspend/resume, monitor reconfiguration, and shell reload. |
| Stress | Shader compilation during interaction, memory pressure, long compositor soak, repeated display hotplug, video/3D workload overlap, and injected GPU reset where safe. |
| Observables | Frame-time distribution, dropped frames, render errors, GPU hangs/resets, visual diffs, shell crashes/restarts, protocol errors, power/thermal telemetry, and user-visible recovery. |
| Evidence | Immutable capture/log/trace IDs, exact expected result, pass/fail, environment, and residuals. |

The minimum matrix is expanded from the board registry's physical capabilities. A pass on an internal display cannot stand in for an external topology, and a compositor launch cannot stand in for a soak, reset, or shell-reload test. A regression in Hyprland or Quickshell is triaged against the same tuple and must not be hidden by weakening Mesa or disabling a required feature without an explicit capability outcome.

## 11. Physical board evidence and lifecycle

The lab maintains one inventory row per physical board/product topology with exact board identity, SoC, memory/storage class, firmware/macOS baseline, display and dock topology, peripherals, location, recovery state, and immutable asset ID. Raw evidence is retained with redaction and a link from the qualification record.

Mesa evidence is collected for the applicable G-02 through G-05 board lanes only after the kernel/firmware tuple has been supplied through canonical interfaces. It covers cold boot, warm reboot, shutdown, display hotplug, multi-display, suspend/resume where supported, compositor soak, GPU memory pressure, shader workload, GPU reset/recovery, power/thermal observation, and update/rollback of the complete platform tuple.

Lifecycle labels are evidence states, not marketing labels:

| State | Meaning for this lane |
|---|---|
| DETECTED | Identity is recognized by the canonical registry; no graphics qualification is implied. |
| BRINGUP | Work is allowed on a pinned tuple; required tests or hardware evidence are missing. |
| EXPERIMENTAL | Some controlled evidence exists; failures, exclusions, or residuals remain. |
| DAILY_DRIVER | The coordinator has accepted a narrower operational evidence set; this is still not FULL. |
| FULL | Only the canonical program and coordinator may assign this after every applicable physical capability and cross-repository gate passes. |

This document never assigns FULL. No physical board evidence is fabricated from a VM, emulator, static source check, or recognized GPU string.

## 12. Release packaging and signatures

### 12.1 Candidate assembly

The release pipeline assembles Mesa packages only from a verified source/queue/build tuple. Package metadata and manifests carry the source SHA, queue digest, build recipe digest, ABI declarations, artifact hashes, architecture, board capability references, dependency closure, and rollback predecessor.

Each candidate includes:

- Runtime, development, debug/source, and test artifacts according to the platform package policy.
- Signed package indexes and release manifest with mandatory signature verification.
- SBOM, license inventory, source provenance, builder attestation, compiler/build transcript, and reproducibility comparison.
- Exact Hyprland/Quickshell compatibility references and qualification result links.
- Board/generation result table with passing tests, required failures, skips, residuals, and lifecycle state.
- Revocation, expiry, and incident metadata. `Optional TrustAll` is forbidden in a release channel.

Promotion from edge to RC to stable copies an already-built immutable digest. It never rebuilds from a moving branch, replaces an unsigned package, or changes board scope without a new manifest and evidence set. A package is not stable merely because it is available in a repository.

### 12.2 Install and activation guards

Before activation, the consumer validates source/queue/build/package hashes, platform-manifest signature, board identity, kernel/firmware ABI, capability record, and rollback compatibility. A mismatch fails before selecting the new graphics stack. The error names the mismatched field and leaves the last-known-good tuple active.

Mesa packaging must not independently solve a missing kernel or firmware dependency by shipping an untracked substitute. Cross-repository version selection remains in `platform-manifest/v1`.

## 13. Rollback, quarantine, and incident handling

Updates write a complete new graphics/platform tuple to inactive versioned storage, verify signatures and hashes, and select it only after package and manifest checks pass. Boot-health and first-session checks record the selected tuple and success marker. A failed boot, compositor health check, required graphics test, or reset recovery returns to the last-known-good tuple according to the platform rollback contract.

Rollback is atomic across Mesa, kernel, firmware, boot artifacts, and required userspace. Downgrading only Mesa is forbidden when its ABI declaration is incompatible with the active kernel or firmware. The prior tuple remains available until the successor has passed promotion and the board's retention policy permits garbage collection.

An incident response must:

- Stop promotion and quarantine affected artifacts and board/generation records.
- Preserve source, queue, build, package, manifest, logs, traces, and physical evidence IDs.
- State whether the issue is source drift, ABI mismatch, compiler correctness, runtime corruption, reset failure, compositor regression, performance/thermal regression, packaging, signature, or lab fault.
- Reproduce on a disposable target where safe and identify the smallest failing tuple.
- Publish a correction or rollback decision with impact scope and requalification requirements.
- Never mark a failed rollback or unknown recovery outcome as a successful release.

The rollback exercise itself is a required release test: install a known-good predecessor, update to the candidate, inject or reproduce the failure, observe automatic fallback, verify macOS and unrelated platform state are untouched, and record the final active manifest SHA.

## 14. Future-chip intake

New Apple announcements enter intake within one business day, but an announcement does not change the supported set. Intake is a controlled sequence:

1. Create or update the exact board record, separating announced, detected, shipping, bring-up, and qualified states.
2. Acquire physical hardware before making physical, reset, performance, or conformance claims. M6 remains target-only until this condition is met.
3. Obtain the exact kernel/firmware/platform-manifest inputs through the canonical repositories, including opaque boot artifacts as supplied by the coordinator.
4. Run identity and ABI admission tests. Unknown or ambiguous identity fails closed.
5. Identify generation-specific shader/compiler, memory, synchronization, display, and reset work. Do not infer instruction or firmware capabilities from the closest prior generation.
6. Build the candidate reproducibly on two builders and run compiler, golden, conformance, performance, reset, and Hyprland/Quickshell matrices.
7. Record board-level physical evidence and residuals in `qualification-record/v1`.
8. Publish only the lifecycle state supported by the evidence. Coordinator approval is required for every promotion.

The intake record lists the work that remains for future chips even when no code change is yet possible. This makes “not yet qualified” observable and prevents a source branch, package repository, or marketing page from silently expanding support.

## 15. Gate ledger and acceptance contract

The following ledger is the minimum for G-01 through G-05. All statuses begin TODO; only the coordinator may change a program slice to DONE.

| Gate | Required evidence | Hard failure examples | Status at design time |
|---|---|---|---|
| G-01 mirror | Owned-repository check, exact authoritative/mirror SHA equality, complete source objects, queue base, synchronization report | Any mirror drift, missing object, unexpected owner/source, extra commit on `origin/main` | TODO |
| G-01 queue | Ordered minimal commits, patch digest, upstream disposition, clean rebase report, changed-file census | Unscoped patch, stale base, duplicate functionality, no removal condition, conflict hidden as a local edit | TODO |
| G-01 build/ABI | Pinned Meson recipe, two-builder reproducibility, complete kernel/firmware/board declarations | Byte mismatch, auto-detected mutable input, unknown ABI, unsigned or inconsistent metadata | TODO |
| G-02 M1/M2 | Reference-board physical records plus compiler/API, conformance/golden/performance/reset, compositor, packaging, rollback results | Any required failure, missing physical record, incomplete tuple, false success, unresolved residual | TODO |
| G-03 M3 | Per-board compiler/display/API and physical qualification evidence | M3 inferred from M1/M2 or family name, unsupported fallback, missing topology/reset evidence | TODO |
| G-04 M4 | Per-board compiler/display/API and physical qualification evidence | M4 inferred from M3, missing firmware/ABI declaration, non-reproducible package | TODO |
| G-05 A18/M5/M6 | Per-board intake, physical hardware, generation-specific compiler work, complete matrix, rollback evidence | Announcement treated as support, no shipping hardware, skipped required features, drift or incomplete tuple | TODO |

Stable promotion is structurally blocked unless G-01 source/queue/build/ABI gates pass and the applicable G-02 through G-05 board records contain every required result. Conformance, performance, reset, compositor, and physical evidence are separate required inputs; one cannot substitute for another.

## 16. Reporting format and residuals

Each implementation or review report must include:

- Repository, branch, exact base SHA, exact tip SHA, and verified authoritative SHA.
- Changed-file census and confirmation that no graphics code or canonical program file was modified by the design lane.
- Gate-by-gate `PASS`, `FAIL`, `BLOCKED`, or `NOT RUN` output with commands and relevant evidence IDs.
- Failure census with counts and exact causes; “no known issues” is not a substitute for a run.
- Mirror drift status, patch queue disposition, ABI tuple, builder identities, package digests, and rollback predecessor.
- Physical boards and topologies actually tested, not inferred coverage.
- Deviations, unsupported features, open upstream work, recovery limits, and residual risks.
- A statement that nothing is DONE unless the coordinator has independently integrated and promoted the required slice.

At design time the residuals are deliberate: the build/release gates are not implemented by this documentation change; no physical board was qualified by this lane; no conformance result is claimed; no generation was promoted; the canonical platform schemas remain the source of truth; and the m1n1 artifact boundary remains opaque under the coordinator fence. These residuals must remain visible until the owning implementation and evidence lanes resolve them.
