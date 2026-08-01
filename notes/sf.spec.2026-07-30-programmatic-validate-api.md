---
id: sfprogvalidateapi20260730
title: 2026 07 30 Programmatic Validate API Behavior
desc: ''
updated: 1785440266000
created: 1785440266000
---

## Purpose

This note specifies portable observable behavior for programmatic validation of a Semantic Flow mesh: a read-only operation that reports the mesh's validity as structured findings with stable machine discriminants.

It specifies operation outcomes and invariants, not a particular programming language, host filesystem, error class, command-line surface, or implementation mechanism. The Weave-specific API contract, including the exact finding-code registry and refusal-family mapping, is `wd.programmatic-validate-api` in the Weave docs.

## Status

Ratified 2026-07-30 (spec review r1 of the Weave task `wa.task.2026.2026-07-29_1219-programmatic-validate-mesh-api`); first implementation built the same day. The first slice validates mesh scope only over mesh-local sources; publication-scope selection, repository-backed/floating sources, and comprehensive integrity traversal are outside this slice.

## Operation

A request supplies one explicit mesh and optionally a set of validation targets (exact designators, each optionally recursive). An absent or empty target set means the whole mesh. The mesh root designator has one unambiguous public representation, distinct from any host storage root.

The operation mutates nothing, acquires no lock, and is safely repeatable. An unlocked read can observe a concurrent writer mid-operation; coherence gates require caller-owned serialization.

## Findings Are The Result, Not An Exception

Mesh invalidity is reported as result data — an ordered collection of findings, each carrying:

- a severity from a closed set (the first slice emits only the error severity; a warning severity is reserved)
- a stable machine code from a closed, documented registry; codes never change meaning and are only ever added
- diagnostic message text that is never a machine contract
- optional attribution to a mesh-relative path and/or a designator

A thrown operation failure is reserved for cannot-validate conditions: an invalid request, an unreadable environment (I/O and permission failures, distinguished from mesh-content problems by a dedicated read-failure discriminant), the absence of any mesh at the supplied root, unmatched targets, or a source-capability limitation. Genuinely unexpected implementation defects propagate as raw errors rather than masquerading as typed refusals.

Valid-but-unsupported mesh shapes (implementation-limitation gates) are findings under a single stable code, so their occurrences can shrink as the implementation generalizes without registry churn.

## Coverage Honesty

The result carries explicit coverage counts: how many designators the mesh declares, and how many pending candidates completed dry-run planning. Implementations must present validation coverage as what it is — planning/preflight plus publication-readiness checks — and must not imply comprehensive per-file integrity traversal they do not perform. A derived boolean validity field is deliberately absent; the findings collection is the single source of truth.

The first slice preserves fail-fast planning: a run reports at least the first blocking mesh finding; exhaustive multi-finding collection is future growth, not a current promise.

## Equivalence

Within the shared capability domain, command-line validation and programmatic validation consume one findings pipeline: identical inputs yield identical findings, with the command-line text being a rendering of the structured findings.
