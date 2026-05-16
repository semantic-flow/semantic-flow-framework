---
id: 2c93t4j3o8m9k7p5v1n6x0q
title: API Examples
desc: ''
updated: 1774416095095
created: 1774339800000
---

This note groups worked Semantic Flow API example payloads that are kept outside the OpenAPI file so the normative contract can stay lean.

## API Example Development Notes

Purpose: keep the example ladders usable as conformance fixtures for Semantic Flow implementations, not just as snapshots of the latest repository state.

When ontology terms, JSON-LD shapes, or expected API payload fields change, update every affected branch in the ladder. Do not rely on test-helper normalization to translate old fixture vocabulary at runtime; each branch should remain independently valid as an example of the API and ontology vocabulary it currently claims to use.

Preferred strategy:

- Start with the earliest affected branch, because later branches usually carry forward RDF, manifests, page definitions, or generated files from earlier states.
- Commit each branch separately with the same focused change description when the edit is mechanical across the ladder.
- After updating a branch, advance to the next branch from its existing branch tip rather than regenerating the whole ladder from the final state.
- Re-run the conformance or Weave fixture tests after the ladder has been updated, and treat failures as evidence that some carried-forward state was missed.

Each conformance transition should also record the command or invocation shape used to climb that rung. The Accord manifest remains the durable semantic contract: operation id, source and destination refs, target designators, source designators, version segment choices, file expectations, and RDF expectations are still what tests should validate. The command transcript is a reproducibility aid for humans and implementors, not a replacement for those semantic fields.

For new rungs, record the invocation before settling the branch, preferably in the transition manifest and secondarily in the conformance README when the manifest vocabulary has not caught up yet. The record should include:

- the working directory, such as the fixture repository root
- the executable and subcommand, such as `weave integrate`, `weave extract`, or the root `weave` operation
- every non-default flag and positional argument, including `--mesh-root`, target selectors, source selectors, grant paths, history/state/manifestation segments, and confirmation flags
- any deliberate batch semantics, such as whether one command applied one shared state segment to several selected DigitalArtifacts
- a clear label when an older rung only has a reconstructed reproduction command rather than the exact command that was originally run

Do not let the transcript become the normative behavior definition. If the CLI spelling changes later, old transcripts may remain historically useful while the manifest contract and spec notes describe the portable operation semantics.

## Alice Bio

The current primary worked example set lives in `../examples/alice-bio/api`.

Those files are anchored to the existing ontology use case in [[ont.use-case.biographical-data-publishing]].

Current files:

- `mesh-create-job-request.jsonld` : the request payload for submitting `mesh.create`
- `mesh-create-job-accepted.jsonld` : the accepted `Job` resource returned immediately after `mesh.create` submission
- `mesh-create-job-succeeded.jsonld` : the terminal successful `mesh.create` `Job` resource with created mesh-surface resources
- `knop-create-job-request.jsonld` : the request payload for submitting `knop.create`
- `knop-create-job-succeeded.jsonld` : the terminal successful `knop.create` `Job` resource with created Knop resources and the updated mesh inventory
- `integrate-job-request.jsonld` : the request payload for submitting `integrate` with one `designatorPath` and one semantic `sourceUri`
- `integrate-job-succeeded.jsonld` : the terminal successful `integrate` `Job` resource with the created payload and Knop resources plus the updated mesh inventory
- `weave-job-request.jsonld` : the request payload for submitting `weave` over the Alice mesh with a narrowed `designatorPaths` target
- `weave-job-succeeded.jsonld` : the terminal successful `weave` `Job` resource with created histories and updated current artifacts
- `mesh.jsonld` : a `SemanticMesh` representation with Hydra affordances for follow-up actions
- `knop.jsonld` : a `Knop` representation with Hydra affordances for follow-up actions
- `knop-add-reference-job-request.jsonld` : the request payload for submitting `knop.addReference`
- `job-accepted.jsonld` : the accepted `Job` resource returned immediately after submission
- `job-succeeded.jsonld` : the terminal successful `Job` resource with result links
- `reference-link.jsonld` : the created `ReferenceLink` resource

Current local-convenience files:

- `payload-update-job-request.jsonld` : the request payload for submitting `payload.update` as a current-working-surface replacement convenience
- `payload-update-job-succeeded.jsonld` : the terminal successful `payload.update` `Job` resource with the updated current payload surface

Next likely additions:

- `extract-job-request.jsonld` : a request payload for extracting identifiers and Knops from terms mentioned in an existing ingested `RdfDocument`, likely starting with an ontology-term extraction example
- `extract-job-succeeded.jsonld` : the terminal successful `extract` `Job` resource with the created identifier and Knop surfaces
- `import-job-request.jsonld` : a request payload for importing outside-the-tree or extra-mesh content into a governed in-tree artifact boundary
- `import-job-succeeded.jsonld` : the terminal successful `import` `Job` resource with the imported artifact and current working file

Those are planned example names, not files that already exist in `../examples/alice-bio/api`.

This is meant to be a realistic vertical slice rather than an exhaustive example catalog.

## Fantasy Rules Branch-Published Ontology

The Fantasy Rules example family is split across two fixture topologies:

- `../examples/sidecar-fantasy-rules/conformance/` covers the docs-rooted sidecar mesh carried by `mesh-sidecar-fantasy-rules`.
- `../examples/branch-fantasy-rules/conformance/` covers the branch-published mesh carried by `mesh-branch-fantasy-rules`.

Both sets are anchored to [[ont.use-case.dereferenceable-ontology]] and the Weave planning notes [[wd.task.2026.2026-05-02-fantasy-rules-sidecar]], [[wd.task.2026.2026-05-13_1655-support-gh-pages-branch-based-deployments]], and [[wd.task.2026.2026-05-15_1113-mesh-branch-fantasy-rules]]. They follow the Alice Bio split where it applies:

- `api/` for request and response payload examples when a public API slice needs an example
- `conformance/` for Accord transition manifests

The sidecar fixture intentionally remains a `docs/` mesh. The branch-published fixture instead keeps authored ontology, SHACL, example, release, attribution, and deterministic replay assets on a clean source branch while public identifiers, generated pages, mesh config, inventories, histories, release bytes, source registries, and references live on publication refs such as `gh-pages`.

The conformance approach follows Alice Bio where it still fits: one manifest per transition, named after the destination branch only as a convenience. Branch-published manifests distinguish source refs from publication refs because the source branch and generated mesh branch are different trees. The branch fixture currently uses numeric `a.` refs, with `main` kept close to `a.01-source-only`, publication rungs such as `a.15-extracted-term-references-woven`, and `gh-pages` fast-forwarded to the accepted publication state.

Current API/example pressure points:

- publication-branch bootstrap with source and publication roots supplied as runtime inputs
- source bindings that use repository/ref/path/digest provenance rather than host-local sibling paths
- `_knop/_sources` source registries for repository materialization bindings and extraction provenance
- a deploy orchestration path that materializes authored ontology, SHACL, and example files into a publication-branch mesh
- `weave` or `version` using custom versioning segments such as ArtifactHistory `releases`, HistoricalState `v0.0.1`, and ArtifactManifestation `ttl`
- `weave` producing artifact-local release located files such as `ontology/releases/v0.0.1/ttl/fantasy-rules-ontology.ttl`
- extracted term Knops whose source provenance can track the current source artifact while curated `ReferenceLink`s remain separate
- `RdfDocument` resource pages that expose raw RDF bytes when those bytes are locally available
- `owl:versionIRI` pointing at versioned `LocatedFile` bytes rather than the abstract historical state
