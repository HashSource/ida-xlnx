# AGENTS.md

## NON-NEGOTIABLE PROGRESS RULE

> EVERY AND ANY PROGRESS (task start, task completion, partial completion, blockers, design choices, test results, and scope changes) MUST be reflected in this `AGENTS.md` file immediately.
>
> If this file is not updated immediately when progress happens, the work is considered incomplete.

---

## 0) Tracking Protocol (Must Follow)

- [ ] 0.1 Keep task statuses current in this file at all times.
  - [ ] 0.1.1 Allowed statuses: `pending`, `in_progress`, `blocked`, `completed`.
  - [ ] 0.1.2 Only one leaf task should be `in_progress` at a time unless parallel work is explicitly noted.
  - [ ] 0.1.3 Any status transition must be mirrored in section "11) Progress Ledger" immediately.
- [ ] 0.2 Record every technical discovery in section "12) Findings Snapshot" immediately.
  - [ ] 0.2.1 Include impacted files.
  - [ ] 0.2.2 Include risk/impact.
  - [ ] 0.2.3 Include next concrete action.
- [ ] 0.3 Record blockers immediately.
  - [ ] 0.3.1 Include exact blocker.
  - [ ] 0.3.2 Include workaround attempt.
  - [ ] 0.3.3 Include unblock condition.
- [ ] 0.4 Maintain strict traceability between spec gaps and implementation tasks.
  - [ ] 0.4.1 Every leaf task maps to at least one UG1283 gap.
  - [ ] 0.4.2 Every completed gap has verification evidence (test/log/manual parse check).

---

## 1) Program Goal

- [ ] 1.1 Achieve robust UG1283-aware parsing behavior with safe handling for unsupported/secure/encrypted features.
  - [ ] 1.1.1 Remove current "happy path only" behavior where feasible.
  - [ ] 1.1.2 When unsupported features remain, explicitly detect and report them.
  - [ ] 1.1.3 Ensure loader never silently misclassifies architecture or ciphertext as executable code.

---

## 2) Architecture and Device Family Support

- [x] 2.1 Expand architecture model beyond current `Unknown/Zynq7000/ZynqMP/PDI`.
  - [x] 2.1.1 Add explicit architecture variants in `src/parser.hpp`.
    - [x] 2.1.1.1 `SpartanUltraScalePlus`. (status: completed)
    - [x] 2.1.1.2 `VersalGen1`. (status: completed)
    - [x] 2.1.1.3 `VersalGen2`. (status: completed)
  - [x] 2.1.2 Ensure format names are architecture-specific in `src/parser.cpp` and surfaced by `src/loader.cpp`. (status: completed)
- [x] 2.2 Implement unambiguous PDI family discrimination logic.
  - [x] 2.2.1 Build a deterministic decision tree in `parse_image`.
    - [x] 2.2.1.1 Gate by `0xAA995566` and `XNLX` checks at correct offsets. (status: completed)
    - [x] 2.2.1.2 Validate family-unique field layout before classifying. (status: completed)
    - [x] 2.2.1.3 Reject/flag if only weak signatures match. (status: completed)
  - [x] 2.2.2 Add Spartan-specific identification guards.
    - [x] 2.2.2.1 Validate Spartan BH layout ending at `0x33C`. (status: completed)
    - [x] 2.2.2.2 Ensure parser does not interpret Spartan `0xC4` as Versal Gen1 meta-header offset. (status: completed)
  - [x] 2.2.3 Add Versal Gen2 identification guards.
    - [x] 2.2.3.1 Validate BH size/layout up to `0x113C` checksum area. (status: completed)
    - [x] 2.2.3.2 Validate Gen2 IHT version/layout expectations. (status: completed)
    - [x] 2.2.3.3 Fail fast with clear message if Gen2 fields are inconsistent. (status: completed)
- [x] 2.3 Implement family-specific parser entry points.
  - [x] 2.3.1 `parse_zynq7000(...)`. (status: completed)
  - [x] 2.3.2 `parse_zynqmp(...)`. (status: completed)
  - [x] 2.3.3 `parse_versal_gen1(...)`. (status: completed)
  - [x] 2.3.4 `parse_spartan(...)`. (status: completed)
  - [x] 2.3.5 `parse_versal_gen2(...)`. (status: completed)
- [x] 2.4 Add unsupported-path safety behavior.
  - [x] 2.4.1 If a family is detected but not fully implemented, populate warnings and avoid unsafe loading decisions. (status: completed)
  - [x] 2.4.2 Ensure `accept()` does not overclaim support when loader cannot safely map partitions. (status: completed)

---

## 3) Processor and Execution-Mode Mapping

- [x] 3.1 Stop hardcoding `processor_name = "arm"`.
  - [x] 3.1.1 Introduce processor-selection metadata in `ParsedImage`.
    - [x] 3.1.1.1 Processor family (`arm`, `mblaze`, etc.). (status: completed)
    - [x] 3.1.1.2 Bitness mode hint (AArch32 vs AArch64 where applicable). (status: completed)
    - [x] 3.1.1.3 Confidence/source of inference. (status: completed)
- [x] 3.2 Implement Versal PLM/PPU MicroBlaze mapping.
  - [x] 3.2.1 Mark PLM partition as MicroBlaze-targeted by default. (status: completed)
  - [x] 3.2.2 Avoid forcing ARM for known MicroBlaze execution contexts. (status: completed)
- [x] 3.3 Implement ZynqMP PMU firmware mapping.
  - [x] 3.3.1 Parse PMUFW region when `pmu_image_length > 0`. (status: completed)
  - [x] 3.3.2 Tag PMUFW partition as MicroBlaze. (status: completed)
- [x] 3.4 Decode destination CPU + execution-state attribute bits and store normalized metadata.
  - [x] 3.4.1 ZynqMP destination CPU mapping (A53/R5/PMU). (status: completed)
  - [x] 3.4.2 ZynqMP A5x execution state mapping (AArch64/AArch32). (status: completed)
  - [x] 3.4.3 Versal destination CPU mapping (A72/R5/PSM/AIE for Gen1, A78/R52/ASU for Gen2). (status: completed)
- [x] 3.5 Loader policy for mixed-CPU images.
  - [x] 3.5.1 Define deterministic processor-selection priority rule. (status: completed)
  - [x] 3.5.2 Emit warnings if selected IDA processor cannot represent all executable partitions. (status: completed)

---

## 4) Boot Header Coverage and Validation

- [x] 4.1 Add checksum validation utilities.
  - [x] 4.1.1 Implement inverse-sum checksum routine for header/partition/IHT checksums. (status: completed)
  - [x] 4.1.2 Add helper APIs that return `valid/invalid/not_present`. (status: completed)
  - [x] 4.1.3 Use helpers for Zynq7000 BH, ZynqMP BH, Versal Gen1 BH, Spartan BH, Versal Gen2 BH. (status: completed)
