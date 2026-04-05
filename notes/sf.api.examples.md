---
id: 2c93t4j3o8m9k7p5v1n6x0q
title: API Examples
desc: ''
updated: 1774416095095
created: 1774339800000
---

This note groups worked Semantic Flow API example payloads that are kept outside the OpenAPI file so the normative contract can stay lean.

## Alice Bio

The current primary worked example set lives in `../examples/alice-bio/api`.

Those files are anchored to the existing ontology use case in [Alice Bio](../../ontology/notes/ont.use-cases.alice-bio.md).

Current files:

- `mesh-create-job-request.jsonld` : the request payload for submitting `mesh.create`
- `mesh-create-job-accepted.jsonld` : the accepted `Job` resource returned immediately after `mesh.create` submission
- `mesh-create-job-succeeded.jsonld` : the terminal successful `mesh.create` `Job` resource with created mesh-surface resources
- `knop-create-job-request.jsonld` : the request payload for submitting `knop.create`
- `knop-create-job-succeeded.jsonld` : the terminal successful `knop.create` `Job` resource with created Knop resources and the updated mesh inventory
- `integrate-job-request.jsonld` : the request payload for submitting `integrate` with one `designatorPath` and one semantic `sourceUri`
- `integrate-job-succeeded.jsonld` : the terminal successful `integrate` `Job` resource with the created payload and Knop resources plus the updated mesh inventory
- `payload-update-job-request.jsonld` : the request payload for submitting `payload.update` with one `designatorPath` and one semantic replacement `sourceUri`
- `payload-update-job-succeeded.jsonld` : the terminal successful `payload.update` `Job` resource with the updated payload artifact
- `weave-job-request.jsonld` : the request payload for submitting `weave` over the Alice mesh with a narrowed `designatorPaths` target
- `weave-job-succeeded.jsonld` : the terminal successful `weave` `Job` resource with created histories and updated current artifacts
- `mesh.jsonld` : a `SemanticMesh` representation with Hydra affordances for follow-up actions
- `knop.jsonld` : a `Knop` representation with Hydra affordances for follow-up actions
- `knop-add-reference-job-request.jsonld` : the request payload for submitting `knop.addReference`
- `job-accepted.jsonld` : the accepted `Job` resource returned immediately after submission
- `job-succeeded.jsonld` : the terminal successful `Job` resource with result links
- `reference-link.jsonld` : the created `ReferenceLink` resource

This is meant to be a realistic vertical slice rather than an exhaustive example catalog.
