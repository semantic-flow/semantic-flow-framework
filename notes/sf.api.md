---
id: 9qb50tea1tul63dt6tlnaac
title: Semantic Flow API
desc: ''
updated: 1775267145775
created: 1774307702138
---


## Draft Job Kinds

This note uses the machine-facing `kind` / `operationId` tokens consistently.

Human-facing command surfaces may still spell these as phrases such as `mesh create` or `knop create`, but the API-facing names in this note use the current draft token forms.

The following draft job kinds have come up as likely first-class Semantic Flow operations:

- `mesh.create`
- `knop.create`
- `knop.addReference`
- `integrate`
- `version`
- `validate`
- `generate`
- `weave` : validate, version, and generate
- `extract` : create Knops for local RDF references in an `RdfDocument`

For the thin public contract, the current direction is to model all submitted work uniformly as `Job`s, even when some implementations may execute quickly enough to feel synchronous to a client.

At minimum, `integrate`, `version`, `mesh.create`, and large validation or generation work should be treated as likely candidates for first-class long-running jobs.

## First Concrete Slice

The first concrete carried public operation example is `mesh.create`.

Current direction for that slice:

- the thin public contract should stay semantic and implementation-neutral
- a `mesh.create` request should define the mesh identity being established, starting with `meshBase`
- `meshIri` remains distinct from `meshBase`, but may be explicit or conventionally derived as `_mesh` resolved against `meshBase`
- host filesystem paths should stay out of the thin core contract even if a specific implementation such as Weave accepts them locally
- the successful result should at minimum make the created `SemanticMesh`, `MeshMetadata`, and `MeshInventory` resources discoverable

Worked examples for that slice now live beside the other Alice Bio API examples in `../examples/alice-bio/api/`.

## Second Concrete Slice

The second concrete carried public operation example is `knop.create`.

Current direction for that slice:

- the target should identify an existing mesh together with one `designatorPath`
- `knopIri` remains distinct from `designatorPath`, but may be explicit or conventionally derived as `designatorPath + "/_knop"` resolved against `meshBase`
- host filesystem paths should stay out of the thin core contract even if a specific implementation such as Weave resolves mesh state from a local workspace
- the successful result should at minimum make the created `Knop`, `KnopMetadata`, and `KnopInventory` resources discoverable and surface that the `MeshInventory` was updated

Worked examples for that slice now also live in `../examples/alice-bio/api/`.

## Third Concrete Slice

The third concrete carried public operation example is `weave`.

Current direction for that slice:

- the target should identify an existing mesh and may optionally narrow the local focus with one or more `designatorPaths`
- the thin request should default to the full high-level `weave` behavior rather than requiring an explicit `steps` object for the common case
- host filesystem paths should stay out of the thin core contract even if a specific implementation such as Weave uses a local workspace as the execution substrate
- the successful result should at minimum make newly created histories or historical states discoverable and surface the updated current artifacts that were versioned and rendered

Worked examples for that slice now also live in `../examples/alice-bio/api/`.

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