- [x] 4.2 Expand parsed boot-header metadata for all supported families.
  - [x] 4.2.1 Capture key source and IV fields as structured metadata. (status: completed)
  - [x] 4.2.2 Capture image offsets/lengths with validated bounds checks. (status: completed)
  - [x] 4.2.3 Capture boot attributes for later policy decisions. (status: completed)
- [x] 4.3 Register init and PUF helper data support.
  - [x] 4.3.1 Add explicit struct coverage for register-init regions. (status: completed)
  - [x] 4.3.2 Add explicit struct coverage for PUF helper-data regions. (status: completed)
  - [x] 4.3.3 Record presence, location, size, and checksum/consistency diagnostics. (status: completed)
- [x] 4.4 ZynqMP PMUFW+FSBL handling.
  - [x] 4.4.1 Parse PMUFW and FSBL boundaries correctly when PMUFW is prefixed. (status: completed)
  - [x] 4.4.2 Add separate partition entries for PMUFW and FSBL. (status: completed)
  - [x] 4.4.3 Ensure entry-point semantics are not conflated. (status: completed)

---

## 5) Image Header Table / Image Header / Optional Data Parsing

- [x] 5.1 Implement real image-header traversal for Zynq7000/ZynqMP.
  - [x] 5.1.1 Add missing `zynq7000::ImageHeader` struct in `src/xilinx_boot.hpp`. (status: completed)
  - [x] 5.1.2 Ensure `zynqmp::ImageHeader` is defined in `src/xilinx_boot.hpp` and actually used by parser. (status: completed)
  - [x] 5.1.3 Replace blind `image_header_word_offset * 4 + 0x10` reads with actual IH chain walking. (status: completed)
  - [x] 5.1.4 Validate per-image partition counts against traversed partition links. (status: completed)
- [x] 5.2 Implement Versal Gen1 image-header parsing.
  - [x] 5.2.1 Parse image names from `0x10-0x1C` in each IH. (status: completed)
  - [x] 5.2.2 Assign partition names from owning image context. (status: completed)
  - [x] 5.2.3 Remove synthetic-only naming except as explicit fallback. (status: completed)
- [x] 5.3 Implement Versal optional-data parsing (Gen1 and Gen2 where applicable).
  - [x] 5.3.1 Parse optional-data list entries (`ID`, `Size`, `Data`, `Checksum`). (status: completed)
  - [x] 5.3.2 Validate optional-data checksums. (status: completed)
  - [x] 5.3.3 Preserve optional-data metadata in parsed model for visibility. (status: completed)
- [x] 5.4 Validate IHT checksums for all families that define them.
  - [x] 5.4.1 Emit warning on checksum mismatch. (status: completed)
  - [x] 5.4.2 Keep parsing with degraded-trust mode instead of silent acceptance. (status: completed)

---

## 6) Security, Encryption, and Authentication Awareness

- [x] 6.1 Add security-state model to `PartitionInfo` and `ParsedImage`.
  - [x] 6.1.1 `is_encrypted`. (status: completed)
  - [x] 6.1.2 `has_auth_certificate`. (status: completed)
  - [x] 6.1.3 `checksum_type` / `hash_algo` where derivable. (status: completed)
  - [x] 6.1.4 `security_warnings[]`. (status: completed)
- [x] 6.2 Parse authentication-certificate offsets and minimal headers.
  - [x] 6.2.1 Zynq7000 AC offset handling. (status: completed)
  - [x] 6.2.2 ZynqMP AC offset handling. (status: completed)
  - [x] 6.2.3 Versal hash-block/AC offsets handling. (status: completed)
  - [x] 6.2.4 Do not treat AC blobs as executable code. (status: completed)
- [x] 6.3 Encrypted partition safety behavior.
  - [x] 6.3.1 Detect encryption via attributes and key-select fields. (status: completed)
  - [x] 6.3.2 Do not silently load ciphertext as executable code. (status: completed)
  - [x] 6.3.3 Either skip load or load as data with explicit warning and tag. (status: completed)
  - [x] 6.3.4 Surface warning messages in loader output and comments. (status: completed)
- [x] 6.4 Secure-header awareness (at least structural parsing).
  - [x] 6.4.1 Parse secure-header-related fields and offsets. (status: completed)
  - [x] 6.4.2 Record key-rolling metadata presence for ZynqMP. (status: completed)
  - [x] 6.4.3 Record partition-level IV/KEK info for Versal families. (status: completed)

---

## 7) Partition Header Attributes and Semantic Loading

- [ ] 7.1 Add explicit bitfield decoding utilities per family.
- [x] 7.1 Add explicit bitfield decoding utilities per family.
  - [x] 7.1.1 Zynq7000 attributes decode. (status: completed)
  - [x] 7.1.2 ZynqMP attributes decode. (status: completed)
  - [x] 7.1.3 Versal Gen1 attributes decode. (status: completed)
  - [x] 7.1.4 Versal Gen2 attributes decode. (status: completed)
- [x] 7.2 Persist decoded attributes in `PartitionInfo`.
  - [x] 7.2.1 Destination CPU. (status: completed)
  - [x] 7.2.2 Destination device (PS/PL/other). (status: completed)
  - [x] 7.2.3 Partition type (ELF/CDO/Cframe/Raw/etc.). (status: completed)
  - [x] 7.2.4 Exec state / exception level / trustzone. (status: completed)
  - [x] 7.2.5 Endianness and handoff/delay flags. (status: completed)
- [x] 7.3 Loader segment policy by partition semantics.
  - [x] 7.3.1 Executable CPU partitions -> `CODE` segments. (status: completed)
  - [x] 7.3.2 Non-executable config partitions (bitstream/CDO/Cframe) -> data/non-code segments. (status: completed)
  - [x] 7.3.3 `0xFFFFFFFF` load-address partitions handled safely (no invalid mapping). (status: completed)
  - [x] 7.3.4 Entry points added only for executable CPU partitions. (status: completed)
- [x] 7.4 Emit attribute-derived diagnostics.
  - [x] 7.4.1 Warn on big-endian partitions if unsupported. (status: completed)
  - [x] 7.4.2 Warn when destination device is PL but code mapping is requested. (status: completed)
  - [x] 7.4.3 Warn on delay-load/delay-handoff semantics not yet implemented. (status: completed)

---

## 8) Struct Accuracy and Codebase Hygiene

- [ ] 8.1 Correct `src/xilinx_boot.hpp` struct fidelity.
  - [ ] 8.1.1 Extend `versal::BootHeader` to include all documented fields through checksum/SHA3 padding.
  - [ ] 8.1.2 Add missing `zynq7000::ImageHeader`.
  - [ ] 8.1.3 Ensure `zynqmp::ImageHeader` exists in the canonical header used by parser.
  - [ ] 8.1.4 Add optional-data-related structures for Versal IHT post-region parsing.
