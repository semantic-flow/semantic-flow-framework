# Fantasy Rules Branch-Published Conformance

This directory holds Accord manifests for the Fantasy Rules ontology fixture ladder.

The directory name still says `sidecar-fantasy-rules` because the current fixture and manifest files were first authored for a `docs/` sidecar mesh. The intended next conformance shape is branch-published ontology delivery: authored source stays on the normal source branch, while the generated Semantic Flow mesh is carried by a publication branch such as `gh-pages`.

## Current Status

The existing JSON-LD manifests in this directory still describe the older docs-rooted sidecar ladder. They are useful historical input for the fixture-generator work, but they should not be treated as the final branch-published contract.

Do not spend a full fixture rerung on the old docs-rooted ladder immediately before replacing it. The next durable step is to update the manifest/replay model for branch-published operation, then regenerate fixture branches in one intentional pass after the branch-published topology, repository-source locator vocabulary, and near-term config/ontology churn have settled.

`bp-01-source-only.jsonld` is the first branch-published proof manifest. It checks `origin/00-blank-slate` -> `bp-01-source-only` in the existing fixture repository and proves the clean authored source branch shape before any publication branch exists.

## Intended Topology

The Fantasy Rules example should prove the clean-source branch story:

- the normal source branch contains authored ontology, SHACL, example dataset, attribution, and ordinary project files
- the normal source branch does not need `_mesh/`, `.weave/`, `docs/`, generated histories, generated pages, or Weave config
- the publication branch uses its branch root as the mesh root by default
- the publication branch carries `_mesh/`, mesh config, inventories, histories, generated ResourcePages, `.nojekyll`, and any manually managed host files
- the public mesh base stays canonical, such as `https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/`, and does not expose the publication branch name
- local sibling worktree paths never appear in public RDF, generated config, or conformance expectations

The fixture repository may keep its current `mesh-sidecar-fantasy-rules` name for continuity unless a separate fixture-renaming task decides otherwise. The topology, not the repository name, is the contract.

## Source Branch

The source branch should remain a compact ontology project. Its authored files include:

- `ontology/fantasy-rules-ontology.ttl`
- `shacl/fantasy-rules-shacl.ttl`
- `examples/gunaar.ttl`
- `NOTICE.md`

The first ontology seed stays small: `AbilityScore`, `Alignment`, `Character`, and representative controlled values or examples. First-pass ontology term IRIs use slash paths under the ontology namespace, such as `ontology/AbilityScore`, with hash-term coverage deferred.

Authored Turtle uses the `fant:` prefix for `https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/ontology/`. SRD 5.2.1 attribution belongs in `NOTICE.md`. Ontology metadata should carry source/provenance for SRD-derived vocabulary, while `dcterms:license` identifies the fantasy-rules ontology's own license.

## Publication Branch

The publication branch should carry the generated mesh and only intentionally preserved publication controls.

The generated mesh should include:

- `_mesh/_meta/meta.ttl`
- `_mesh/_inventory/inventory.ttl`
- `_mesh/_config/config.ttl`
- generated current and historical ResourcePages
- versioned ontology and SHACL payload histories
- extracted term Knop surfaces and pages
- named release histories such as `ontology/releases/v0.0.1/ttl/fantasy-rules-ontology.ttl`

For a `gh-pages` deployment, the branch root is both the publication source folder and the mesh root unless a later manifest explicitly tests a `/docs` override.

## Source Provenance

Branch-published source bindings must use durable repository-source provenance rather than mesh-root-relative local working paths.

Expected RDF should use the core Semantic Flow source locator shape:

- `sflo:RepositorySourceLocator`
- `sflo:hasTargetRepositorySource`
- `sflo:sourceRepositoryUrl`
- `sflo:sourceRepositoryRef`
- `sflo:sourceRepositoryCommit` when known
- `sflo:sourceRepositoryPath`
- `sflo:hasContentDigest` on the locator or `sflo:expectsContentDigest` on the target relator when deterministic replay needs a byte pin

