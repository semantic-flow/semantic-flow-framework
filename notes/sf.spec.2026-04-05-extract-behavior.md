---
id: 7q4nc3p3m9x1v0y2j5t8l6kr
title: 2026 04 05 Extract Behavior
desc: ''
updated: 1775487600000
created: 1775487600000
---

## Purpose

This note captures the current expected behavior of the first `extract` slice for Semantic Flow.

It is a current Semantic Flow behavior spec for the first carried path, not the complete thin public API contract.

## Status

This is the current next carried slice after the completed local `10-alice-bio-updated` -> `11-alice-bio-v2-woven` work.

The first acceptance target is the settled `mesh-alice-bio` transition from `11-alice-bio-v2-woven` to `12-bob-extracted`.

The next carried extension is the `mesh-sidecar-fantasy-rules` transition from `07-shacl-integrated-woven` to `08-ontology-and-shacl-terms-extracted`. That extension keeps the same non-woven extraction boundary while proving docs-rooted sidecar operation, multiple governed payload artifacts, repeated term extraction, and explicit source-payload selection for terms mentioned by more than one woven RDF document.

This note is intentionally narrower than a generic RDF graph-refactoring or payload-splitting design. If future fixture work wants broader extraction behavior, that should be specified explicitly rather than inferred from this slice.

## Inputs

- the machine-facing job kind and manifest `operationId` stay `extract`
- the first local CLI surface should be `weave extract <designatorPath>`
- `designatorPath` is required and identifies the new local Semantic Flow identifier to extract into a minimal Knop-managed surface
- the first local shared `core` request should stay narrow and target-oriented: target `designatorPath`, resolved `referenceTargetDesignatorPath`, resolved `referenceTargetStatePath`, current mesh inventory, and the current woven source payload state needed to justify the extraction
- the Alice Bio slice does not expose a separate source-designator selector because only one eligible woven payload artifact mentions `bob`
- the Fantasy Rules sidecar slice may provide an explicit source designator, such as `--source-designator-path ontology` or `--source-designator-path shacl`, when the target term is mentioned by multiple eligible woven payload artifacts
- the target workspace must already contain `_mesh/_meta/meta.ttl`, `_mesh/_inventory/inventory.ttl`, and at least one already woven payload artifact with an explicit latest historical state
- in the carried `12` acceptance target, the runtime resolves `bob` from the current working `alice/bio` payload surface and pins the created Bob reference to `alice/bio/_history001/_s0002`
- if zero eligible woven payload artifacts mention the target designator, or more than one eligible woven payload artifact mentions it, the first local slice should fail closed rather than guessing
- when an explicit source designator is provided, that source must identify one eligible woven payload artifact and that payload must mention the target designator; otherwise extraction fails closed

## What Extract Does

`extract` creates the first minimal Knop-managed current surface for a locally referenced resource that is still described inside an existing payload artifact.

In the current first slice, that means:

- updating `_mesh/_inventory/inventory.ttl` so the mesh registers `<D/_knop>` as a current `Knop` with `hasWorkingKnopInventoryFile <D/_knop/_inventory/inventory.ttl>`
- creating `D/_knop/_meta/meta.ttl`
- creating `D/_knop/_inventory/inventory.ttl`
- creating `D/_knop/_references/references.ttl`
- creating one stable `ReferenceLink` rooted at `<D/_knop/_references#reference001>`
- recording that link as about `<D>` with `referenceLinkFor <D>`, `hasReferenceRole <.../ReferenceRole/Supplemental>`, `referenceTarget <T>`, and `referenceTargetState <S>`

For sidecar term extraction, the mesh inventory update should preserve the existing multi-payload mesh inventory and append the new term Knop facts instead of reconstructing a single-payload inventory shape. The created term support artifacts still use the same minimal KnopMetadata, KnopInventory, and ReferenceCatalog files. Extraction does not add `hasResourcePage` for the term in the non-woven branch; generated pages belong to the following weave step.

For the carried Bob extraction target:

- `D` is `bob`
- `T` is `alice/bio`
- `S` is `alice/bio/_history001/_s0002`

## What Extract Does Not Do

In this first slice, `extract` does not:

- split `bob` into a new payload artifact
- rewrite `alice-bio.ttl`
- update `alice/_knop/_inventory/inventory.ttl`
- update `alice/_knop/_references/references.ttl`
- update `alice/bio/_knop/_inventory/inventory.ttl`
- create Bob histories under `bob/_knop/_meta/_history001`, `bob/_knop/_inventory/_history001`, or `bob/_knop/_references/_history001`
- create `bob/index.html`, `bob/_knop/index.html`, `bob/_knop/_meta/index.html`, `bob/_knop/_inventory/index.html`, or `bob/_knop/_references/index.html`
- run `weave`, `version`, `validate`, or `generate` as separate historical-materialization steps
- expose a broad source-selection or graph-surgery API
- introduce daemon behavior

## Invariants

- the created `ReferenceLink` identity should be a stable fragment IRI rooted at the Bob `ReferenceCatalog`, not at a historical state
- `referenceLinkFor` points to `<bob>`, not to `<bob/_knop>`
- the carried first slice should use `ReferenceRole/Supplemental`
- `referenceTargetState` should point to the latest woven historical state of the source payload artifact, not to the source Knop or to the working payload file
- `alice-bio.ttl` must remain unchanged
- existing Alice support artifacts and page files must remain unchanged
- Bob should gain no `hasPayloadArtifact` relationship in this slice
- extracted ontology and SHACL terms should gain no `hasPayloadArtifact` relationship in the non-woven extraction slice
- if `bob/_knop/_meta/meta.ttl`, `bob/_knop/_inventory/inventory.ttl`, or `bob/_knop/_references/references.ttl` already exist, the operation should fail closed rather than silently overwrite them
- if any extracted term's `_knop/_meta/meta.ttl`, `_knop/_inventory/inventory.ttl`, or `_knop/_references/references.ttl` already exists, the operation should fail closed rather than silently overwrite it
- runtime-local `.weave/logs` output is not part of the semantic mesh surface

## Relationship To Neighboring Specs

This note is adjacent to, but not replaced by:

- [[sf.spec.2026-04-03-weave-behavior]]
- [[sf.spec.2026-04-04-knop-add-reference-behavior]]

The current extract slice is best understood as:

- `knop.create`-like creation of a new minimal Knop support surface
- plus `knop.addReference`-like creation of a `ReferenceCatalog`
- but with extraction-specific source resolution and an explicit `referenceTargetState`
- and without weave/history/page generation

## Acceptance Reference

The first behavior-level comparison target is:

- fixture repo: `dependencies/github.com/semantic-flow/mesh-alice-bio`
- from ref: `11-alice-bio-v2-woven`
- to ref: `12-bob-extracted`
- manifest: `dependencies/github.com/semantic-flow/semantic-flow-framework/examples/alice-bio/conformance/12-bob-extracted.jsonld`
- local CLI execution should match that manifest-scoped result while ignoring unrelated fixture `README.md` churn

The sidecar term-extraction behavior-level comparison target is:

- fixture repo: `dependencies/github.com/semantic-flow/mesh-sidecar-fantasy-rules`
- from ref: `07-shacl-integrated-woven`
- to ref: `08-ontology-and-shacl-terms-extracted`
- manifest: `dependencies/github.com/semantic-flow/semantic-flow-framework/examples/sidecar-fantasy-rules/conformance/08-ontology-and-shacl-terms-extracted.jsonld`
- local CLI execution should run from the fixture workspace root with `--mesh-root docs`, extracting the selected term set with explicit `--source-designator-path` values where needed