- [ ] 8.2 Remove or reconcile duplicate schema definitions.
  - [ ] 8.2.1 Evaluate `src/zynqmp_boot.hpp` duplication.
  - [ ] 8.2.2 Eliminate dead/unused duplicated struct definitions or redirect includes to one source of truth.
- [x] 8.3 Add compile-time layout checks where safe.
  - [x] 8.3.1 `static_assert(sizeof(...))` for fixed-size headers. (status: completed)
  - [x] 8.3.2 Offset sanity checks for critical fields. (status: completed)

---

## 9) Hexpat Spec Artifact Corrections

- [ ] 9.1 Correct known pointer-scaling issue in `xilinx_bootgen.hexpat`.
  - [ ] 9.1.1 Review `pointer_word` implementation.
  - [ ] 9.1.2 Confirm proper word-to-byte scaling.
  - [ ] 9.1.3 Re-validate all pointer-based fields after correction.
- [ ] 9.2 Align hexpat structures with parser canonical structs.
  - [ ] 9.2.1 Ensure field names and offsets match.
  - [ ] 9.2.2 Add missing family definitions or clearly mark unsupported families.

---

## 10) Verification Matrix (Tests + Evidence)

- [ ] 10.1 Unit tests for architecture discrimination.
  - [ ] 10.1.1 Zynq7000 valid image.
  - [ ] 10.1.2 ZynqMP valid image.
  - [ ] 10.1.3 Versal Gen1 valid image.
  - [ ] 10.1.4 Spartan valid image (not misdetected as Versal Gen1).
  - [ ] 10.1.5 Versal Gen2 valid image (not misdetected as Gen1/Spartan).
- [ ] 10.2 Unit tests for checksum validation behavior.
  - [ ] 10.2.1 Valid checksums accepted.
  - [ ] 10.2.2 Invalid checksums produce warnings/degraded-trust markers.
- [ ] 10.3 Unit tests for parser semantics.
  - [ ] 10.3.1 Real IH chain name extraction for Zynq7000/ZynqMP.
  - [ ] 10.3.2 Versal IH names replace synthetic defaults when present.
  - [ ] 10.3.3 Optional-data parsing and checksums.
  - [ ] 10.3.4 Encrypted partition detection and safe-load policy.
  - [ ] 10.3.5 AC offset detection and non-code treatment.
- [ ] 10.4 Loader behavior tests.
  - [ ] 10.4.1 Code vs data segment classification by partition type/device.
  - [ ] 10.4.2 Entry-point creation only for executable partitions.
  - [ ] 10.4.3 Processor selection policy in mixed images.
- [ ] 10.5 Regression hardening.
  - [ ] 10.5.1 Build Debug and Release.
  - [ ] 10.5.2 Ensure tests do not rely solely on `assert` in Release.
  - [ ] 10.5.3 Add deterministic synthetic image fixtures for all families.

---

## 11) Progress Ledger (Append Immediately on Any Change)

