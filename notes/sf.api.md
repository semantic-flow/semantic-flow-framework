---
id: 9qb50tea1tul63dt6tlnaac
title: Semantic Flow API
desc: ''
updated: 1774502021396
created: 1774307702138
---


## Tentative Operations

The following operations have come up as likely first-class Semantic Flow operations, though naming and exact boundaries are still tentative:

- `mesh create`
- `knop create`
- `knop.addReference`
- `integrate`
- `version`
- `validate`
- `generate`
- `weave` : version, validate, and generate
- `extract`: create knops for local RDF references in RdfDocument

For the thin public contract, the current direction is to model all submitted work uniformly as `Job`s, even when some implementations may execute quickly enough to feel synchronous to a client.

At minimum, `integrate`, `version`, `mesh create`, and large validation or generation work should be treated as likely candidates for first-class long-running jobs.

## Mesh Identity

The current direction is to distinguish:

- `meshBase` : the canonical base IRI used to form Semantic Flow identifiers in a mesh
- `meshIri` : the canonical identifier of the `SemanticMesh` resource itself

By convention, `meshIri` is the mesh handle resolved as `_mesh` against `meshBase`.

This keeps namespace formation separate from mesh-resource identity, aligns the mesh model more closely with the handle pattern already used for `Knop`, and avoids overloading `meshBase` with two different roles.

## Hydra Layer

The current direction is to keep `OpenAPI` as the normative HTTP contract while adding a `Hydra` hypermedia layer to JSON-LD representations.

That split lets the API support both:

- design-time understanding of the contract through OpenAPI
- runtime exploration of resources, collections, searches, and follow-up actions through Hydra

This is especially attractive for Semantic Flow because follow-up actions are often discoverable only after a resource exists. A newly created `Knop` may expose Hydra affordances for actions such as:

- `knop.addReference`
- `weave`
- `validate`
- `generate`

Under this model, Hydra advertises what can be done next with a specific resource, while submitted work is still executed uniformly as `Job`s.

## Examples

Worked example payloads live outside the OpenAPI file in `../examples/alice-bio/`.

See `sf.api.examples.md` for the current example set and how it maps back to the ontology use case.
