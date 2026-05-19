---
id: s2n1wdqg8mzz9xg1k4xw3cb
title: 2026 04 03 Mesh Create
desc: ''
updated: 1775277600000
created: 1775277600000
---

## Purpose

This note captures the current expected behavior of the first `mesh.create` / `mesh create` slice for Semantic Flow.

It is a current Semantic Flow behavior spec for the first carried bootstrap path, not the complete thin public API contract.

## Status

This is the current bootstrap target.

The first acceptance target is the settled `mesh-alice-bio` transition from `01-source-only` to `02-mesh-created`.

## Inputs

- `meshBase` is required.
- `meshBase` must be an absolute IRI and must end with a trailing `/`.
- `workspace` identifies the local workspace root, is resolved from the command working directory, and defaults to `.`
- `meshRoot` identifies the mesh root path, is resolved from the command working directory, must stay inside `workspace`, and defaults to `.`
- publication-host controls such as GitHub Pages `.nojekyll` are outside implicit core `mesh.create`; they belong to a selected or resolved publication-host preset, which may be composed with mesh creation at request time
- the target workspace may already contain non-mesh files such as a source RDF document

## What Mesh Create Does

`mesh create` establishes the first mesh-managed support surface for a workspace. For a whole-workspace mesh, the mesh root is the workspace root. For a sidecar mesh, the workspace root is the containing project and the mesh root is a child path such as `docs/`.

In the current bootstrap slice, that means creating:

- `_mesh/_meta/meta.ttl`
- `_mesh/_inventory/inventory.ttl`
- `_mesh/_config/config.ttl`, only when the mesh root differs from the workspace root

Those paths are relative to the mesh root. With `--workspace . --mesh-root docs`, the created support files include `docs/_mesh/_meta/meta.ttl`, `docs/_mesh/_inventory/inventory.ttl`, and `docs/_mesh/_config/config.ttl`.

For a sidecar mesh root such as `docs/`, `mesh create` also creates `docs/_mesh/_config/config.ttl`. The config is an `sfcfg:MeshConfig` and records the portable workspace relationship with `sfcfg:workspaceRootRelativeToMeshRoot "../"`. Whole-workspace meshes do not get a config file solely to record `"."`.

Core `mesh.create` does not create static-host control files as hidden bootstrap behavior. A user-facing create request may compose mesh creation with a publication-host preset such as GitHub Pages, either by selecting it explicitly or by using a conservative `auto` publication profile that resolves to a concrete preset. In that case the preset may create `.nojekyll` during the same operation, but that file is a host control rather than an RDF support artifact and is not listed in mesh inventory. Custom-domain host files are human-owned for now.

If `auto` profile resolution is used, the operation result should report the resolved publication profile, including `none` when no profile was applied. Host inference from `meshBase` should be conservative: for now, `*.github.io` is the only strong GitHub Pages signal. An arbitrary custom domain is not enough by itself, and CI metadata or repository remotes should not participate in auto-inference.

When a publication profile is selected or resolved, `mesh.create` should persist the concrete profile in `MeshConfig` using `sfcfg:hasPublicationProfile`. Persist the resolved value such as `sfcfg:publicationProfile_githubPages` or `sfcfg:publicationProfile_none`, not the request-time `auto` mode. A whole-workspace mesh that otherwise would not need `_mesh/_config/config.ttl` may still create one in order to carry this portable mesh setting.

The created RDF should establish at least:

- the `SemanticMesh` resource at `_mesh`
- the `meshBase`
- the `MeshMetadata` artifact
- the `MeshInventory` artifact
- working located-file links for the metadata and inventory Turtle files in inventory
- for sidecar meshes, a mesh-owned config artifact recording the workspace root relative to the mesh root
- when a publication profile was selected or resolved, a mesh-owned config artifact recording the resolved `sfcfg:hasPublicationProfile`

## What Mesh Create Does Not Do

In this first slice, `mesh create` does not:

- create any `Knop`
- create payload history
- generate `ResourcePage` HTML
- add local path access grants
- apply publication-host preset controls such as GitHub Pages `.nojekyll` unless the caller selected that preset or selected `auto` and the operation resolved `auto` to that preset
- run full `weave`
- introduce daemon behavior

## Invariants

- existing non-mesh workspace files remain unchanged
- the first carried Alice Bio path should leave `alice-bio.ttl` byte-identical to the `01-source-only` state
- the created mesh support files should match the current intended `02-mesh-created` fixture state for Alice Bio
- `meshRoot` must stay inside the workspace root
- whole-root meshes do not create `_mesh/_config/config.ttl` solely for the workspace relationship
- whole-root meshes may create `_mesh/_config/config.ttl` when they need to persist a selected or resolved publication profile
- sidecar mesh config records a portable relative path and no extra-mesh access grants
- if target support-artifact files already exist, the operation should fail closed rather than silently overwrite them
- runtime-local `.weave/logs` output is not part of the semantic mesh surface

## Acceptance Reference

The first behavior-level comparison target is:

- fixture repo: `dependencies/github.com/semantic-flow/mesh-alice-bio`
- from ref: `01-source-only`
- to ref: `02-mesh-created`
- manifest: `dependencies/github.com/semantic-flow/semantic-flow-framework/examples/alice-bio/conformance/02-mesh-created.jsonld`
- local CLI execution should match that manifest-scoped result

## Related Specs

- [[sf.spec.2026-05-18-publication-source-binding]]
