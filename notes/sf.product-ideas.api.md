---
id: 9x9jpzawjmxdxvpaef3bhwn
title: API
desc: ''
updated: 1783403928019
created: 1783403928019
---

Speculative API surfaces that are not yet committed specs. Each idea should record its motivation, how it fits the existing model, and why it is deferred, so a later reader can pick it up without re-deriving the analysis. Graduating an idea means writing an `sf.spec.*` note and wiring it into [[sf.api]].

## Direct-content version (bytes + IRI + optional metadata)

Let a caller submit artifact bytes plus a target `DigitalArtifact` IRI/designator and optional metadata, and have Weave perform a `version` — append a new `HistoricalState` — directly from the supplied bytes, without the caller first writing a working file into the mesh.

Rough shape:

- target `DigitalArtifact` IRI or designator (required)
- content bytes as a dataset/quad stream/string, plus input format when the source is textual (required)
- optional caller-supplied `HistoricalState` IRI and `ArtifactHistory` IRI/name, so the caller owns state identity when it wants correspondence
- optional support metadata, capture/provenance links, and reference data to attach to the state
- requested on-disk serialization format
- returns a plan/result: created state, manifestation, located file, digest, and mesh/Knop inventory updates

### How it fits the existing model

This is a thin content-source-plus-identity front door onto the same `version` planner, not a new engine. Three facts from the core ontology make it API-only rather than an ontology change:

- `hasWorkingLocatedFile` is already optional and explicitly non-identity, so supplying bytes directly is just another content source, not a fake working file.
- `stateOrdinal` / `nextStateOrdinal` are already documented as auto-issued defaults for numbered `_sNNNN` states. Ordinals stay per-serialization and Weave-assigned; the caller's optional state IRI is additive identity, not a competing ordinal stream.
- File extensions belong to the realized `LocatedFile`, not to the supplied bytes or the in-memory graph identity. `previousHistoricalState` already exists for ordering, so caller-named states need no ordinal to be sequenced.

File-based `version` and the multi-target payload batch remain unchanged; this is an alternate content source over the same job, and would ideally be implemented as a lower-level "realize artifact state" planner that file-based `version` also calls.

### Why it is deferred

Not needed while consumers keep a filesystem-backed mesh and write working files at save-point time; in that flow each save is one new working file per artifact, which the existing file-based `version` and the multi-target batch already handle. Passing bytes directly mainly saves the working-file round-trip.

It also opens the Weave-as-a-library packaging surface, including package-relative resource loading, which carries real cost — the Accord `v0.1.0` JSR release shipped a shapes-loading bug of exactly this class (`import.meta.url` resolving to a non-`file:` URL off the local filesystem). Defer until an application genuinely wants to version without a disk write.

### Related

- A companion idea from the same analysis: record capture correspondence ("this state captures checkpoint X") as an application-authored RdfDocument payload `DigitalArtifact` — a checkpoint/flush manifest naming the changed states — rather than as a new core predicate or as custom inventory statements. Weave realizes it as opaque payload and gains no domain vocabulary. This is the grouped-provenance/citable-batch shape, worth doing independently of this API.
- Prior analysis: [[sf.report.2026-07-06-in-memory-support-codex]] and [[sf.report.2026-07-06-in-memory-support-chatgp]]. Both are useful on the conceptual split but frame the state ordinal as identity; the committed direction here treats the ordinal as a per-serialization default and the caller-supplied name as identity.
