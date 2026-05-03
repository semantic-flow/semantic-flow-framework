---
id: 1as5uvnhovdarxqglpw3zxn
title: Glossary
desc: ''
updated: 1775934888879
created: 1774245492640
---

## Artifact Resolution

Artifact resolution is the general process of taking a policy-bearing target description and resolving it to the concrete bytes an application should use.

In the current core ontology this pattern is modeled by `ArtifactResolutionTarget`.

An `ArtifactResolutionTarget` may resolve through:

- a target `DigitalArtifact`
- a direct mesh-local path string such as `targetLocalRelativePath`
- a direct remote/external URL such as `targetAccessUrl`
- a target `LocatedFile`
- another explicit packaged target if the vocabulary later grows one

This means a target artifact IRI is allowed but not required. If a direct mesh-local path, direct access URL, or direct `LocatedFile` is sufficient, that is a complete and valid resolution target on its own.

Typical consequences:

- if the target is a `DigitalArtifact` in `Current` mode, resolution usually follows that artifact's current `hasWorkingLocatedFile`
- if the target is a `DigitalArtifact` in `Current` mode and that artifact also declares `workingLocalRelativePath`, local runtime resolution should follow `workingLocalRelativePath` first and treat `hasWorkingLocatedFile` as the semantic `LocatedFile` facet when present
- if the target is a `DigitalArtifact` in `Current` mode and that artifact declares `workingAccessUrl`, a runtime may use that URL only when its operational profile explicitly permits remote current-byte access
- if the target is a `DigitalArtifact` in `Pinned` mode, resolution follows the requested history or state subject to the allowed fallback policy
- if the target is a direct `targetLocalRelativePath`, resolution uses that exact path relative to mesh root with fail-closed behavior, subject to any configured allowed-directory boundary
- if the target is a direct `targetAccessUrl`, resolution may use that URL only when its operational profile explicitly permits remote target access
- if the target is already a direct `LocatedFile`, no artifact-history lookup is needed; resolution can use that file directly
- imported content is not a separate resolution kind once imported; after import it participates in governed artifact resolution like any other managed `DigitalArtifact`

So the important split is not “artifact source vs imported source.” The important split is:

- governed artifact resolution
- direct mesh-path resolution
- direct access-URL resolution
- direct located-file resolution

This is both a runtime term and now also an ontology term through `ArtifactResolutionTarget`.

Related current-byte rule:

- `workingLocalRelativePath` is the operational local-path hook for a `DigitalArtifact`
- `workingAccessUrl` is the operational remote/external current-byte hook for a `DigitalArtifact`
- `hasWorkingLocatedFile` remains the semantic `LocatedFile` hook
- when multiple current-byte locators are present for the same working surface, they should identify the same current bytes; mismatch should fail closed rather than silently picking one

## Sidecar Mesh

A sidecar mesh is a `SemanticMesh` that rides alongside the primary source files in a repository rather than being the repository's main subject.

A common form is a docs-rooted sidecar mesh: the mesh root is a publishable directory such as `docs/`, while working payload files remain elsewhere in the same repo under explicit repo-local path policy. The mesh governs public identifiers, generated resource pages, and historical snapshots without requiring the source tree itself to be organized as the public mesh.

Typical consequences:

- `workingLocalRelativePath` may point from the mesh root to adjacent source files such as `../ontology/example-ontology.ttl`
- repo-local operational policy decides which adjacent paths the runtime may read
- when versioning is enabled, woven historical snapshots should still be materialized inside the mesh by default
- immutable external or remote `LocatedFile` references may be modeled separately, but they are not the default replacement for mesh-owned historical snapshots
