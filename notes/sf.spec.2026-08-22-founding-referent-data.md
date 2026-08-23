---
id: sffoundingreferentdata20260822
title: 2026 08 22 Founding Referent Data Behavior
desc: 'Portable creation, settlement, correction, and validation contract for bounded founding referent facts'
created: 1787439044000
---

## Purpose

This note specifies the portable observable behavior for optionally carrying a bounded RDF record about a newly initialized Semantic Flow identifier's referent and settling or correcting that record through ordinary immutable artifact states.

It specifies outcomes and invariants. Filesystem paths and CLI spellings below describe the first hierarchy-backed Weave surface; the semantic contract does not require another implementation to use the same host layout.

## Artifact And Discovery Contract

A non-root Knop may own at most one `FoundingReferentData` artifact through `hasFoundingReferentData`.

The artifact:

- is a `DigitalArtifact` and `RdfDocument`
- is not a `SemanticFlowResource` in this slice and has no `ResourcePage`
- is distinct from `KnopMetadata`, a primary `PayloadArtifact`, a `ReferenceCatalog`, and source provenance
- is conventionally serialized at `D/_knop/_founding`, with working Turtle at `D/_knop/_founding/data.ttl`
- is discoverable locally through MeshInventory → KnopInventory → `hasFoundingReferentData` → `hasWorkingLocatedFile`

The working `LocatedFile` is an `RdfDocument`. Mutable working bytes carry no standing content digest. Digests describe immutable historical manifestations and snapshot files.

## Initialization Input

`knop.create` accepts optional founding Turtle bytes in addition to its existing required inputs. Omitted founding input preserves the existing no-founding transition and output.

The hierarchy-backed CLI surface is:

```text
weave knop create <D> --founding-data <path>
```

The supplied path is an explicitly requested local source. It resolves from the command working directory, must satisfy active local-path policy, and must not identify the conventional target itself. Implementations admission-copy the bytes before planning.

Initialization validates the complete request before mutation and then creates the ordinary Knop support files, exact founding working bytes, and settled discovery/type facts as one operation. A pre-existing metadata, inventory, or founding target refuses; adoption and silent overwrite are not supported.

Initialization creates no payload, reference catalog, reference link, source registry, history, historical state, manifestation, snapshot, or page. It performs no network access, source resolution, generation, weaving, versioning, or publication.

## Founding Document Profile

The first profile is a non-empty Turtle document of no more than 64 KiB and 256 triples.

For public referent IRI `D`:

- parsing injects no base IRI, and a lexer-level `@base` directive is forbidden
- every triple is in the default graph and has exactly the normalized absolute lexical IRI `D` as subject
- every named node is an absolute IRI after prefix expansion
- predicates are named IRIs; objects are named IRIs or literals
- blank nodes are forbidden in every term position
- named graphs, RDF-star quoted triples, and generalized RDF are outside the profile
- every SFLO or SFCFG predicate is forbidden
- an `rdf:type` object in the SFLO or SFCFG namespace is forbidden
- the root designator is refused when founding input is supplied

Equivalent but differently spelled subjects, `D#fragment`, relative IRIs, malformed or non-UTF-8 input, empty/comment-only input, and excess size or triple count refuse. Language-tagged and typed literals, absolute IRI objects, downstream `rdf:type` objects, and downstream predicates such as Stagecraft's `incarnationOf` are allowed.

Parser and profile diagnostics are fixed and content-free. Operational and audit logs must not include input source lines, literal values, parsed terms, or parser messages.

## Settlement And Correction

Before a press containing founding working data may land, those exact working bytes must be settled into an initial immutable HistoricalState without page generation.

The hierarchy-backed CLI surface is:

```text
weave version <D> --artifact-role founding-referent-data [--source <path>]
```

The programmatic surface is `versionFoundingReferentData({ meshRoot, designatorPath, bytes? })`. `versionPayloads` remains payload-only.

With omitted source/bytes, the operation versions the current founding working bytes. With supplied source/bytes, it admission-copies and validates the corrected bytes, plans the working replacement and next historical state from one in-memory overlay, and writes them as one composed operation. The supplied source follows the same local-source policy and collision rules as initialization.

The first state uses the ordinary chain:

`FoundingReferentData → ArtifactHistory → HistoricalState → ArtifactManifestation → LocatedFile`

The manifestation and immutable snapshot file carry the canonical SHA-256 digest of the exact snapshot bytes. The snapshot bytes are byte-identical to the admitted working bytes, including a UTF-8 BOM or CRLF line endings.

A later correction:

- updates the mutable working file and appends a later HistoricalState
- advances the history's latest/next progression
- preserves every earlier state and snapshot byte-for-byte
- creates no founding-data page
- lands through a later press; it never rewrites an already committed or published state

Reset-and-replay is a repair only before a press lands. After commit/publication, correction requires a later state and a new press.

## Validation

Ordinary authoring validation treats working bytes that differ from the latest settled snapshot as pending work, not corruption.

Publication or press validation reports `unsettled-founding-referent-data` when a registered founding working file has no matching latest settled snapshot. This applies before initial settlement and after a working correction that has not yet been settled.

Validation reports the existing `content-digest-mismatch` finding when immutable founding snapshot bytes disagree with their declared digest. It does not apply that finding to mutable working bytes. Whole-mesh validation checks every registered founding artifact; exact target validation checks only selected designators. Validation performs no network access.

## Preservation

Every ordinary operation that reconstructs or advances a KnopInventory preserves the complete founding support subgraph: owner slot, artifact and working-file facts, histories, states, manifestations, snapshot files, digest claims, and unknown settled facts rooted under the founding artifact.

In particular, ordinary `weave` and `knop add-reference` must not orphan founding data or generate a founding-data page.

## Failure Atomicity

All founding inputs are admitted and validated before writes. A planning or preflight refusal leaves the mesh unchanged.

The first filesystem-backed implementation provides in-process rollback for create and for composed correction: it removes only paths created by the failing operation, restores prior updated bytes, and reports rollback failure separately using safe paths. Cross-file crash atomicity is not claimed; validation detects an interrupted surface.

## Acceptance Transitions

The carried Accord sequence is:

1. `founding-created`: exact working bytes and discovery facts exist; history, pages, payloads, references, and sources do not.
2. `founding-versioned`: immutable state 1 and its digest exist; working bytes are unchanged and no founding page exists.
3. `founding-corrected`: corrected working bytes and state 2 exist; state 1 is byte-identical to the prior transition, latest/current selection advances, and no founding page exists.

## Non-Goals

- generic `ReferentMetadata` or arbitrary assertions in `KnopMetadata`
- Stagecraft vocabulary in SFLO
- root founding data
- blank-node or structured founding graphs
- adoption, retraction, remote fetching, or a standalone founding update command
- payload inference, page generation, or founding facts as identifier-page input
- batch Knop initialization