The branch-published manifests should reject RDF or config that serializes developer-specific paths such as `../mesh-sidecar-fantasy-rules-source/ontology/fantasy-rules-ontology.ttl`. A fixture runner may accept local source root and publication root paths as command inputs, configured publication-root values, or CI checkout layout, but those paths are operational inputs, not public mesh facts.

Raw URLs may appear as access/rendering hints when useful, but URL-first source binding is not the default contract. Repository/ref/path/digest provenance is the durable model because it works for local worktrees, CI checkouts, private repositories, pinned commits, and deterministic replay.

## Branch-Published Ladder Draft

The branch-published ladder should model source and publication refs explicitly. The current one-ref `fromRef` / `toRef` manifest shape is not enough by itself because the source branch and publication branch are separate trees.

The next manifest/replay shape should be able to name:

- source repository and source ref before the operation
- publication repository and publication ref before the operation
- publication ref after the operation
- local source root and publication root as runtime-only execution inputs
- generated files and RDF expectations scoped to the publication branch
- source-branch cleanliness expectations scoped to the source branch
- repository-source locator expectations that bridge the two without leaking local paths

A likely first ladder is:

- seed source branch with authored ontology, SHACL, example dataset, and attribution
- bootstrap the publication branch and create the mesh support surface at branch root
- weave initial support artifacts and current ResourcePages
- bind the ontology payload to its repository-source locator and materialize the first ontology history
- bind the SHACL payload to its repository-source locator and materialize the first SHACL history
- extract selected ontology and SHACL terms into Knop-managed identifier surfaces
- weave extracted term surfaces and generated pages
- add root/example collection Knops if they remain useful for the fixture story
- bind and weave the Gunaar example dataset if example datasets stay in the branch-published fixture
- prepare first-release metadata in the authored sources
- publish named ontology and SHACL release histories using `releases/v0.0.1/ttl` paths
- later, prove explicit return from named release state naming to default ordinal history/state allocation

The first implementation should prove the clean-source branch story before trying to preserve the entire old sidecar ladder. Alice Bio already exercises whole-repo behavior; Fantasy Rules should exercise branch-published ontology delivery.

## Term Extraction Expectations

The first term-extraction pair should keep the extracted term set narrow and explicit, starting with class-like terms such as:

- `ontology/AbilityScore`
- `ontology/Alignment`
- `ontology/Character`
- `ontology/CharacterShape`

`ontology/CharacterShape` stays under the ontology namespace even when its source facts come from the SHACL source artifact. Extracted term page facts should follow the term Knop inventory's source binding. They should not infer the source artifact from the term path prefix.

## Release Expectations

First release/weave transitions use custom ArtifactHistory segment `releases`, HistoricalState segment `v0.0.1`, and Turtle ArtifactManifestation segment `ttl`.

First versioned located files should be:

- `ontology/releases/v0.0.1/ttl/fantasy-rules-ontology.ttl`
- `shacl/releases/v0.0.1/ttl/fantasy-rules-shacl.ttl`

Because the publication branch root is the mesh root, these paths do not carry a `docs/` prefix in the branch-published shape.

## Manifest Authoring Notes

Conventions used here:

- one Accord `Manifest` per transition
- one `TransitionCase` per manifest until a branch-published replay shape requires separate source/publication cases
- fixture paths in publication expectations are publication-branch-root-relative
- source cleanliness expectations are source-branch-root-relative
- whole-tree transition completeness checks should be enabled where practical, with `ignorePaths` used only for intentional non-contract paths
- manifests should not both ignore and explicitly expect the same path
- RDF-bearing files use `rdfCanonical`
- generated Resource Page HTML expectations should omit `compareMode`; their `changeType` and `path` are presence/absence contracts, not exact HTML content contracts
- non-text control files such as `.nojekyll` use `bytes`

These manifests should be authored before any generated fixture branch is allowed to define the behavior. The intended acceptance loop is manifest-first: write or update the transition expectation, validate the manifest itself, run the fixture generator/replay path, inspect generated branch diffs, then run the relevant fixture tests.
