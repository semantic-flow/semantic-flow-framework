---
id: p5vx8gm2q19h4e7w0t6n3ab
title: 2026 05 18 Publication Source Binding
desc: ''
updated: 1779113220000
created: 1779113220000
---

## Purpose

This note captures the intended Semantic Flow behavior boundary for publication setup, source binding, explicit import, update/refresh, validation, and host-specific publication controls.

It is a behavior spec for dissolving the old idea that a dedicated `prepare gh-pages` operation owns branch-published setup. There should be no durable `prepare gh-pages` API concept and no compatibility wrapper in the portable model. Publication setup should be expressed through mesh bootstrap, integration, default `weave` orchestration or narrower version/validate/generate operations, optional host presets, and explicit import only when a working file is intentionally copied into the mesh. Later locator or policy changes should be handled through explicit update/refresh operations; ordinary floating working-source publication follows the already-bound working locator.

## Status

This is a current design target, not yet a fully carried fixture ladder.

The immediate pressure comes from branch-published ontology releases, but the rule is broader: the semantic behavior should not depend on whether source bytes are intra-repository, inter-repository, sidecar, whole-repository, or branch-published. Topology affects operational configuration and provenance. It should not create a separate Semantic Flow object model.

## Core Boundary

Publication is a composition of generic operations, not a special branch-only mode.

The portable pieces are:

- `mesh.create` establishes the `SemanticMesh` support surface at a mesh root.
- `integrate` establishes governed payload-artifact surfaces by linking to available source bytes without moving those bytes.
- `import` copies a working file into the mesh or publication tree when the copy itself should become the governed local working file.
- `weave` is the default orchestration surface for already governed targets: by default it runs the configured versioning, validation, and generation phases. Its versioning phase may append historical states; generation can also render governed artifacts that are intentionally unversioned.
- `version` is the narrower surface for explicitly appending versioned payload states.
- `validate` is the narrower surface for reporting mesh or publication problems without recording new state.
- `generate` is the narrower surface for rendering ResourcePages and other generated surfaces from the current mesh state.
- publication-host presets add static-host controls such as GitHub Pages `.nojekyll`.
- publication validation checks concrete publication-readiness concerns under the selected operational profile.
- optional git output handling commits, tags, pushes, or otherwise publishes the resulting tree.

No one of those pieces is inherently tied to `gh-pages`.

## Source Availability Modes

Source bytes can be available to `integrate` in several ways:

- mesh-local: the source file is already inside the mesh root and can be linked directly.
- allowed live local: the source file is outside the mesh root but inside an explicitly allowed workspace/source boundary, and current-byte operations may follow `workingLocalRelativePath` under policy. Portable workspace grants can be mesh-carried; machine-specific separate-checkout grants should be host-local operational config so public mesh facts do not expose absolute checkout paths.
- repository-backed source: the source is identified by repository/ref/path and optionally commit/digest facts. The binding may follow working source bytes, such as a branch/ref that is intentionally followed, or exact source bytes, such as a commit/digest that must not drift.
- imported local: the source file has already been copied into the mesh or publication tree by an explicit import operation, so `integrate` can link to that governed local copy.
- remote/current access: a `workingAccessUrl` or similar remote locator names current bytes, but runtime access requires explicit network policy.

The selected mode must be visible in configuration, provenance, or source registry facts. It must not be an implicit side effect of `weave`.

## Integrate

`integrate` is the ordinary source-binding operation for sidecar meshes, branch-published ontology sites, same-repository publication lanes, and separate-repository source lanes.

`integrate` should:

- identify the source locator and resolution policy.
- identify repository, ref, commit, path, and digest facts only when the caller deliberately supplies repository-backed or exact source-state evidence.
- record whether the binding follows working bytes, asks for the latest settled state, or names exact source bytes.
- create or update the target designator's payload-artifact support surface.
- leave the source file in its source lane.

For floating working sources outside the mesh root, a `KnopSourceRegistry` source binding should use `targetLocalRelativePath` for the approved local operational locator and `artifactResolutionMode_working` for the resolution policy. The single-source binding uses a deterministic internal fragment id such as `payload-source`. It should not persist repository ref, commit, path, content digest, or `expectsContentDigest` facts by default.

For deliberately repository-backed sources, the same source binding may use `hasTargetRepositorySource` for portable repository provenance. Repository provenance may name a mutable ref, but deterministic release workflows should use an explicit exact source-state policy or deliberately supplied commit/digest evidence instead of silently turning every floating working-source integration into a pinned binding. When a runtime observes local bytes during an explicit repository-backed or exact-state integration, recording a computed digest is evidence about the observed bytes; it is not an import, refresh, or fetch by itself.

For the docs-rooted sidecar Fantasy Rules shape, `integrate` links source files such as `../ontology/fantasy-rules-ontology.ttl` from the `docs/` mesh under constrained local-path policy.

For branch-published ontology releases, the source checkout or source repository should still be bound with `integrate`. The branch/publication topology affects locator policy and provenance; it does not turn source binding into import.

## Import

`import` is narrower: it copies a working file into the mesh or publication tree so the copy becomes the governed local working file.

That operation should:

