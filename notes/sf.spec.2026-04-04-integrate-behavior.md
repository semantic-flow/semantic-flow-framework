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
- the current first local runtime slice accepts an existing local source file in the workspace, addressed either by path or equivalent `file:` URL
- broader runtime profiles may accept policy-approved adjacent, branch, or separate-repository source files with current or pinned source policy; if a source file is copied into the mesh/publication tree first, that copy step is `import`, not `integrate`
- shared `core` planning should operate on the resulting semantic source locator and working-file locator rather than on an absolute host filesystem path

## What Integrate Does

`integrate` establishes the first payload-artifact surface for a designator in an existing mesh.

The semantic boundary is the same whether the source bytes stay in the mesh tree, an adjacent source root, the same repository, a publication branch, or a separate repository. Topology affects operational policy and provenance; it should not decide whether the operation is called `integrate`.

`integrate` leaves the source bytes where they are. It records or follows a source locator with an explicit current or pinned resolution policy. Bringing bytes into the mesh or publication tree as a new governed local copy is `import`. If `integrate` is later run against that imported copy, the copy already exists before integration begins.

In the current first slice, that means:

- creating `D/_knop/_meta/meta.ttl`
- creating `D/_knop/_inventory/inventory.ttl`
- updating `_mesh/_inventory/inventory.ttl` so the mesh registers `D/_knop`
- updating `_mesh/_inventory/inventory.ttl` so the payload artifact `D` is a `PayloadArtifact` with `hasWorkingLocatedFile` pointing at the existing working file
- keeping the working payload bytes at the existing `alice-bio.ttl` path for the carried Alice Bio slice rather than relocating them during `integrate`

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
