---
id: 9qb50tea1tul63dt6tlnaac
title: Semantic Flow API
desc: ''
updated: 1779162382393
created: 1774307702138
---


## Role

This note is the high-level map of the Semantic Flow API surface.

Detailed behavior belongs in the corresponding `sf.spec.*` notes. Worked JSON-LD payload examples live in [[sf.api.examples]]. This note should stay small enough that it can point readers to the right contract without restating that contract.

Machine-facing API examples use the `kind` / `operationId` token forms below. Human-facing command surfaces may spell the same operations as phrases such as `mesh create` or `knop create`.

## Job Model

Submitted work should be modeled uniformly as `Job`s, even when an implementation executes some jobs synchronously.

Likely first-class job kinds:

- `mesh.create`
- `knop.create`
- `knop.addReference`
- `integrate`
- `import`
- `version`
- `validate`
- `generate`
- `weave`
- `extract`

`payload.update` is still better understood as a local convenience mutation surface. Replacing current working bytes is real implementation work, but the semantically meaningful lifecycle step remains `version` or the versioning phase of `weave` when a new historical state is intended.

At minimum, `mesh.create`, `integrate`, `version`, `weave`, and large validation/generation work should be treated as likely long-running jobs.

## Operation Map

- `mesh.create`: establishes the first `SemanticMesh`, `MeshMetadata`, and `MeshInventory` support surface for a mesh root. See [[sf.spec.2026-04-03-mesh-create]].
- `knop.create`: creates the first Knop-managed support surface for one designator in an existing mesh. See [[sf.spec.2026-04-03-knop-create]].
- `integrate`: binds available source bytes to a target designator and payload artifact surface without moving the source bytes. Source bytes may be mesh-local, policy-approved live local bytes, or repository-backed bytes resolved with working, latest-state, or exact source policy. Sidecar and branch-published ontology payloads should use `integrate`. See [[sf.spec.2026-04-04-integrate-behavior]] and [[sf.spec.2026-05-18-publication-source-binding]].
- `knop.addReference`: creates or updates a Knop-owned `ReferenceCatalog` with curated `ReferenceLink` facts. See [[sf.spec.2026-04-04-knop-add-reference-behavior]].
- `weave`: the default orchestration job for already governed targets. By default it runs the configured versioning, validation, and generation phases; the versioning phase may append historical states, while generation can also render unversioned governed artifacts. It may expose options such as `--validate-before` and `--validate-after` for whole-mesh validation, but it does not create semantic integrations, fetch source repositories, apply host presets, or publish git refs. See [[sf.spec.2026-04-03-weave-behavior]].
- `extract`: creates identifiers and Knop support surfaces for resources already mentioned in governed source artifacts. See [[sf.spec.2026-04-05-extract-behavior]].
- `payload.update`: replaces the current working bytes of an already managed payload artifact as a local convenience, without itself versioning or rendering. See [[sf.spec.2026-04-04-payload-update-behavior]].
- `import`: copies a working file into the mesh or publication tree so that the copy becomes a governed local working file. It is not the operation for ordinary sidecar or branch-published ontology source binding. The full `import` operation still needs its own behavior spec; first-pass page-source constraints are covered in [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]].
- payload version intent: a narrow metadata update for selecting a payload's current/default `ArtifactHistory` and optional next state segment without creating a state. This is payload-only and is consumed by `version` or the versioning phase of `weave`.
- `version`: a narrower operation for explicitly appending versioned payload states, using selected history and next-state intent when supplied.
- `validate`: a narrower operation for reporting problems without recording new state. Initial scopes are `validate mesh` for whole-mesh validation and `validate publication` for narrower publication-readiness checks. Whole-mesh validation should include retained publication checks when a publication surface or profile exists.
- `generate`: a narrower operation for rendering ResourcePages and other generated surfaces from the current mesh state.
- publication/source binding: not a single core job kind by default. It is a composed boundary for integration, optional import, publication validation, host presets, optional git output handling, and later explicit update/refresh when a source locator, source policy, or imported copy changes. See [[sf.spec.2026-05-18-publication-source-binding]].

Identifier-page customization and root lifecycle behavior are specified separately because they are mostly about generated public page authority and source resolution rather than a standalone submitted job. See [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]].

## Boundary Principles

The thin API should keep the workspace/mesh boundary crisp:

- a workspace is an execution and storage boundary used by an implementation.
- a mesh is the governed Semantic Flow namespace and artifact graph rooted at a `meshBase`.
- local paths, worktrees, branch names, checkout roots, source-directory grants, publication refs, and deployment targets are implementation/runtime concerns.
- mesh facts are the durable RDF assertions that remain meaningful after the local command has finished.

`integrate` associates available bytes with a target designator and payload surface while leaving the source where it is. `import` copies a working file into the mesh or publication tree so the copy becomes governed local working content. These are related but distinct boundaries.

Same-repository, separate-repository, sidecar, whole-repository, and branch-published topologies should use the same Semantic Flow artifact model. The topology affects operational configuration and provenance, not whether a separate API concept exists.

Publication-host controls are modular presets. For now, the GitHub Pages preset creates or validates `.nojekyll` only; custom-domain host files are human-owned. Core `mesh.create` should not hide host-specific files inside bootstrap behavior, but a user-facing create request may compose mesh creation with a selected publication profile. A conservative `auto` profile may infer from strong signals such as `*.github.io`, as long as the resolved profile is reported, can be overridden, and is persisted as concrete mesh config rather than as `auto`.

## Branch-Published Ontology Sequence

A branch-published ontology site should be described as composed Semantic Flow operations, not as a special `prepare gh-pages` API operation.

The intended portable shape is:

- create or locate the publication mesh with `mesh.create`.
- apply the selected publication-host preset if one is needed.
- make source-lane bytes available through mesh-local files, allowed live local locators, or repository-backed locators with working, latest-state, or exact source policy.
- `integrate` governed payload artifacts at their target designator paths.
- run default `weave` for the normal version/validate/generate workflow over already integrated targets.
- use narrower `version`, `validate mesh`, `validate publication`, or `generate` operations when a workflow needs those phases separated.
- run extraction and reference curation over governed artifacts when term pages or curated links are needed.
- validate and publish the resulting tree through implementation-specific CI/CD or git output handling.

The root designator can identify an ordinary welcome/about resource for the publication while `_mesh` remains the standardized `SemanticMesh` resource. Root page composition can use the generic identifier-page customization model only when default rendering is not enough.

## Mesh Identity

The current direction is to distinguish:

- `meshBase`: the canonical base IRI used to form Semantic Flow identifiers in a mesh.
- `meshIri`: the canonical identifier of the `SemanticMesh` resource itself.

By convention, `meshIri` is the mesh handle resolved as `_mesh` against `meshBase`.

This keeps namespace formation separate from mesh-resource identity, aligns the mesh model with the handle pattern already used for `Knop`, and avoids overloading `meshBase` with two different roles.

## Hydra Layer

The current direction is to keep OpenAPI as the normative HTTP contract while adding a Hydra hypermedia layer to JSON-LD representations.

That split lets the API support both:

- design-time understanding of the contract through OpenAPI.
- runtime exploration of resources, collections, searches, and follow-up actions through Hydra.

A newly created `Knop` may expose Hydra affordances for actions such as `knop.addReference`, `weave`, `validate`, or `generate`. Under this model, Hydra advertises what can be done next with a specific resource, while submitted work is still executed uniformly as `Job`s.

## Examples

Worked example payloads live outside the OpenAPI file in `../examples/alice-bio/` and related fixture families. See [[sf.api.examples]] for the current example set and how it maps back to the ontology use cases.
