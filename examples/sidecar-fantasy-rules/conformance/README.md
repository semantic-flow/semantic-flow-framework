# Fantasy Rules Sidecar Conformance

This directory will contain Accord manifests for the `mesh-sidecar-fantasy-rules` fixture ladder.

Each file describes one transition between named fixture refs in the separate `github.com/semantic-flow/mesh-sidecar-fantasy-rules` repository. The first ladder is expected to be linear, so manifest files are named after the destination branch even though the manifest unit is still a transition.

Planned first naming:

- `01-source-only.jsonld` means `00-blank-slate` -> `01-source-only`
- `02-sidecar-mesh-created.jsonld` means `01-source-only` -> `02-sidecar-mesh-created`
- `03-sidecar-mesh-created-woven.jsonld` means `02-sidecar-mesh-created` -> `03-sidecar-mesh-created-woven`
- `04-ontology-integrated.jsonld` means `03-sidecar-mesh-created-woven` -> `04-ontology-integrated`
- `05-ontology-integrated-woven.jsonld` means `04-ontology-integrated` -> `05-ontology-integrated-woven`
- `06-shacl-integrated.jsonld` means `05-ontology-integrated-woven` -> `06-shacl-integrated`
- `07-shacl-integrated-woven.jsonld` means `06-shacl-integrated` -> `07-shacl-integrated-woven`
- `08-ontology-and-shacl-terms-extracted.jsonld` means `07-shacl-integrated-woven` -> `08-ontology-and-shacl-terms-extracted`
- `09-ontology-and-shacl-terms-extracted-woven.jsonld` means `08-ontology-and-shacl-terms-extracted` -> `09-ontology-and-shacl-terms-extracted-woven`
- `10-root-knop.jsonld` means `09-ontology-and-shacl-terms-extracted-woven` -> `10-root-knop`
- `11-gunaar-example-dataset.jsonld` means `10-root-knop` -> `11-gunaar-example-dataset`
- `12-gunaar-example-dataset-woven.jsonld` means `11-gunaar-example-dataset` -> `12-gunaar-example-dataset-woven`
- `13-first-release.jsonld` means `12-gunaar-example-dataset-woven` -> `13-first-release`
- `14-first-release-woven.jsonld` means `13-first-release` -> `14-first-release-woven`

How to read the ladder:

- destination branches are human-reviewable expected states
- manifests are transition contracts, not branch metadata
- most steps come in operation/weave pairs
- the first branch in a pair introduces or edits the current working surface without yet materializing new histories or public pages
- the second branch in a pair is usually the `weave` step that versions those changes and regenerates current public HTML
- tests can check out the source branch, run the intended operation or `weave` in a temporary workspace, and compare the result to the destination branch under the corresponding manifest

Ladder walkthrough:

- `00 -> 01`: fixture seed only. This is not a public Semantic Flow operation. It should introduce the small authored ontology and SHACL source tree, the SRD attribution boundary in `NOTICE.md`, and no docs-rooted mesh yet.
- `01 -> 02 -> 03`: create the docs-rooted sidecar mesh, then weave it so `docs/_mesh` support artifacts and initial generated pages exist.
- `03 -> 04 -> 05`: integrate the governed ontology artifact at public path `ontology`, using a policy-approved adjacent `workingLocalRelativePath` such as `../ontology/fantasy-rules-ontology.ttl`, then weave it into public sidecar history and pages.
- `05 -> 06 -> 07`: integrate the governed SHACL artifact at public path `shacl`, using a policy-approved adjacent `workingLocalRelativePath` such as `../shacl/fantasy-rules-shacl.ttl`, then weave it into public sidecar history and pages.
- `07 -> 08 -> 09`: extract selected slash-IRI ontology and SHACL terms from the woven ontology and SHACL documents into Knop-managed identifier surfaces, then weave those surfaces so the terms have public histories and dereferenceable pages. This includes `ontology/CharacterShape`, which stays under the ontology namespace even though its pinned source facts come from the `shacl` artifact.
- `09 -> 10`: add a friendly root Knop for the repository Resource Page and an `examples/` Knop to act as the example-dataset collection surface.
- `10 -> 11 -> 12`: integrate the Gunaar example dataset at public path `examples/gunaar`, using a policy-approved adjacent `workingLocalRelativePath` such as `../examples/gunaar.ttl`, then weave it into public sidecar history and pages.
- `12 -> 13 -> 14`: publish the first named ontology and SHACL release histories using `releases/v0.0.1/ttl` paths, while keeping prior ordinal histories available as earlier publication history.

Source-shape conventions for the first ladder:

- the public mesh root is `docs/`
- mesh-owned helper page content lives under `docs/_mesh/content/`
- authored ontology source lives under `ontology/`
- authored SHACL source lives under `shacl/`
- authored example datasets live under `examples/`
- first-pass ontology term IRIs use slash paths under the ontology namespace, such as `ontology/AbilityScore`, with hash-term coverage deferred
- the first term-extraction pair should keep the extracted term set narrow and explicit, starting with class-like terms such as `ontology/AbilityScore`, `ontology/Alignment`, and `ontology/Character`, plus a narrow SHACL shape term such as `ontology/CharacterShape` if the authored SHACL graph already carries that public IRI
- authored Turtle uses the `fant:` prefix for `https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/ontology/`
- extracted term page facts should follow the term `ReferenceCatalog` `referenceTarget` and `referenceTargetState`; they should not infer the source artifact from the term path prefix
- first release/weave transitions use custom ArtifactHistory segment `releases`, HistoricalState segment `v0.0.1`, and Turtle ArtifactManifestation segment `ttl`
- first versioned located files should be `docs/ontology/releases/v0.0.1/ttl/fantasy-rules-ontology.ttl` and `docs/shacl/releases/v0.0.1/ttl/fantasy-rules-shacl.ttl`
- example dataset integration should add the constrained `../examples/` local path grant only when the Gunaar dataset artifact is integrated
- the first ontology seed stays small: `AbilityScore`, `Alignment`, `Character`, and representative controlled values or examples
- SRD 5.2.1 attribution belongs in the fixture repository `NOTICE.md`
- the SRD 5.2 Markdown transcription at `https://github.com/springbov/dndsrd5.2_markdown/blob/main/DND-SRD-5.2-CC.md` may be used as a convenience source for review and extraction
- the official SRD source and attribution statement remain authoritative
- ontology metadata should carry source/provenance for SRD-derived vocabulary, while `dcterms:license` identifies the fantasy-rules ontology's own license

Conventions used here:

- one Accord `Manifest` per file
- one `TransitionCase` per manifest for now
- fixture paths in expectations are repository-root-relative, so sidecar mesh paths include the `docs/` prefix
- RDF-bearing files use `rdfCanonical`
- generated Resource Page HTML expectations should omit `compareMode`; their `changeType` and `path` are presence/absence contracts, not exact HTML content contracts
- non-text control files such as `.nojekyll` use `bytes`

These manifests should be authored before any dedicated runner or generated fixture path is allowed to define the behavior. The intended acceptance loop is manifest-first: write the transition expectation, validate the manifest itself, then run any implementation or pseudo-runner against it.
