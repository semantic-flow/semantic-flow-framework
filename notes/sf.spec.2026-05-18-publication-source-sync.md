---
id: p5vx8gm2q19h4e7w0t6n3ab
title: 2026 05 18 Publication Source Sync
desc: ''
updated: 1779113220000
created: 1779113220000
---

## Purpose

This note captures the intended Semantic Flow behavior boundary for publication setup, import/source synchronization, and host-specific publication controls.

It is a behavior spec for dissolving the old idea that a dedicated `prepare gh-pages` operation owns branch-published setup. A Weave implementation may keep `prepare gh-pages` temporarily as a transitional CLI wrapper, but portable Semantic Flow behavior should be expressed through mesh bootstrap, integration, import/source synchronization, ordinary weave/version/generate work, publication validation, and optional host presets.

## Status

This is a current design target, not yet a fully carried fixture ladder.

The immediate pressure comes from branch-published ontology releases, but the rule is broader: the semantic behavior should not depend on whether source bytes are intra-repository, inter-repository, sidecar, whole-repository, or branch-published. Topology affects operational configuration and provenance. It should not create a separate Semantic Flow object model.

## Core Boundary

Publication is a composition of generic operations, not a special branch-only mode.

The portable pieces are:

- `mesh.create` establishes the `SemanticMesh` support surface at a mesh root.
- `integrate` establishes governed payload-artifact surfaces for available bytes without moving the source bytes.
- `import` or source synchronization creates governed local copies when bytes need to move into the mesh or publication tree.
- `weave`, `version`, `validate`, and `generate` record and render the current mesh state.
- publication-host presets add static-host controls such as GitHub Pages `.nojekyll` or `CNAME`.
- publication validation checks that the generated/public tree is publishable under the selected operational profile.
- optional git output handling commits, tags, pushes, or otherwise publishes the resulting tree.

No one of those pieces is inherently tied to `gh-pages`.

## Source Availability Modes

Source bytes can be available to an operation in several ways:

- mesh-local: the source file is already inside the mesh root and can be modeled with a mesh-addressable `LocatedFile`.
- allowed live local: the source file is outside the mesh root but inside an explicitly allowed workspace/source boundary, and current-byte operations may follow `workingLocalRelativePath` under policy.
- imported local: the source file has been copied into the target mesh or publication tree by an explicit import/source-sync operation and is now a governed local copy.
- remote/current access: a `workingAccessUrl` or similar remote locator names current bytes, but runtime access requires explicit network policy.

The selected mode must be visible in configuration, provenance, or source registry facts. It must not be an implicit side effect of `weave`.

## Import And Source Synchronization

When source bytes live outside the target mesh tree and should be copied into the mesh or publication tree, that action is import/source synchronization. It is not integration.

That operation should:

- identify the source repository, ref, commit, path, and digest when those facts are known.
- copy exactly the requested source bytes into the target tree, or fail closed.
- update source/provenance records so later operations can tell where the current bytes came from.
- expose the imported copy as a governed local artifact or working file that later operations can follow.

If the imported copy is meant to become the payload surface for a designator, that payload association should be explicit. The copy step remains import/source synchronization; `integrate` does not become a moving or copying operation.

Same-repository and separate-repository sources should follow the same import/source-sync contract when bytes are copied. The durable facts are repository/ref/path/digest/provenance and the resulting governed local copy, not the host-local checkout path that happened to be used during the run.

## Publication-Host Presets

Static host controls are modular publication-host presets.

The GitHub Pages preset may create or validate:

- `.nojekyll` at the publication root.
- `CNAME` when a custom domain is configured.
- host-specific publish-root constraints needed by GitHub Pages.

Other presets may cover GitLab Pages, Vercel, Netlify, plain static hosting, or local preview. These presets should be optional and selected explicitly, or resolved through a conservative `auto` publication profile. Core `mesh.create` should not create GitHub-specific files as hidden bootstrap behavior, but a user-facing create request may compose mesh creation with a selected or resolved publication-host preset.

`auto` should resolve only from strong signals and should be visible in the operation plan/result. A `meshBase` under `github.io` is a strong GitHub Pages signal. A custom domain alone is not enough to infer GitHub Pages, because the hosting provider is not encoded in that URL. Callers should be able to override inference with an explicit preset or `none`.

The resolved publication profile should be persisted in mesh-carried config when the mesh has `_mesh/_config/config.ttl` or when the create operation needs to create it for that purpose. Use `sfcfg:hasPublicationProfile` on `MeshConfig` and persist concrete values such as `sfcfg:publicationProfile_githubPages` or `sfcfg:publicationProfile_none`. Do not persist `auto` as the mesh profile; `auto` is a request-time resolution mode.

## Publication Validation

Publication validation is generic even when host presets add host-specific checks.

Useful checks include:

- source root and mesh or publication root are distinct when the chosen topology requires distinct roots.
- the mesh root remains inside the configured workspace boundary.
- generated outputs are fresh relative to the current mesh state, or the operation clearly reports stale generated output.
- public RDF and HTML do not leak absolute host-local paths such as `/home/...`, `file:///...`, or checkout-specific temporary paths.
- relative operational paths do not escape their allowed boundary unless that boundary is explicitly configured.
- dirty publication worktrees are reported as warnings by default, because a human may intentionally stage local static files such as a favicon before committing.

Validation should be usable for sidecar meshes, whole-repository meshes, and branch-published meshes. It is not a `gh-pages`-only concern.

## Git Output Handling

Committing the resulting publication tree is an implementation convenience, not part of the core semantic operation.

A local implementation may offer:

- no commit, leaving the worktree changed for inspection.
- a local commit after successful validation.
- a tag, push, deployment trigger, or other release action supplied by an outer CI/CD workflow.

The same optionality should apply to whole-repository, sidecar, and branch-published topologies.

## What This Replaces

The old `prepare gh-pages` shape bundled too many concerns:

- detached or branch-root mesh bootstrap.
- source/publication root validation.
- GitHub Pages static controls.
- import/source synchronization.
- provenance updates.
- stale-output checks.
- path leakage checks.
- optional local commit.

Those responsibilities should be factored into the generic pieces above. If a CLI keeps `prepare gh-pages` while the pieces are being implemented, it should behave as a named wrapper over those pieces rather than as a separate Semantic Flow API concept.

## Invariants

- Branch publication, sidecar publication, whole-repository publication, and separate-repository source publication use the same Semantic Flow artifact model.
- Host-specific controls are selected through publication-host presets, not inferred by core mesh behavior.
- Publication-host presets may be applied at mesh creation time when explicitly requested.
- An `auto` publication profile may infer a preset from strong host signals, but the resolved profile must be reported and overrideable.
- The resolved concrete publication profile is portable mesh config and should be persisted in `MeshConfig`.
- Source copies are explicit imports or source-sync actions and are provenance-bearing.
- `integrate` leaves source bytes where they are.
- `weave` does not fetch, copy, or resynchronize source bytes as a hidden preparation step.
- Public outputs should not contain absolute host-local paths or checkout-specific implementation details.
- A re-run over unchanged source bytes, unchanged config, and unchanged target designators should either be a no-op or reproduce the same semantic current surface.

## Related Specs

- [[sf.spec.2026-04-03-mesh-create]]
- [[sf.spec.2026-04-04-integrate-behavior]]
- [[sf.spec.2026-04-03-weave-behavior]]
- [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]]