- 2026-03-01: Created this deeply nested UG1283 parity remediation plan and mandatory immediate-update protocol.
- 2026-03-01: Started task 2.1.1.1 (`SpartanUltraScalePlus` enum variant in `src/parser.hpp`) and set status to in_progress.
- 2026-03-01: Completed task 2.1.1.1 by adding `Arch::SpartanUltraScalePlus` to `src/parser.hpp`; verified with `ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 2.1.1.2 (`VersalGen1` enum variant in `src/parser.hpp`) and set status to in_progress.
- 2026-03-01: Completed task 2.1.1.2 by adding `Arch::VersalGen1` to `src/parser.hpp`.
- 2026-03-01: Started task 2.1.1.3 (`VersalGen2` enum variant in `src/parser.hpp`) and set status to in_progress.
- 2026-03-01: Completed task 2.1.1.3 by adding `Arch::VersalGen2` to `src/parser.hpp`.
- 2026-03-01: Started task 2.1.2 (architecture-specific format names surfaced by loader) and set status to in_progress.
- 2026-03-01: Completed task 2.1.2 by adding architecture->format-name mapping in `src/parser.cpp`, classifying Versal Gen1 as a distinct arch for named PDI handling, and updating tests; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 2.1 after finishing 2.1.1.1/2.1.1.2/2.1.1.3/2.1.2.
- 2026-03-01: Started task 2.2.1.1 (strict PDI signature gating at documented offsets) and set status to in_progress.
- 2026-03-01: Completed task 2.2.1.1 by introducing explicit magic gating helpers (`has_magic_at`) at Zynq (`0x20/0x24`) and PDI (`0x10/0x14`) offsets in `src/parser.cpp`.
- 2026-03-01: Completed task 2.2.1.2 by adding family-specific layout validators for Versal Gen1 (`has_versal_gen1_layout`), Spartan (`has_spartan_layout`), and Gen2 IHT (`has_versal_gen2_iht_layout`) in `src/parser.cpp`.
- 2026-03-01: Completed task 2.2.1.3 by rejecting weak PDI matches (classification returns `Arch::Unknown`) and emitting diagnostic logger messages when family validation fails.
- 2026-03-01: Completed task 2.2.2.1 by implementing Spartan boot-header guard checks through `0x33C` checksum-area readability and Spartan-specific source/length sanity constraints in `src/parser.cpp`.
- 2026-03-01: Completed task 2.2.2.2 by preventing Gen1 classification from Spartan `0xC4` values unless a strict Gen1 meta-header + IHT layout is validated.
- 2026-03-01: Completed task 2.2.3.1 by adding Gen2 boot-header layout checks including `0x113C` checksum-area readability and Gen2 source/length constraints.
- 2026-03-01: Completed task 2.2.3.2 by implementing Gen2 IHT layout validation + bounded discovery (`find_versal_gen2_iht_offset`) in `src/parser.cpp`.
- 2026-03-01: Completed task 2.2.3.3 by adding explicit fail-fast diagnostics for inconsistent Gen2 header/IHT combinations.
- 2026-03-01: Added/updated discrimination tests for Versal Gen1/Spartan/Versal Gen2/weak-signature rejection in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 2.2 after finishing 2.2.1.1/2.2.1.2/2.2.1.3/2.2.2.1/2.2.2.2/2.2.3.1/2.2.3.2/2.2.3.3.
- 2026-03-01: Started task 2.3.1 (`parse_zynq7000(...)` family-specific parser entry point) and set status to in_progress.
- 2026-03-01: Completed task 2.3.1 by extracting Zynq7000 parsing logic into `parse_zynq7000(...)` in `src/parser.cpp`.
- 2026-03-01: Started task 2.3.2 (`parse_zynqmp(...)` family-specific parser entry point) and set status to in_progress.
- 2026-03-01: Completed task 2.3.2 by extracting ZynqMP parsing logic into `parse_zynqmp(...)` in `src/parser.cpp`.
- 2026-03-01: Started task 2.3.3 (`parse_versal_gen1(...)` family-specific parser entry point) and set status to in_progress.
- 2026-03-01: Completed task 2.3.3 by extracting Versal Gen1 parsing logic into `parse_versal_gen1(...)` in `src/parser.cpp`.
- 2026-03-01: Started task 2.3.4 (`parse_spartan(...)` family-specific parser entry point) and set status to in_progress.
- 2026-03-01: Completed task 2.3.4 by adding dedicated Spartan parse entry point with bounded metadata diagnostics in `src/parser.cpp`.
- 2026-03-01: Started task 2.3.5 (`parse_versal_gen2(...)` family-specific parser entry point) and set status to in_progress.
- 2026-03-01: Completed task 2.3.5 by adding dedicated Versal Gen2 parse entry point with Gen2 probe diagnostics in `src/parser.cpp`.
- 2026-03-01: Refactored `parse_image(...)` dispatch to call family-specific parser entry points via `switch` in `src/parser.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 2.3 after finishing 2.3.1/2.3.2/2.3.3/2.3.4/2.3.5.
- 2026-03-01: Started task 2.4.1 (unsupported-path warnings + safe-load gating) and set status to in_progress.
- 2026-03-01: Completed task 2.4.1 by adding `load_supported` + `warnings` to `ParsedImage`, warning emission helpers, and explicit safe-load disablement for unsupported families (`SpartanUltraScalePlus`, `VersalGen2`, generic `PDI`) in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Started task 2.4.2 (ensure `accept()` does not overclaim unsupported families) and set status to in_progress.
- 2026-03-01: Completed task 2.4.2 by updating `src/loader.cpp` so `accept()` only returns a match when `img.load_supported` is true and `load()` hard-fails with `Error::unsupported` when safe loading is disabled; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Updated tests to assert safe-load policy for supported vs unsupported families in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 2.4 after finishing 2.4.1/2.4.2.
- 2026-03-01: Performed parser cleanup to silence unused-parameter paths in unsupported family entry points (`parse_spartan`, `parse_versal_gen2`) and re-verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 3.1.1.1 (processor family metadata in `ParsedImage`) and set status to in_progress.
- 2026-03-01: Completed task 3.1.1.1 by adding `ProcessorFamily` and `ProcessorSelection` metadata to `ParsedImage` in `src/parser.hpp` and deriving `processor_name` via family->IDA mapping in `src/parser.cpp`.
- 2026-03-01: Started task 3.1.1.2 (bitness mode hint metadata) and set status to in_progress.
- 2026-03-01: Completed task 3.1.1.2 by adding `ArmBitnessHint` and populating per-family hints (`AArch32` for Zynq7000, `Unknown` fallback for ZynqMP/Versal Gen1) in `src/parser.cpp`.
- 2026-03-01: Started task 3.1.1.3 (processor inference confidence/source metadata) and set status to in_progress.
- 2026-03-01: Completed task 3.1.1.3 by adding `ProcessorInferenceConfidence` + `source` fields and populating deterministic inference provenance in `src/parser.cpp`; updated tests in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 3.1.1 after finishing 3.1.1.1/3.1.1.2/3.1.1.3.
- 2026-03-01: Completed parent task 3.1 after removing direct hardcoded `processor_name` assignment sites in favor of processor-selection metadata.
- 2026-03-01: Started task 3.2.1 (mark Versal PLM partition as MicroBlaze-targeted by default) and set status to in_progress.
- 2026-03-01: Completed task 3.2.1 by tagging Versal Gen1 `PLM` partition entries with `ProcessorFamily::MicroBlaze` in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Started task 3.2.2 (avoid forcing ARM for known MicroBlaze execution contexts) and set status to in_progress.
- 2026-03-01: Completed task 3.2.2 by switching Versal Gen1 processor selection to `mblaze` when a PLM partition context is present and updating tests in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 3.2 after finishing 3.2.1/3.2.2.
- 2026-03-01: Started task 3.3.1 (parse ZynqMP PMUFW region when `pmu_image_length > 0`) and set status to in_progress.
- 2026-03-01: Completed task 3.3.1 by parsing PMUFW as a prefixed ZynqMP region and adjusting FSBL bootloader offset by `total_pmu_fw_length` fallback logic in `parse_zynqmp(...)` (`src/parser.cpp`).
- 2026-03-01: Started task 3.3.2 (tag ZynqMP PMUFW partition as MicroBlaze) and set status to in_progress.
- 2026-03-01: Completed task 3.3.2 by emitting `PMUFW` `PartitionInfo` entries with `processor_family=MicroBlaze` in `parse_zynqmp(...)` and adding targeted regression test `test_zynqmp_pmufw_prefix` in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 3.3 after finishing 3.3.1/3.3.2.
- 2026-03-01: Started task 3.4.1 (ZynqMP destination CPU mapping decode) and set status to in_progress.
- 2026-03-01: Completed task 3.4.1 by adding normalized `DestinationCpu` metadata and decoding ZynqMP destination CPU from partition attributes (`11:8`) into `PartitionInfo` in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Started task 3.4.2 (ZynqMP A5x execution state mapping) and set status to in_progress.
- 2026-03-01: Completed task 3.4.2 by decoding ZynqMP A5x execution state from attribute bit `3` into `PartitionInfo.arm_bitness_hint` (`AArch64/AArch32`) in `src/parser.cpp`.
- 2026-03-01: Started task 3.4.3 (Versal destination CPU mapping for Gen1/Gen2) and set status to in_progress.
- 2026-03-01: Completed task 3.4.3 by adding Versal Gen1 and Gen2 destination-CPU decode helpers and applying decoded Gen1 partition metadata in parser paths (including PLM=>PSM context) in `src/parser.cpp`; updated tests in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 3.4 after finishing 3.4.1/3.4.2/3.4.3.
- 2026-03-01: Follow-up refinement for task 3.4.3 by wiring Gen2 destination-CPU decode helper into `parse_versal_gen2(...)` diagnostics path and re-verifying with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 3.5.1 (define deterministic mixed-CPU processor-selection priority policy) and set status to in_progress.
- 2026-03-01: Completed task 3.5.1 by implementing deterministic mixed-family scoring/tie-break policy (`apply_mixed_cpu_policy`) with architecture-aware priority defaults in `src/parser.cpp`.
- 2026-03-01: Started task 3.5.2 (emit warnings for non-representable executable partitions under selected processor) and set status to in_progress.
- 2026-03-01: Completed task 3.5.2 by adding mixed-executable mismatch warnings and regression assertions for policy source/warning behavior in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 3.5 after finishing 3.5.1/3.5.2.
- 2026-03-01: Started task 4.1.1 (inverse-sum checksum helper implementation) and set status to in_progress.
- 2026-03-01: Completed task 4.1.1 by implementing generic inverse-sum checksum validation routine in `src/parser.cpp`.
- 2026-03-01: Completed task 4.1.2 by adding checksum helper status model (`valid/invalid/not_present`) and warning adapter utilities in `src/parser.cpp`.
- 2026-03-01: Completed task 4.1.3 by wiring boot-header checksum validation helpers into Zynq7000/ZynqMP/Versal Gen1/Spartan/Versal Gen2 parse paths in `src/parser.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 4.1 after finishing 4.1.1/4.1.2/4.1.3.
- 2026-03-01: Started task 4.2.1 (structured boot-header key-source/IV metadata capture) and set status to in_progress.
- 2026-03-01: Completed task 4.2.1 by adding structured boot-header security metadata (`key_source`, secure/header IV variants, black-key IV) in `src/parser.hpp` and populating per-family values in `src/parser.cpp`.
- 2026-03-01: Completed task 4.2.2 by adding `BootImageRange` tracking with readability bounds validation for FSBL/PMUFW/PLM/PMC/IHT offsets and lengths in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Completed task 4.2.3 by adding named boot-attribute capture (`BootAttributeWord`) for Zynq7000/ZynqMP/Versal Gen1 fields and wiring regression assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 4.2 after finishing 4.2.1/4.2.2/4.2.3.
- 2026-03-01: Started task 4.3.1 (explicit register-init struct coverage) and set status to in_progress.
- 2026-03-01: Completed task 4.3.1 by adding explicit register-init region structs for Zynq7000/ZynqMP/Versal in `src/xilinx_boot.hpp` and wiring parser diagnostics metadata in `src/parser.cpp`.
- 2026-03-01: Completed task 4.3.2 by adding explicit PUF helper-data structs for ZynqMP/Versal in `src/xilinx_boot.hpp` and metadata capture hooks in `src/parser.cpp`.
- 2026-03-01: Completed task 4.3.3 by recording region presence/offset/size/readability/checksum-state diagnostics via `BootRegionDiagnostic` in `src/parser.hpp` and parser emission; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 4.3 after finishing 4.3.1/4.3.2/4.3.3.
- 2026-03-01: Started task 4.4.1 (ZynqMP PMUFW+FSBL boundary semantics) and set status to in_progress.
- 2026-03-01: Completed task 4.4.1 by refining ZynqMP PMU-prefixed FSBL boundary handling and explicit FSBL offset computation in `parse_zynqmp(...)` (`src/parser.cpp`).
- 2026-03-01: Completed task 4.4.2 by emitting distinct ZynqMP `PartitionInfo` entries for `PMUFW` and synthetic `FSBL` bootloader partition in `src/parser.cpp`; updated assertions in `tests/test_main.cpp`.
- 2026-03-01: Completed task 4.4.3 by adding `is_bootloader_partition` semantics and loader skip logic to prevent duplicate FSBL segment/entrypoint creation in `src/parser.hpp` and `src/loader.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 4.4 after finishing 4.4.1/4.4.2/4.4.3.
- 2026-03-01: Started task 5.1.1 (add canonical `zynq7000::ImageHeader` struct) and set status to in_progress.
- 2026-03-01: Completed task 5.1.1 by adding canonical `zynq7000::ImageHeader` definition to `src/xilinx_boot.hpp`.
- 2026-03-01: Completed task 5.1.2 by adding canonical `zynqmp::ImageHeader` definition to `src/xilinx_boot.hpp` and switching parser traversal to use it.
- 2026-03-01: Completed task 5.1.3 by replacing per-partition blind image-name reads with explicit Zynq7000/ZynqMP image-header chain walking and context lookup in `src/parser.cpp`.
- 2026-03-01: Completed task 5.1.4 by validating image-header partition-count/link consistency against traversed partition headers and emitting targeted mismatch warnings in `src/parser.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 5.1 after finishing 5.1.1/5.1.2/5.1.3/5.1.4.
- 2026-03-01: Normalized section 5 checkbox state by removing stale duplicate unchecked `5.1` line to keep task status unambiguous.
- 2026-03-01: Started task 5.2.1 (Versal IH name parsing from `0x10-0x1C`) and set status to in_progress.
- 2026-03-01: Completed task 5.2.1 by adding canonical `versal::ImageHeader` and parsing fixed-size Versal image names from IH records in `src/xilinx_boot.hpp` and `src/parser.cpp`.
- 2026-03-01: Completed task 5.2.2 by building Versal image-header->partition ownership mapping and assigning partition names from owning image context in `parse_versal_gen1(...)` (`src/parser.cpp`).
- 2026-03-01: Completed task 5.2.3 by switching Versal partition naming to image-context names with synthetic `PDI_PART_*` as explicit fallback only; updated regression coverage in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 5.2 after finishing 5.2.1/5.2.2/5.2.3.
- 2026-03-01: Started task 5.3.1 (Versal optional-data entry parsing) and set status to in_progress.
- 2026-03-01: Completed task 5.3.1 by parsing Versal optional-data entries (`ID`, `Size`, payload, checksum word) from post-IHT regions into `BootHeaderMetadata.optional_data_entries` in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Completed task 5.3.2 by validating optional-data checksums (sum of prior words) and emitting warnings for mismatches in `src/parser.cpp`.
- 2026-03-01: Completed task 5.3.3 by preserving optional-data metadata entries (including checksum status and data words) in parsed image model and adding regression assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 5.3 after finishing 5.3.1/5.3.2/5.3.3.
- 2026-03-01: Started task 5.4.1 (IHT checksum mismatch warnings) and set status to in_progress.
- 2026-03-01: Completed task 5.4.1 by wiring IHT checksum validation warnings for ZynqMP and Versal Gen1/Gen2 parse paths in `src/parser.cpp`.
- 2026-03-01: Completed task 5.4.2 by keeping parse flow non-fatal on IHT checksum mismatch and adding warning-based degraded-trust signaling; validated with `test_zynqmp_iht_checksum_warning` and `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 5.4 after finishing 5.4.1/5.4.2.
- 2026-03-01: Started task 6.1.1 (partition/image security-state model: encryption flag) and set status to in_progress.
- 2026-03-01: Completed task 6.1.1 by adding `is_encrypted` state to `PartitionInfo` and deriving it from Zynq7000/ZynqMP/Versal partition fields in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Completed task 6.1.2 by adding `has_auth_certificate` state and deriving it from AC offset/authentication presence bits/fields in `src/parser.cpp`.
- 2026-03-01: Completed task 6.1.3 by adding normalized partition checksum/hash metadata (`PartitionChecksumType`, `PartitionHashAlgorithm`) and decoding where derivable (Zynq7000/ZynqMP attributes) in `src/parser.hpp` and `src/parser.cpp`.
- 2026-03-01: Completed task 6.1.4 by adding partition/image-level `security_warnings` collections and populating ciphertext-safety warnings during parse; updated assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 6.1 after finishing 6.1.1/6.1.2/6.1.3/6.1.4.
- 2026-03-01: Started task 6.2.1 (Zynq7000 authentication-certificate offset handling) and set status to in_progress.
- 2026-03-01: Completed task 6.2.1 by adding Zynq7000 partition AC offset normalization (`ac_offset * 4`) and minimal AC header capture (`header_words[4]`, readability state) in `src/parser.hpp` and `src/parser.cpp`; added regression assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 6.2.2 (ZynqMP authentication-certificate offset handling) and set status to in_progress.
- 2026-03-01: Completed task 6.2.2 by adding ZynqMP partition AC offset normalization (`ac_offset * 4`) and minimal AC header capture (`header_words[4]`, readability state) in `src/parser.cpp`; added regression assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 6.2.3 (Versal hash-block/AC offsets handling) and set status to in_progress.
- 2026-03-01: Completed task 6.2.3 by parsing Versal partition auth metadata from `hash_block_ac_offset` with `authentication_header` fallback, adding deterministic dual-offset handling warning, and capturing minimal AC header words/readability in `src/parser.cpp`; added regression assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 6.2.4 (ensure AC blobs are never treated as executable code) and set status to in_progress.
- 2026-03-01: Completed task 6.2.4 by adding loader guardrails that skip partition mapping when payload offset collides with parsed AC metadata offset, preventing AC blobs from being mapped as executable segments in `src/loader.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 6.3.1 (detect encryption via attributes and key-select fields) and set status to in_progress.
- 2026-03-01: Completed task 6.3.1 by confirming/retaining parser-side encryption derivation from per-family partition fields (Zynq7000 encrypted-length, ZynqMP attr+length, Versal key-select) and exposing those signals in `PartitionInfo.is_encrypted` used by loader policy.
- 2026-03-01: Completed task 6.3.2 by changing loader partition mapping policy to avoid creating `CODE` segments for encrypted partitions in `src/loader.cpp`.
- 2026-03-01: Completed task 6.3.3 by loading encrypted partitions as `DATA` segments (not `CODE`) and suppressing entry-point creation for encrypted payloads in `src/loader.cpp`.
- 2026-03-01: Completed task 6.3.4 by surfacing partition/image security warnings in loader output (`SECURITY WARNING` messages) during load in `src/loader.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent task 6.3 after finishing 6.3.1/6.3.2/6.3.3/6.3.4.
- 2026-03-01: Started task 6.4.1 (secure-header structural fields/offset metadata) and set status to in_progress.
- 2026-03-01: Completed task 6.4.1 by capturing secure-header-related auth/hash offsets as structured boot attributes/ranges (Zynq7000/ZynqMP IHT AC, Versal meta-header AC/auth header/hash block) in `src/parser.cpp` and validating via parser tests in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed task 6.4.2 by recording ZynqMP key-rolling presence/words from `obfuscated_black_key_storage` into `BootHeaderMetadata` in `src/parser.hpp` and `src/parser.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed task 6.4.3 by recording Versal partition-level IV and IV/KEK vectors in `PartitionInfo` and populating them in `parse_versal_gen1(...)` (`src/parser.hpp`, `src/parser.cpp`); added assertions in `tests/test_main.cpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Started task 8.3.1 (`static_assert` struct-size layout checks) and set status to in_progress.
- 2026-03-01: Completed task 8.3.1 by adding compile-time `static_assert(sizeof(...))` checks for Zynq7000/ZynqMP/Versal boot/image/partition structs and fixed-size helper regions in `src/xilinx_boot.hpp`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed task 8.3.2 by adding `offsetof(...)` sanity checks for critical security/layout fields in `src/xilinx_boot.hpp` and correcting `zynqmp::ImageHeaderTable` padding from 8 to 9 words to preserve documented `checksum @ 0x3C`; verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).
- 2026-03-01: Completed parent tasks 6.2, 6.4, and 8.3 after finishing all corresponding leaf tasks.
- 2026-03-01: Started task 7.1.1 (Zynq7000 partition-attribute bitfield decode utility) and set status to in_progress.
- 2026-03-01: Completed task 7.1.1 by adding explicit Zynq7000 attribute decode utility (`decode_zynq7000_attributes`) and wiring decoded destination-device/partition-type heuristics into partition metadata in `src/parser.cpp`.
- 2026-03-01: Started task 7.1.2 (ZynqMP partition-attribute bitfield decode utility) and set status to in_progress.
- 2026-03-01: Completed task 7.1.2 by adding explicit ZynqMP attribute decode utility (`decode_zynqmp_attributes`) covering destination CPU/device, exec-state, exception level, trustzone, endian, and early-handoff bits in `src/parser.cpp`.
- 2026-03-01: Started task 7.1.3 (Versal Gen1 partition-attribute bitfield decode utility) and set status to in_progress.
- 2026-03-01: Completed task 7.1.3 by adding explicit Versal Gen1 attribute decode utility (`decode_versal_gen1_attributes`) covering partition type, destination CPU/device derivation, endian, exec-state, EL, and trustzone in `src/parser.cpp`.
- 2026-03-01: Started task 7.1.4 (Versal Gen2 partition-attribute bitfield decode utility) and set status to in_progress.
- 2026-03-01: Completed task 7.1.4 by adding explicit Versal Gen2 attribute decode utility (`decode_versal_gen2_attributes`) including destination cluster + lockstep flags and wiring it into Gen2 probe diagnostics in `src/parser.cpp`.
- 2026-03-01: Completed parent task 7.1 after finishing 7.1.1/7.1.2/7.1.3/7.1.4.
- 2026-03-01: Completed tasks 7.2.1/7.2.2/7.2.3/7.2.4/7.2.5 by expanding `PartitionInfo` with decoded semantic fields (destination device, partition type, exception level, trustzone validity/state, endianness, handoff/delay, cluster/lockstep) in `src/parser.hpp` and populating them in parser paths in `src/parser.cpp`.
- 2026-03-01: Completed parent task 7.2 after persisting decoded semantic attributes in `PartitionInfo`.
- 2026-03-01: Completed tasks 7.3.1/7.3.2/7.3.3/7.3.4 by updating loader segment/entry policy to classify executable vs non-executable partitions semantically, map config/encrypted/non-exec partitions as `DATA`, skip `0xFFFFFFFF` load-address mappings with warning, and only create entry points for executable CPU partitions in `src/loader.cpp`.
- 2026-03-01: Completed parent task 7.3 after semantic loader policy wiring.
- 2026-03-01: Completed tasks 7.4.1/7.4.2/7.4.3 by adding attribute-derived parser diagnostics for unsupported big-endian partitions, PL-target + exec-address conflicts, and delay-load/delay-handoff semantics in `src/parser.cpp`.
- 2026-03-01: Completed parent task 7.4 and overall task group 7; extended regression assertions in `tests/test_main.cpp` (`test_zynqmp_attribute_diagnostics`, enhanced Zynq7000/Versal checks) and verified with `cmake --build . && ctest --output-on-failure` (1/1 tests passed).

