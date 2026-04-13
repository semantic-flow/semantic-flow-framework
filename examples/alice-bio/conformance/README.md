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
