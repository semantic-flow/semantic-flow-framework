---
id: 7w9q2m4p6x1k8r3t5v0n2hb
title: 2026 04 04 Integrate Behavior
desc: ''
updated: 1775365200000
created: 1775365200000
---

## Purpose

This note captures the current expected behavior of the first `integrate` slice for Semantic Flow.

It is a current Semantic Flow behavior spec for the first carried path, not the complete thin public API contract.

## Status

This is the current next carried slice after the first completed local `weave` implementation.

The first acceptance target is the settled `mesh-alice-bio` transition from `05-alice-knop-created-woven` to `06-alice-bio-integrated`.

## Inputs

- `designatorPath` is required.
- the first local CLI surface takes the source artifact as the primary positional input and requires `designatorPath` either as a second positional argument or via `--designator-path`
- the target workspace must already contain `_mesh/_meta/meta.ttl` and `_mesh/_inventory/inventory.ttl`
- `meshBase` is resolved from the existing mesh metadata rather than being repeated on the CLI
- the current local runtime slice accepts an existing local source file addressed either by path or equivalent `file:` URL
- source files outside the mesh root must be allowed by operational local-path policy before `integrate` can follow them; workspace-contained grants can be mesh-carried, while host-local grants may approve separate checkouts without making absolute host paths public mesh facts
- extra-mesh working-source bindings record the approved working locator with `artifactResolutionMode_working` while leaving the bytes in the source checkout
- repository-backed source bindings may additionally record repository URL/ref/path plus optional commit and digest evidence when that evidence is deliberately supplied
- broader runtime profiles may add latest-state or exact source policy; if a source file is copied into the mesh/publication tree first, that copy step is `import`, not `integrate`
- shared `core` planning should operate on the resulting semantic source locator and working-file locator rather than on an absolute host filesystem path

## What Integrate Does

`integrate` establishes the first payload-artifact surface for a designator in an existing mesh.

The semantic boundary is the same whether the source bytes stay in the mesh tree, an adjacent source root, the same repository, a publication branch, or a separate repository. Topology affects operational policy and provenance; it should not decide whether the operation is called `integrate`.

`integrate` leaves the source bytes where they are. It records or follows a source locator with explicit working, latest-state, or exact resolution policy. Bringing bytes into the mesh or publication tree as a new governed local copy is `import`. If `integrate` is later run against that imported copy, the copy already exists before integration begins.

In the current first slice, that means:

- creating `D/_knop/_meta/meta.ttl`
- creating `D/_knop/_inventory/inventory.ttl`
- creating `D/_knop/_sources/sources.ttl` when the source is outside the mesh root or the request supplies repository-backed source metadata
- updating `_mesh/_inventory/inventory.ttl` so the mesh registers `D/_knop`
- updating `_mesh/_inventory/inventory.ttl` so the payload artifact `D` is a `PayloadArtifact` with `hasWorkingLocatedFile` for mesh-local working files or `workingLocalRelativePath` for approved extra-mesh working files
- keeping the working payload bytes at the existing `alice-bio.ttl` path for the carried Alice Bio slice rather than relocating them during `integrate`

When the integrated source is outside the mesh root, the source registry records a `KnopSourceRegistry` source binding for the payload artifact. The default binding uses the deterministic internal `payload-source` fragment id, `targetLocalRelativePath` for the local operational locator, and `artifactResolutionMode_working` for mutable working-source resolution. Floating working bindings do not persist repository ref, commit, path, digest evidence, or `expectsContentDigest`.

When repository-backed metadata is deliberately supplied, the source registry may also record `targetRepositorySource` for repository URL/ref/path plus optional commit evidence. A runtime may record a digest it observed from the local source bytes, such as `sha256:<hex>`, as `expectsContentDigest` on the binding and `hasContentDigest` on the repository locator for that repository-backed binding.

## What Integrate Does Not Do

In this first slice, `integrate` does not:

- create explicit artifact history
- create `alice/bio/index.html` or any Knop support-artifact pages
- run `weave`, `version`, `validate`, or `generate`
- auto-create referenced-resource Knops such as `bob`
- copy, relocate, import, or rewrite the working payload bytes
- fetch, copy, import, or refresh remote/separate-repository sources implicitly; any source copy or later update/refresh must be explicitly requested and policy-approved
- introduce daemon behavior

## Invariants

- the source payload bytes remain unchanged by the operation
- any source locator recorded in public mesh state must be portable and provenance-bearing, not an absolute checkout-local path
- `_mesh/_meta/meta.ttl` and previously woven pages such as `alice/index.html` remain unchanged
- the created and updated files should match the current intended `06-alice-bio-integrated` fixture state for Alice Bio
- if the target payload-Knop support-artifact files already exist, the operation should fail closed rather than silently overwrite them
- runtime-local `.weave/logs` output is not part of the semantic mesh surface

## Acceptance Reference

The first behavior-level comparison target is:

- fixture repo: `dependencies/github.com/semantic-flow/mesh-alice-bio`
- from ref: `05-alice-knop-created-woven`
- to ref: `06-alice-bio-integrated`
- manifest: `dependencies/github.com/semantic-flow/semantic-flow-framework/examples/alice-bio/conformance/06-alice-bio-integrated.jsonld`
- local CLI execution should match that manifest-scoped result

## Related Specs

- [[sf.spec.2026-05-18-publication-source-binding]]
