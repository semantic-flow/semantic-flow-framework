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
- `import`
- `version`
- `validate`
- `generate`
- `weave` : validate, version, and generate
- `extract` : create identifiers and Knops from terms mentioned in an existing `RdfDocument`

`payload.update` is currently better understood as a local convenience mutation surface than as a strong first-class Semantic Flow job kind. Replacing current working bytes is real implementation work, but the semantically meaningful lifecycle step is still `version` / `weave`, which records those bytes into explicit artifact history.

For the thin public contract, the current direction is to model all submitted work uniformly as `Job`s, even when some implementations may execute quickly enough to feel synchronous to a client.

At minimum, `integrate`, `version`, `mesh.create`, and large validation or generation work should be treated as likely candidates for first-class long-running jobs.

## Concrete Slices

Ordering is not especially important here. The useful distinction is between:

- concrete carried slices that already have worked API examples
- adjacent next slices that should be reflected in the thin API note before they are fully exemplified

### Current Carried Slices

#### `mesh.create`

Current direction for that slice:

- the thin public contract should stay semantic and implementation-neutral
- a `mesh.create` request should define the mesh identity being established, starting with `meshBase`
- `meshIri` remains distinct from `meshBase`, but may be explicit or conventionally derived as `_mesh` resolved against `meshBase`
- host filesystem paths should stay out of the thin core contract even if a specific implementation such as Weave accepts them locally
- the successful result should at minimum make the created `SemanticMesh`, `MeshMetadata`, and `MeshInventory` resources discoverable

Worked examples for that slice now live beside the other Alice Bio API examples in `../examples/alice-bio/api/`.

#### `knop.create`

Current direction for that slice:

- the target should identify an existing mesh together with one `designatorPath`
- `knopIri` remains distinct from `designatorPath`, but may be explicit or conventionally derived as `designatorPath + "/_knop"` resolved against `meshBase`
- host filesystem paths should stay out of the thin core contract even if a specific implementation such as Weave resolves mesh state from a local workspace
- the successful result should at minimum make the created `Knop`, `KnopMetadata`, and `KnopInventory` resources discoverable and surface that the `MeshInventory` was updated

Worked examples for that slice now also live in `../examples/alice-bio/api/`.

#### `integrate`

Current direction for that slice:

- the target should identify an existing mesh together with one `designatorPath`
- the thin request should also carry one semantic `sourceUri` for the bytes being integrated
- host filesystem paths should stay out of the thin core contract even if a local implementation such as Weave accepts paths or `file:` URLs at its CLI/runtime boundary
- the successful result should at minimum make the created payload artifact and Knop surfaces discoverable and surface that the `MeshInventory` was updated

Worked examples for that slice now also live in `../examples/alice-bio/api/`.

#### `weave`

Current direction for that slice:

- the target should identify an existing mesh and may optionally narrow the local focus with one or more `designatorPaths`
- the thin request should default to the full high-level `weave` behavior rather than requiring an explicit `steps` object for the common case
- host filesystem paths should stay out of the thin core contract even if a specific implementation such as Weave uses a local workspace as the execution substrate
- the successful result should at minimum make newly created histories or historical states discoverable and surface the updated current artifacts that were versioned and rendered

Worked examples for that slice now also live in `../examples/alice-bio/api/`.

### Next Likely Slices

#### `extract`

Current direction for that slice:

- `extract` should operate over an existing ingested `RdfDocument` or similar already-governed source artifact, not over arbitrary host-local source bytes
- the typical use case is minting identifiers and Knops for the terms mentioned in that source, for example creating identifier surfaces for ontology terms referenced in an ontology document
- the thin request should identify the existing source artifact being extracted from and may later narrow scope with selectors, target classes, or similar extraction criteria
- the successful result should at minimum make the created identifiers, Knops, and their discoverable support surfaces visible as outputs of the extraction
- a specific implementation may still carry a narrower first local extract slice, for example extracting one explicitly targeted resource from one existing woven `RdfDocument`, but that narrower runtime slice should not define the broader API concept

Worked examples for this slice are still thinner than the carried `mesh.create` / `knop.create` / `integrate` / `weave` set and should be expanded deliberately rather than implied from the fixture ladder alone.

### Local Convenience Surfaces

#### `payload.update`

Current direction for that slice:

- `payload.update` should not currently be treated as a core semantic job kind on par with `integrate`, `import`, or `weave`
- it is better understood as a convenience mutation for replacing the current working bytes of an already-known payload surface
- the semantically meaningful lifecycle step remains `version` / `weave`, not the replacement itself
- a local implementation such as Weave may still keep this command for repo/workspace ergonomics even if the thin public API does not emphasize it

Worked examples may still exist for this surface in `../examples/alice-bio/api/`, but they should be read as convenience examples rather than as a strong signal that `payload.update` belongs in the core job taxonomy.

#### `import`

Current direction for that slice:

- `import` should be the explicit boundary for bringing outside-the-tree or extra-mesh content into a governed in-tree artifact
- `import` should stay distinct from `integrate`
- `integrate` associates bytes with a target designator and payload surface, while `import` establishes a governed local artifact boundary that later operations may follow
- the thin request should identify an existing mesh together with one semantic outside source, not a host-specific local-path contract
- the successful result should at minimum make the imported governed artifact and its current working file discoverable

This matters for resource-page source behavior in particular: a page definition should follow the imported in-tree artifact's current working file, not a direct live outside source.

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
