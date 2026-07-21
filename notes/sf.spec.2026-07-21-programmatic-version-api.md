---
id: sfprogversionapi20260721
title: 2026 07 21 Programmatic Version API Behavior
desc: ''
updated: 1784667355000
created: 1784667355000
---

## Purpose

This note specifies portable observable behavior for recording caller-supplied payload content into a Semantic Flow mesh as one coherent version request.

It specifies operation outcomes and invariants, not a particular programming language, host filesystem, error class, command-line surface, or implementation mechanism. The Weave-specific API contract is [[wd.programmatic-version-api]].

## Status

This is the pre-build behavior contract for the first programmatic version API slice. Implementation is STOP-gated on PM approval in [[wa.task.2026.2026-07-21_1322-programmatic-version-api]].

The first slice is limited to existing, mesh-local, UTF-8 text/RDF payload artifacts selected by exact designator path. Binary payloads, recursive selection, repository-backed or floating sources, payload-IRI input, whole-mesh validation, and cross-writer coordination are outside this slice.

## Operation

A request supplies:

- one explicit mesh
- one or more items, each containing an exact payload designator and payload bytes
- optional per-item history, state, and manifestation segments
- optional request-wide segment defaults
- optional history-tracking policy override
- optional permission to overwrite an existing state

The operation records the supplied items as one coherent batch, including when the batch contains exactly one item.

The mesh root designator is a valid exact payload designator. Implementations must expose one unambiguous public representation for it and must not confuse it with a host storage root.

## Admission And Capture Boundary

The operation copies every supplied byte sequence at admission. Those admitted copies are authoritative for the rest of the request.

Consequently:

- caller mutation of a supplied buffer after admission cannot change the recorded payload
- multiple items backed by the same caller-owned buffer or overlapping views are each governed by their own admitted copy
- the operation validates that each admitted copy is valid UTF-8 before loading or planning the mesh transition
- non-text content, invalid UTF-8, and content that is ineligible for the target text/RDF payload refuse before planning
- the working payload content that existed before admission is not a competing capture source and does not produce a changed-under-capture refusal

There is no analogue of a working-file changed-under-capture check for the supplied payload bytes. Changes by another writer after the request has passed planning are outside the supported concurrency contract; they are not reclassified as capture conflicts.

The caller owns serialization of all mutation against a mesh. The operation does not promise locking, rollback, journaling, or coordination with another API or command-line writer.

## Identity And Eligibility

Every item identifies exactly one existing payload artifact by exact designator path.

The request refuses without mutation when:

- it contains no items
- a designator is malformed or denotes a recursive selection
- two items normalize to the same designator
- a designator is unknown
- a designator exists but does not denote a payload artifact
- the payload working source is not mesh-local
- the target's declared content kind is not supported by this text/RDF operation
- an admitted payload is empty, whitespace-only, or otherwise ineligible for the target content kind

The root designator is eligible when it denotes an otherwise eligible existing payload artifact.

## Batch Coherence

Every request is a coherent batch. Cardinality one is not a weaker single-target mode.

The operation:

- resolves, validates, and plans every member before any mesh mutation
- uses deterministic canonical designator order for planning and result order
- combines shared support-resource progression into one coherent plan
- refuses the entire request if any member or shared progression is invalid or conflicting
- leaves the whole mesh unchanged after any admission, load, or planning refusal

No working payload, historical snapshot, current pointer, inventory, or other version output may change before the whole request is plan-green.

This pre-write refusal guarantee is semantic, not a storage transaction. Once writing begins, an operational write failure can leave a disclosed partial result. The operation does not promise rollback of those writes.

Overwrite is deliberately narrower in the first slice:

- overwrite may be requested only for a one-item batch
- an overwrite request must identify the existing current history and state it intends to replace
- a multi-item overwrite request refuses as an invalid request before any mesh mutation

## Naming And Policy Precedence

For each naming decision, the first applicable source wins in this order:

1. the item's explicit history, state, or manifestation segment
2. the corresponding request-wide default
3. persisted payload intent, including the selected current history and the selected history's next-state hint
4. the effective target, nearest-ancestor, then mesh naming policy
5. the built-in naming default

Persisted intent contributes only the values it actually defines. In particular, lack of persisted manifestation intent does not block policy or built-in manifestation naming.

