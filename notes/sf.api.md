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

## Workspace, Mesh, Integrate, And Import

The thin API should keep the workspace/mesh boundary crisp:

- a workspace is an execution and storage boundary used by an implementation
- a mesh is the governed Semantic Flow namespace and artifact graph rooted at a `meshBase`
- local paths, worktrees, branch names, checkout roots, and source-directory grants are implementation/runtime concerns
- mesh facts are the durable RDF assertions that remain meaningful after the local command has finished

`integrate` binds existing bytes to a mesh designator and creates or updates the governed payload surface for that designator. In a local implementation, those bytes may already be inside the mesh root or may sit in a policy-approved adjacent source directory. The durable result is not "this host path was used"; it is the mesh artifact, its working-byte locator, source provenance, and the updated mesh/Knop support surfaces.

`import` is the boundary-crossing operation for content that should first become a governed in-tree/local artifact before other mesh behavior follows it. This matters most for outside-origin or extra-workspace content, especially page sources. Import establishes the local governed artifact boundary; later `integrate`, page generation, `version`, or `weave` follows that governed artifact rather than a live outside source.

For a branch-published site, files read from the source checkout and bound into the publication mesh are usually `integrate` inputs, not imports. They are trusted source-lane files being associated with mesh identifiers. Use `import` when the operation needs to bring content across an outside boundary and create the governed local copy that later operations should follow.

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
- the thin request should also carry one semantic source locator for the bytes being integrated
- host filesystem paths should stay out of the thin core contract even if a local implementation such as Weave accepts paths or `file:` URLs at its CLI/runtime boundary
- the local/runtime slice should support both in-mesh local files and policy-approved adjacent source files, because that is the main path for associating current bytes with a payload artifact before `weave` / `version`
- when a local implementation explicitly integrates an adjacent current file, it may also create or update a narrow `MeshConfig` access grant for that source so later `weave` / `version` runs can resolve the carried `workingLocalRelativePath`
- that generated grant should be visible in mesh-owned config, constrained to the integrated source file or an explicitly requested source directory, and should not imply workspace-wide access
- the successful result should at minimum make the created payload artifact and Knop surfaces discoverable and surface that the `MeshInventory` was updated

Settled boundary:

- the current Weave CLI/runtime slice already supports mesh-local and policy-approved adjacent local file sources
- exact-file access remains implicit in the selected source; directory-prefix grants are explicit local runtime input
- remote or outside-origin content should generally cross an `import` boundary first, then be integrated or followed as a governed local artifact
- `integrate` should stay the ordinary operation for binding source-repository files into a branch-published publication mesh

Worked examples for that slice now also live in `../examples/alice-bio/api/`.

#### `weave`

Current direction for that slice:

- the target should identify an existing mesh and may optionally narrow the local focus with one or more `designatorPaths`
- target entries may carry version-naming fields for payload artifacts, such as `historySegment`, `stateSegment`, and `manifestationSegment`, when the operation includes versioning
- a broader target may carry general payload version segment defaults for all matched payload artifacts, while more specific target entries can override those defaults
- version-naming fields should not rename support artifacts unless a separate support-artifact naming contract is defined
- once a payload history has established a named state such as `v0.0.1`, a later `weave` or `version` should fail before writes if that payload would be versioned without an explicit next `stateSegment` or explicit ordinal fallback segment
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
- extraction provenance should be visible as source provenance, not as a curated reference: the extracted Knop links to one primary `ExtractionSource`, while the `ExtractionSource` details live in the Knop's `_knop/_sources` registry together with other source bindings
- `ReferenceLink`s remain the curated reference surface; a later API may derive reference-link candidates from extraction provenance, but extraction should not silently collapse provenance into reference curation
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

- `import` should be the explicit boundary for bringing outside-origin or extra-workspace content into a governed local artifact
- `import` should stay distinct from `integrate`
- `integrate` associates available bytes with a target designator and payload surface, while `import` establishes the governed local artifact boundary that later operations may follow
- the thin request should identify an existing mesh together with one semantic outside source, not a host-specific local-path contract as the durable API fact
- the successful result should at minimum make the imported governed artifact and its current working file discoverable

In the usual import shape, the imported governed copy becomes the artifact's current working file. If that copy is mesh-addressable, it should normally be modeled as the artifact's `hasWorkingLocatedFile`; if the runtime also needs an operational path literal, `workingLocalRelativePath` should identify the same local copy. The outside origin should remain source/provenance, or an explicitly modeled remote locator such as `workingAccessUrl`, rather than becoming the file that page generation follows directly.

This matters for resource-page source behavior in particular: a page definition should follow the imported governed artifact's current working file, not a direct live outside source.

## Branch-Published Ontology Sequence

A branch-published ontology site should be described as a mesh setup and integration sequence, not as a bulk import.

For a source branch such as `main` and a publication branch such as `gh-pages`, the intended portable shape is:

- create the publication mesh with `mesh.create`, using the public Pages base as `meshBase`
- create a root Knop with `knop.create` for `/` when the mesh root itself should be dereferenceable as a Semantic Flow identifier
- create a small root welcome/about RDF artifact such as `welcome.ttl` and `integrate` it at `/`
- put root-page title and description in that RDF, for example with `dcterms:title` and `dcterms:description`
- let the default ResourcePage renderer use the root payload RDF facts before reaching for a custom `_knop/_page` page-definition artifact
- integrate source-lane ontology files such as core, config, and SHACL Turtle documents as ordinary payload artifacts at their target designator paths
- weave/version those payload artifacts with the desired release/history/state/manifestation names
- run all-terms extraction from the governed source artifacts, with explicit source references when those references should become curated mesh facts
- weave/generate the extracted term pages and preview the publication branch before pushing

The root designator does not have to identify "the semantic site" as a special kind of resource. For this ontology publication it can simply identify the publication's welcome/about resource, while `_mesh` remains the standardized `SemanticMesh` resource. This keeps three things separate:

- root welcome/about description: ordinary RDF payload for `/`
- mesh description: standardized `_mesh` resources
- page composition: optional `ResourcePageDefinition` only when default presentation is not enough
- outside-origin acquisition: future `import`, used when content must first cross into a governed local artifact

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
