# Alice Bio Conformance

This directory contains Accord manifests for the `mesh-alice-bio` fixture ladder.

Each file describes one transition between named fixture refs in the separate `github.com/semantic-flow/mesh-alice-bio` repository. The current ladder is linear, so the files are named after the destination branch even though the manifest unit is still a transition.

Current naming:

- `01-source-only.jsonld` means `00-blank-slate` -> `01-source-only`
- `02-mesh-created.jsonld` means `01-source-only` -> `02-mesh-created`
- `03-mesh-created-woven.jsonld` means `02-mesh-created` -> `03-mesh-created-woven`
- `04-alice-knop-created.jsonld` means `03-mesh-created-woven` -> `04-alice-knop-created`
- `05-alice-knop-created-woven.jsonld` means `04-alice-knop-created` -> `05-alice-knop-created-woven`
- `06-alice-bio-integrated.jsonld` means `05-alice-knop-created-woven` -> `06-alice-bio-integrated`
- `07-alice-bio-integrated-woven.jsonld` means `06-alice-bio-integrated` -> `07-alice-bio-integrated-woven`
- `08-alice-bio-referenced.jsonld` means `07-alice-bio-integrated-woven` -> `08-alice-bio-referenced`
- `09-alice-bio-referenced-woven.jsonld` means `08-alice-bio-referenced` -> `09-alice-bio-referenced-woven`
- `10-alice-bio-updated.jsonld` means `09-alice-bio-referenced-woven` -> `10-alice-bio-updated`
- `11-alice-bio-v2-woven.jsonld` means `10-alice-bio-updated` -> `11-alice-bio-v2-woven`
- `12-bob-extracted.jsonld` means `11-alice-bio-v2-woven` -> `12-bob-extracted`
- `13-bob-extracted-woven.jsonld` means `12-bob-extracted` -> `13-bob-extracted-woven`
- `14-alice-page-customized.jsonld` means `13-bob-extracted-woven` -> `14-alice-page-customized`
- `15-alice-page-customized-woven.jsonld` means `14-alice-page-customized` -> `15-alice-page-customized-woven`
- `16-alice-page-main-integrated.jsonld` means `15-alice-page-customized-woven` -> `16-alice-page-main-integrated`
- `17-alice-page-main-integrated-woven.jsonld` means `16-alice-page-main-integrated` -> `17-alice-page-main-integrated-woven`
- `18-alice-page-artifact-source.jsonld` means `17-alice-page-main-integrated-woven` -> `18-alice-page-artifact-source`
- `19-alice-page-artifact-source-woven.jsonld` means `18-alice-page-artifact-source` -> `19-alice-page-artifact-source-woven`
- `20-bob-page-imported-source.jsonld` means `19-alice-page-artifact-source-woven` -> `20-bob-page-imported-source`
- `21-bob-page-imported-source-woven.jsonld` means `20-bob-page-imported-source` -> `21-bob-page-imported-source-woven`
- `22-root-knop-created.jsonld` means `21-bob-page-imported-source-woven` -> `22-root-knop-created`
- `23-root-knop-created-woven.jsonld` means `22-root-knop-created` -> `23-root-knop-created-woven`
- `24-root-page-customized.jsonld` means `23-root-knop-created-woven` -> `24-root-page-customized`
- `25-root-page-customized-woven.jsonld` means `24-root-page-customized` -> `25-root-page-customized-woven`

How to read the ladder:

- most steps come in pairs
- the first branch in a pair introduces or edits the current working surface without yet materializing new histories or public pages
- the second branch in a pair is usually the `weave` step that versions those changes and regenerates current public HTML
- the manifests are transition-shaped even when the branch names make the ladder look branch-shaped

Ladder walkthrough:

- `00 -> 01`: fixture seed only. This is not a public Semantic Flow operation.
- `01 -> 02 -> 03`: create the mesh, then weave it so `_mesh/_inventory` and `_mesh/index.html` exist.
- `03 -> 04 -> 05`: create Alice's Knop, then weave it so Alice gets Knop support-artifact histories and `alice/index.html`.
- `05 -> 06 -> 07`: integrate the governed payload artifact `alice/bio`, then weave it into explicit artifact history and updated public pages.
- `07 -> 08 -> 09`: add Alice reference-catalog support artifacts, then weave them.
- `09 -> 10 -> 11`: update Alice bio content, then weave the new current artifact state and regenerated pages.
- `11 -> 12 -> 13`: extract Bob from existing RDF content, then weave Bob's first identifier page and support surfaces.
- `13 -> 14 -> 15`: add Alice's knop-owned page definition at `alice/_knop/_page/page.ttl`, then weave it so `alice/index.html` follows customized Markdown sources instead of the generic page path.
- `15 -> 16 -> 17`: introduce a separate governed Markdown artifact `alice/page-main`, then weave that artifact. Alice's page still points at the old direct file source at this stage.
- `17 -> 18 -> 19`: repoint Alice's page definition so the main region follows governed artifact `alice/page-main`, then weave that page-definition revision.
- `19 -> 20 -> 21`: import outside-origin Markdown into governed artifact `bob/page-main`, then weave Bob's page-definition revision so `bob/index.html` follows that imported governed artifact. `bob/page-main` itself intentionally remains unwoven in this pair.
- `21 -> 22 -> 23`: add the root Knop later in the mesh lifecycle, then weave it so root `index.html` and root `_knop` support surfaces exist.
- `23 -> 24 -> 25`: add the root page definition, root repo-tour content, root sidebar, and root stylesheet, then weave that root page-definition revision so `index.html` becomes a customized root page.

Conventions used here:

- one Accord `Manifest` per file
- one `TransitionCase` per manifest for now
- RDF-bearing files use `rdfCanonical`
- generated HTML pages use `text`
- non-text control files such as `.nojekyll` use `bytes`

The first transition, `00-blank-slate` -> `01-source-only`, is fixture seeding rather than a public Semantic Flow operation. Its `operationId` is intentionally named as fixture setup.

These manifests are authored before any dedicated runner. They are meant to serve as the normative acceptance layer that a later pseudo-runner or validator will execute against.

For the current Alice reference transitions, the active convention is:

- the `ReferenceCatalog` support artifact lives at `alice/_knop/_references`
- the `ReferenceLink` identities are stable fragment IRIs rooted at that catalog resource
