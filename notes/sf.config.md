---
id: t2gytmqeb26qajgc4puw3k7
title: Config
desc: ''
updated: 1777828535352
created: 1777828535352
---

This note records the current Semantic Flow configuration model. It is intentionally narrower than a full application-settings story: it focuses on RDF configuration that affects how a mesh is interpreted or how a local implementation is allowed to resolve local resources.

## Purpose

Config should make resolution policy explicit without making portable mesh data depend on one host machine. A mesh should be able to carry config that is meaningful wherever the mesh is checked out, while machine-local or operator-specific policy stays outside the mesh and is supplied by the runtime environment.

## Config Layers

Semantic Flow distinguishes config source, config attachment, and config layer:

- a `sfcfg:ConfigArtifact` gives config stable identity, history, dereferenceability, and reuse
- a `sfcfg:ConfigSource` describes how to resolve config bytes or a config artifact
- `sfcfg:hasConfig` and `sfcfg:hasConfigSource` attach local config to recognized config-bearing subjects such as `sflo:SemanticMesh` and `sflo:Knop`
- `sfcfg:hasInheritableConfig` and `sfcfg:hasInheritableConfigSource` attach Knop-authored subtree defaults for descendant Knops

There is intentionally no reusable-config layer. If a mesh-level local attachment references a config artifact stored under a Knop, that content participates as mesh-level config because the mesh attachment used it. If a Knop-local attachment references the same artifact, the same content participates as Knop-local config. Source location and reusable identity are provenance, not authority.

For ordinary resolution, application defaults are lower precedence than mesh-local config, which is lower than config projected into a Knop scope from ancestor Knop inheritable offers, which is lower than Knop-local config, with explicit operation or command overrides above ordinary authored layers. A `sfcfg:ConfigResolutionConfig` can name the exact profile. `sfcfg:configLayerRole_knopInheritable` is best read as an authored outbound offer; it does not apply to the declaring Knop unless the same config is also attached through local Knop config. When consumed by a descendant, that projected content participates as inherited config, not as the original Knop's local layer.

The main practical rule is that mesh-carried config can describe constrained mesh/workspace-relative behavior, while broader host access belongs to implementation-specific runtime settings outside the portable Semantic Flow config vocabulary.

## Mesh Config

`MeshConfig` should live under `_mesh/_config/` as a mesh support artifact. The default file name for the first implementation slice is:

- `_mesh/_config/config.ttl`

For a docs-rooted sidecar mesh, that means:

- `docs/_mesh/_config/config.ttl`

The file should be treated like other support RDF: discoverable from mesh inventory, versionable by Weave, and suitable for conformance fixtures. It should not live at a repository root name such as `.sf-repo-access.ttl`, because meshes do not always occupy an entire repository and do not always live in repositories.

Knop config should conventionally live at:

- `<knop>/_knop/_config/config.ttl`

The filesystem location bootstraps discovery for that scope. Within the graph, `hasConfig` / `hasConfigSource` mean local config for the attachment subject, while `hasInheritableConfig` / `hasInheritableConfigSource` mean Knop-authored defaults for descendant Knop scopes. Mesh config has no separate mesh-inheritable modality; mesh-level policy can cover Knop-governed targets through ordinary selectors and scope.

A newly created sidecar mesh should include a `MeshConfig` support artifact when the caller specifies a workspace root that differs from the mesh root. For example, `weave mesh create --workspace . --mesh-root docs ...` should create `docs/_mesh/_config/config.ttl`. Whole-root meshes do not need a config artifact merely to record that the workspace root and mesh root are the same, but they may still need one when portable mesh config such as a publication profile is selected or resolved.

The sidecar `MeshConfig` should record the portable workspace relationship with `sfcfg:workspaceRootRelativeToMeshRoot`. For a `docs/` sidecar, that value is `"../"`. This is a relative relationship from the mesh root to the containing workspace root; it is not an absolute host path and it does not grant access by itself.

`mesh.create` should not write initial `sfcfg:hasMeshWorkspacePathRule` entries into the config file. Constrained adjacent-source rules belong to the integration step that introduces the corresponding sidecar artifact, so the mesh grants access only when there is a concrete artifact need.

## Local Path Policy

Mesh workspace path policy uses explicit rule objects rather than implicit trust in a repository layout. A mesh-carried rule identifies:

- `sfcfg:workspacePathPrefix`: a POSIX-style relative path prefix interpreted from the mesh root
- one or more locator kinds through `sfcfg:appliesToLocalPathLocatorKind`, such as `sfcfg:localPathLocatorKind_workingLocalRelativePath` or `sfcfg:localPathLocatorKind_targetLocalRelativePath`

For sidecar meshes, a mesh-carried `sfcfg:MeshWorkspacePathRule` may allow known adjacent source directories such as `../ontology/`, `../shacl/`, or `../examples/`. Add these rules when ontology, SHACL, or example artifacts are integrated and need `workingLocalRelativePath` access to those adjacent source trees. That is still constrained workspace access, not arbitrary host traversal. A mesh-carried rule should fail closed if it attempts to grant broad `..` traversal or host-absolute access, and the resolved prefix must stay inside the workspace described by `sfcfg:workspaceRootRelativeToMeshRoot` and the active runtime context.

Broader machine-local policy such as user-home or absolute-path allowances belongs to implementation/runtime settings, not portable Semantic Flow config. For example, Weave stores per-user host-local source grants in its own settings vocabulary rather than in `sfcfg:MeshConfig`.

## `mesh.create` Local Surface

The thin API contract for `mesh.create` should remain semantic and implementation-neutral, as described in [[sf.api]]. Local filesystem placement belongs to implementation surfaces such as the Weave CLI.

For Weave-style local execution, use this conceptual split:

- `--workspace`: the checked-out project or local workspace root
- `--mesh-root`: the mesh root path, defaulting to `.`
- `--mesh-base`: the public base IRI for identifiers in that mesh

The CLI resolves both `--workspace` and `--mesh-root` from the command working directory, then requires the resolved mesh root to stay inside the resolved workspace root.

For a docs-rooted GitHub Pages sidecar, the command shape should be:

```sh
weave mesh create --workspace . --mesh-root docs --mesh-base https://semantic-flow.github.io/mesh-sidecar-fantasy-rules/
```

This is different from treating `docs` as the workspace. `docs` is the mesh root on disk; the workspace is the containing project, which may also contain adjacent authored source files such as `ontology/`, `shacl/`, and `examples/`.

The default `--mesh-root .` keeps whole-repo meshes such as Alice Bio ergonomic. A sidecar mesh opts into a non-root mesh location.

## Publication Profiles

Publication-host behavior should be represented as a mesh-carried publication profile rather than as hidden mesh bootstrap behavior.

Use `sfcfg:hasPublicationProfile` on `MeshConfig` to persist the resolved concrete publication profile for the mesh. Initial profile values include:

- `sfcfg:publicationProfile_none`
- `sfcfg:publicationProfile_githubPages`

Request-time `auto` is not a persisted publication profile. It is a resolution mode. If `mesh.create` receives `publicationProfile=auto`, the operation may infer a concrete profile from strong signals such as a `meshBase` under `github.io`, but it should record the resolved concrete value in `MeshConfig` and report it in the operation result. Arbitrary custom domains, CI metadata, and repository remotes should not imply GitHub Pages by themselves.

The GitHub Pages profile may create or validate `.nojekyll` during the same user-facing create operation. That file is a static host control, not an RDF support artifact, and should not be listed in mesh inventory. Custom-domain host files are human-owned for now.