An explicit history-tracking policy override applies to the whole request and wins over persisted configuration for that invocation. Overwrite permission is also request-explicit and defaults to disabled; it is not inferred from persisted intent or configuration.

If request members resolve to planning policies that cannot participate in one coherent batch, the entire request refuses. An implementation must not silently choose one member's policy for the others.

Configuration and shared support resources are load and planning inputs, but they are not represented as covered by the payload capture boundary. The caller-owned single-writer rule applies if they are changed concurrently.

## Observable State Transitions

### Applied, normal advancement

For an item reported as applied:

- its working payload becomes byte-identical to the admitted bytes
- a first or next historical state is materialized according to the resolved naming and policy
- the historical payload snapshot for that state is byte-identical to the admitted bytes
- current history, current state, and latest-state relations resolve to the applied state
- required shared support and inventory progression is coherent with every applied member
- older historical states and their payload snapshots remain unchanged

The operation returns the resolved payload identity and the selected history, state, manifestation, and snapshot location for each applied item.

### Applied, explicit overwrite

For an allowed one-item overwrite:

- the item identifies an existing current history and current state
- no new historical state is introduced merely because overwrite was requested
- the working payload and the selected current historical snapshot become byte-identical to the admitted bytes
- an attempt to overwrite a missing, non-current, or differently manifested state refuses before mutation

### Already current

An item is already current when its admitted bytes and requested resolved naming already describe its current latest recorded state.

An already-current item:

- is a successful per-item outcome, not a refusal
- creates no new history or state
- causes no payload or support-resource mutation solely for that item
- reports the same resolved identity fields as an applied item

If every member is already current, the whole request succeeds as a no-op and the mesh is unchanged.

If some members are already current and others require application, the request plans one coherent transition for only the needed advancement while returning one outcome for every requested member.

## Retry Semantics

Re-running a successfully completed request with the same admitted bytes, identities, naming inputs, and effective policies succeeds with already-current outcomes rather than creating duplicate states.

This idempotent-success rule applies equally to one-member and multi-member non-overwrite requests. It does not promise automatic recovery from every operational write failure; a write-stage failure may have produced partial observable changes and must disclose that possibility for caller-directed recovery.

## Refusal Families

The operation distinguishes at least these behavioral refusal families:

- invalid request or identity: missing items, malformed fields, normalized duplicates, recursive targets, or multi-item overwrite
- unknown target: an exact designator is not present in the mesh
- wrong target kind: the designator does not resolve to a payload artifact
- malformed mesh: required metadata, inventory, configuration, settled-shape, or progression facts are missing, conflicting, invalid, or unparsable
- inconsistent policy: members do not resolve to one coherent planning policy
- unsupported source: the payload is not backed by an eligible mesh-local working source
- unsupported content: invalid UTF-8, non-text content kind, or otherwise ineligible text/RDF content
- capture conflict: a covered non-authoritative read dependency changes during a bounded capture, if the implementation exposes such a covered set; admitted payload bytes never produce this refusal
- plan conflict: requested naming, existing-state, output identity, or preflight validation conflicts with the proposed transition
- operational write failure: mutation could not be completed and the observable result may be partial

Diagnostic prose is not the portable machine contract. Implementations should expose stable family and phase discriminants and enough target or partial-result detail for callers to respond without matching messages.

## Validation Boundary

The required validation sequence is:

1. request and identity shape, byte admission, and strict UTF-8 validation
2. mesh, inventory, configuration, settled payload shape, source, and content-kind eligibility
3. coherent planning plus validation of every proposed text/RDF output and destination before mutation
4. mutation and result construction

Whole-mesh validation is not part of this operation. A caller may perform broader validation separately.

## Result Invariants

A successful result contains one outcome per normalized request item in canonical designator order, plus request-level created and updated resource locations.

Each outcome reports:

- applied or already current
- canonical designator path
- resolved payload artifact IRI
- selected history segment
- selected state segment
- selected manifestation segment
- historical snapshot location

The result does not expose a mutable implementation plan.

## Non-Goals

This first behavior contract does not specify:

- binary payload recording
- recursive or mixed-kind target batches
- payload-IRI input
- repository-backed, floating, or remote working sources
- cross-writer locking or transactional storage
- command-line syntax
- implementation language types, module names, local path policy, or error classes
- page generation or whole-mesh validation