- identify the source repository, ref, commit, path, and digest when those facts are known.
- copy exactly the requested source bytes into the target tree, or fail closed.
- update source/provenance records so later operations can tell where the copied bytes came from.
- expose the imported copy as a governed local artifact or working file that later operations can follow.

If the imported copy is meant to become the payload surface for a designator, that payload association should still be explicit. The copy step remains import; `integrate` is the later source-binding step, not a moving or copying operation.

Same-repository and separate-repository sources should follow the same import contract only when bytes are actually copied. The durable facts are repository/ref/path/digest/provenance and the resulting governed local copy, not the host-local checkout path that happened to be used during the run.

Ordinary import is usually a one-time acquisition boundary for one governed local copy. If the same upstream source changes later, refreshing that already-imported copy is an update/refresh operation rather than another implicit first import. Manifest-driven or batch import can wait until repeated workflows prove that one-at-a-time import is too awkward.

## Publication-Host Presets

Static host controls are modular publication-host presets.

For now, the GitHub Pages preset may create or validate `.nojekyll` at the publication root.

Custom-domain host files are human-owned for now. A user publishing GitHub Pages through a custom domain should explicitly select the GitHub Pages profile; the profile still only manages `.nojekyll`.

Other presets may cover GitLab Pages, Vercel, Netlify, plain static hosting, or local preview. These presets should be optional and selected explicitly, or resolved through a conservative `auto` publication profile. Core `mesh.create` should not create GitHub-specific files as hidden bootstrap behavior, but a user-facing create request may compose mesh creation with a selected or resolved publication-host preset.

`auto` should resolve only from strong signals and should be visible in the operation plan/result. For now, a `meshBase` under `github.io` is the only strong GitHub Pages signal. A custom domain alone is not enough to infer GitHub Pages, because the hosting provider is not encoded in that URL. CI metadata and repository remotes should not participate in auto-inference yet. Callers should be able to override inference with an explicit preset or `none`.

The resolved publication profile should be persisted in mesh-carried config when the mesh has `_mesh/_config/config.ttl` or when the create operation needs to create it for that purpose. Use `sfcfg:hasPublicationProfile` on `MeshConfig` and persist concrete values such as `sfcfg:publicationProfile_githubPages` or `sfcfg:publicationProfile_none`. Do not persist `auto` as the mesh profile; `auto` is a request-time resolution mode.

## Publication Validation

Publication validation is generic even when host presets add host-specific checks.

Validation should use explicit scopes:

- `weave validate mesh`: whole-mesh validation. This should grow over time into the comprehensive mesh integrity check and should include retained publication-readiness checks when a publication surface or profile exists.
- `weave validate publication`: publication-readiness validation only. This may remain as a narrower convenience for release and page-regeneration workflows.

For ordinary weaving, implementations may expose validation options such as `--validate-before` and `--validate-after`. Those options should invoke whole-mesh validation, not a publication-only validation mode.

First-pass publication validation candidates are:

- public RDF and HTML do not leak absolute host-local paths such as `/home/...`, `file:///...`, or checkout-specific temporary paths.
- selected host presets are satisfied; for GitHub Pages this currently means `.nojekyll`.
- dirty publication worktrees warn only when an operation requests an optional local commit.

Generated-output freshness is not a publication validation rule for now. "Stale" depends on generation policy, current renderer/config, and caller expectation. If needed later, freshness should be designed as generation check behavior rather than default publication validation.

Source-root/publication-root boundary checking is deferred until operations expose planned read/write sets or equivalent path-policy hooks. When revisited, it should check actual planned behavior rather than infer from filesystem layout alone.

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
- source binding/integration.
- source copying only for cases that intentionally create a mesh-local working-file copy.
- provenance updates.
- path leakage checks.
- optional local commit.

Those responsibilities should be factored into the generic pieces above. `prepare gh-pages` should be removed rather than preserved as a wrapper.

## Invariants

- Branch publication, sidecar publication, whole-repository publication, and separate-repository source publication use the same Semantic Flow artifact model.
- Host-specific controls are selected through publication-host presets, not inferred by core mesh behavior.
- Publication-host presets may be applied at mesh creation time when explicitly requested.
- An `auto` publication profile may infer a preset from strong host signals, but the resolved profile must be reported and overrideable.
- The resolved concrete publication profile is portable mesh config and should be persisted in `MeshConfig`.
- Sidecar and branch-published ontology source files are bound with `integrate`, not copied with `import`, unless the user explicitly asks to create a mesh-local working-file copy.
- `integrate` leaves source bytes where they are and records working, latest-state, or exact source policy.
- Source copies are explicit imports and are provenance-bearing.
- `weave` does not fetch, copy, import, or refresh source bytes as a hidden preparation step.
- `weave --validate-before` and `weave --validate-after`, if exposed, run whole-mesh validation.
- Public outputs should not contain absolute host-local paths or checkout-specific implementation details.
- A re-run over unchanged source bytes, unchanged config, and unchanged target designators should either be a no-op or reproduce the same semantic current surface.

## Related Specs

- [[sf.spec.2026-04-03-mesh-create]]
- [[sf.spec.2026-04-04-integrate-behavior]]
- [[sf.spec.2026-04-03-weave-behavior]]
- [[sf.spec.2026-04-11-identifier-page-customization-and-root-lifecycle]]