---

## 12) Findings Snapshot (Append Immediately on Discovery)

- 2026-03-01: Current parser implements a limited happy-path subset and hardcodes ARM processor mapping across all detected families.
- 2026-03-01: Current parser can misclassify PDI families because detection is broad and not guarded by family-specific layout validation.
- 2026-03-01: Current loader treats all mapped partitions as `CODE`, including partitions that can be non-executable/configuration data.
- 2026-03-01: Loader already surfaces parser-provided format names directly via `accept()`; impacted files: `src/parser.cpp`, `src/loader.cpp`; risk/impact: parser naming is the single source of truth for UI format labels; next concrete action: keep canonical family names in parser and add deterministic family discrimination in task 2.2.
- 2026-03-01: Current Versal Gen1 naming path uses a provisional `meta_header_offset` heuristic (`0xC4` nonzero + word-aligned); impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: ambiguous PDI cases can still fall back to generic `Arch::PDI`; next concrete action: implement strict family guards in task 2.2.1-2.2.3.
- 2026-03-01: PDI detection now uses deterministic family discrimination (Gen2 -> Spartan -> Gen1) with hard layout checks and weak-signature rejection; impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: stricter checks can classify malformed edge samples as `Unknown` instead of permissive generic PDI; next concrete action: add targeted fixture corpus for borderline real-world images in verification tasks 10.1.x.
- 2026-03-01: Gen2 validation currently uses bounded IHT discovery heuristics anchored from header-derived offsets + forward scan; impacted files: `src/parser.cpp`; risk/impact: unusual but valid Gen2 layouts outside scan assumptions may need broader discovery policy; next concrete action: refine IHT location strategy once canonical Gen2 fixtures are added.
- 2026-03-01: Parser now uses explicit family-specific entry points (`parse_zynq7000`, `parse_zynqmp`, `parse_versal_gen1`, `parse_spartan`, `parse_versal_gen2`) behind a single dispatch in `parse_image`; impacted files: `src/parser.cpp`; risk/impact: spartan/gen2 entry points currently provide diagnostics but do not yet map full partitions; next concrete action: implement unsupported-path safety behavior and warnings in tasks 2.4.1-2.4.2.
- 2026-03-01: Parser now carries explicit safe-load policy (`load_supported`) and warning messages per image; impacted files: `src/parser.hpp`, `src/parser.cpp`; risk/impact: unsupported families are no longer silently loadable, reducing unsafe mappings but changing prior permissive behavior; next concrete action: implement full partition parsers for Spartan/Gen2 to flip safe-load support in future tasks.
- 2026-03-01: Loader `accept()` now declines unsupported families by policy and `load()` returns a structured `unsupported` error if invoked unsafely; impacted files: `src/loader.cpp`; risk/impact: users can no longer accidentally load unsupported families through this loader, but those images require future implementation before they appear selectable; next concrete action: complete 3.x processor mapping and 4.x/5.x metadata coverage for supported families.
- 2026-03-01: Parser now records processor inference metadata (`family`, `arm_bitness_hint`, `confidence`, `source`) and derives `processor_name` from family mapping instead of direct literal assignment; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: Versal Gen1 currently remains an ARM low-confidence fallback until destination-CPU decoding (tasks 3.2/3.4) is implemented; next concrete action: implement MicroBlaze-aware mapping for Versal PLM in task 3.2.
- 2026-03-01: Versal Gen1 now defaults to MicroBlaze processor selection when PLM context is present, and PLM partitions are explicitly tagged MicroBlaze; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: mixed-CPU Versal images may still require a smarter image-level processor arbitration policy; next concrete action: implement destination-CPU attribute decoding (task 3.4) and mixed-image processor policy (task 3.5).
- 2026-03-01: ZynqMP parsing now models PMUFW as a prefixed MicroBlaze partition when `pmu_image_length > 0` and shifts FSBL bootloader offset past the PMU prefix using `total_pmu_fw_length` fallback logic; impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: PMU load/exec address is currently inferred with a fixed base and may need attribute-driven refinement; next concrete action: implement destination-CPU/exec-state decoding in task 3.4 and full PMU/FSBL boundary semantics in task 4.4.
- 2026-03-01: Partition metadata now decodes destination CPU and AArch32/AArch64 execution hints from ZynqMP and Versal Gen1 partition attributes, and includes a Gen2 CPU decode helper for future full Gen2 partition parsing; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: Gen2 decode is currently helper-only because Gen2 partition traversal is not implemented yet; next concrete action: define mixed-image processor arbitration policy in task 3.5.
- 2026-03-01: Parser now applies a deterministic mixed-CPU arbitration policy and emits warnings when selected processor cannot represent all executable partitions; impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: policy currently uses scoring heuristics and may need refinement once richer partition semantics (type/device/entry semantics) are implemented; next concrete action: calibrate policy with real mixed-image fixtures under verification task 10.4.3.
- 2026-03-01: Parser now includes reusable checksum validators with `not_present/valid/invalid` states and emits degraded-trust warnings on boot-header checksum mismatches across all currently detected families; impacted files: `src/parser.cpp`; risk/impact: sparse synthetic samples without checksum words remain `not_present` and do not warn, while malformed checksummed samples now surface explicit trust degradation; next concrete action: extend checksum helper usage to partition/IHT checksums in tasks 5.4 and 10.2.
- 2026-03-01: Parsed images now retain structured boot-header metadata (`BootHeaderMetadata`) including key source/IV sets, validated image ranges, and boot attribute words for supported families; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: range validation is readability-based and can under-report if a truncated sample still exposes first/last byte, but materially improves offset/length sanity visibility; next concrete action: extend metadata coverage to register-init and PUF helper regions in task 4.3.
- 2026-03-01: Parser metadata now includes explicit register-init and PUF helper diagnostics (`BootRegionDiagnostic`) backed by dedicated struct definitions in `xilinx_boot.hpp`; impacted files: `src/xilinx_boot.hpp`, `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: region diagnostics are structural/readability-based and checksum state is currently `NotPresent` for these regions pending deeper per-region validation rules; next concrete action: implement ZynqMP PMUFW+FSBL boundary semantics in task 4.4.
- 2026-03-01: ZynqMP parser now models PMUFW and FSBL as distinct partition records with a dedicated bootloader flag to avoid duplicate loader mappings and conflated entry-point semantics; impacted files: `src/parser.hpp`, `src/parser.cpp`, `src/loader.cpp`, `tests/test_main.cpp`; risk/impact: FSBL can appear both as synthetic bootloader and as PH-derived entry in unusual images, requiring future dedupe heuristics across partition traversal; next concrete action: begin task 5.1 real image-header traversal to replace heuristic naming/partition relationships.
- 2026-03-01: Zynq7000/ZynqMP naming and partition accounting now use explicit image-header chain traversal with per-image link/count validation rather than per-partition offset heuristics; impacted files: `src/xilinx_boot.hpp`, `src/parser.cpp`; risk/impact: malformed images now emit more warnings and may fall back to synthetic names when IH links are inconsistent; next concrete action: implement Versal image-header parsing in task 5.2.
- 2026-03-01: Versal Gen1 partition naming now derives from parsed image-header contexts (0x10-0x1C names) with ownership linkage to partition chains; impacted files: `src/xilinx_boot.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: malformed image-header ownership links can still trigger fallback synthetic names with warnings; next concrete action: implement optional-data parsing and checksums in task 5.3.
- 2026-03-01: Versal metadata now includes parsed optional-data entries with checksum validation for Gen1 and probe-path Gen2 IHT reads; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: optional-data parsing relies on `optional_data_length` and can under-parse malformed chained entries after the first structural inconsistency; next concrete action: implement IHT checksum validation/degraded-trust behavior in task 5.4.
- 2026-03-01: IHT checksum validation now emits explicit degraded-trust warnings for ZynqMP and Versal IHT structures while preserving parse continuity; impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: checksum-mismatched images are still traversed, so downstream loaders must continue honoring warning signals; next concrete action: begin 6.x security-state modeling to make trust posture explicit in partition metadata.
- 2026-03-01: Partition and image models now expose explicit security posture fields (`is_encrypted`, `has_auth_certificate`, checksum/hash hints, `security_warnings`) with parser-side derivation for Zynq7000/ZynqMP/Versal Gen1 where fields are available; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: this is structural awareness only and does not yet parse full authentication certificate bodies or secure headers; next concrete action: implement 6.2 AC offset/header handling and 6.3 ciphertext-safe loader behavior.
- 2026-03-01: Authentication-certificate presence is currently inferred from AC-offset/attribute bits but AC blob bounds/metadata are not parsed as first-class partition-adjacent regions; impacted files: `src/parser.cpp`, `src/xilinx_boot.hpp`; risk/impact: parser can detect security intent without surfacing AC placement integrity details; next concrete action: add AC offset normalization and minimal header/bounds diagnostics for Zynq7000 first (task 6.2.1).
- 2026-03-01: Zynq7000 partition parsing now captures AC metadata (`present`, normalized byte `offset`, `header_readable`, first 4 header words) and emits partition security warning when AC offset is present but unreadable; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: AC layout is still minimally parsed (no full certificate schema validation), but placement integrity is now surfaced in parsed model; next concrete action: mirror the same offset/header treatment for ZynqMP in task 6.2.2.
- 2026-03-01: ZynqMP partition parsing now mirrors Zynq7000 AC metadata capture (`present`, normalized byte `offset`, `header_readable`, first 4 header words) with unreadable-header security warnings; impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: AC interpretation remains structural/minimal and currently only covers partition-level AC word offsets; next concrete action: extend equivalent handling to Versal `hash_block_ac_offset` and `authentication_header` fields in task 6.2.3.
- 2026-03-01: Versal partition parsing now captures AC metadata from hash-block/auth-header offsets with stable precedence (`hash_block_ac_offset` first) and unreadable-header warnings; impacted files: `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: when both offsets differ parser records only the prioritized AC header payload and flags the mismatch as security context, so full dual-certificate body parsing remains future work; next concrete action: enforce loader behavior that never treats AC/ciphertext as executable code (tasks 6.2.4 and 6.3.x).
- 2026-03-01: Loader now emits parser-derived security warnings, maps encrypted partitions as `DATA`, suppresses encrypted entry points, and blocks payload mappings that collide with parsed AC offsets; impacted files: `src/loader.cpp`; risk/impact: this strengthens ciphertext/AC safety but may reduce auto-analysis convenience for samples that embed decryptable stubs in encrypted-tagged partitions; next concrete action: implement 6.4 secure-header structural metadata so loader diagnostics can reference secure-header/key-rolling context explicitly.
- 2026-03-01: Existing boot metadata captures secure-header IV vectors but not explicit key-rolling presence nor partition-level Versal IV/KEK material; impacted files: `src/parser.hpp`, `src/parser.cpp`; risk/impact: loader warnings cannot currently point analysts to key-rolling/partition IV context when security features are present; next concrete action: add structural metadata fields and populate them in 6.4.1-6.4.3.
- 2026-03-01: Secure-header structural metadata now includes key security offsets/ranges (IHT/Versal auth/hash fields), ZynqMP key-rolling presence, and Versal partition IV/IV-KEK vectors; impacted files: `src/parser.hpp`, `src/parser.cpp`, `tests/test_main.cpp`; risk/impact: metadata is visibility-focused and does not yet validate full secure-header/certificate schemas; next concrete action: move to 7.x semantic attribute decoding + loader policy to classify executable vs configuration partitions.
- 2026-03-01: New compile-time layout assertions exposed a latent ZynqMP IHT schema drift (`checksum` expected at `0x3C` but struct packed to `0x38` due short padding); impacted files: `src/xilinx_boot.hpp`; risk/impact: incorrect struct size can silently skew field interpretation in future direct-struct reads; next concrete action: keep canonical struct assertions enabled and reconcile duplicate `src/zynqmp_boot.hpp` definitions under task 8.2.
- 2026-03-01: Parser now decodes per-family partition attribute semantics into normalized metadata (device/type/EL/TZ/endian/handoff/cluster flags), and loader uses those semantics for `CODE` vs `DATA` mapping + entrypoint gating; impacted files: `src/parser.hpp`, `src/parser.cpp`, `src/loader.cpp`, `tests/test_main.cpp`; risk/impact: some Zynq7000/ZynqMP partition-type values are inferred heuristically because legacy headers do not encode explicit type, so future fixture calibration is needed; next concrete action: execute task group 8 (struct fidelity + duplicate schema reconciliation) and then strengthen verification matrix in task group 10.
