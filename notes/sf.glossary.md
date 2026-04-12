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
- a direct mesh-local path string such as `targetMeshPath`
- a direct remote/external URL such as `targetAccessUrl`
- a target `LocatedFile`
- another explicit packaged target if the vocabulary later grows one

This means a target artifact IRI is allowed but not required. If a direct mesh-local path, direct access URL, or direct `LocatedFile` is sufficient, that is a complete and valid resolution target on its own.

Typical consequences:

- if the target is a `DigitalArtifact` in `Current` mode, resolution usually follows that artifact's current `hasWorkingLocatedFile`
- if the target is a `DigitalArtifact` in `Current` mode and that artifact also declares `workingFilePath`, local runtime resolution should follow `workingFilePath` first and treat `hasWorkingLocatedFile` as the semantic `LocatedFile` facet when present
- if the target is a `DigitalArtifact` in `Current` mode and that artifact declares `workingAccessUrl`, a runtime may use that URL only when its operational profile explicitly permits remote current-byte access
- if the target is a `DigitalArtifact` in `Pinned` mode, resolution follows the requested history or state subject to the allowed fallback policy
- if the target is a direct `targetMeshPath`, resolution uses that exact path relative to mesh root with fail-closed behavior, subject to any configured allowed-directory boundary
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

- `workingFilePath` is the operational local-path hook for a `DigitalArtifact`
- `workingAccessUrl` is the operational remote/external current-byte hook for a `DigitalArtifact`
- `hasWorkingLocatedFile` remains the semantic `LocatedFile` hook
- when multiple current-byte locators are present for the same working surface, they should identify the same current bytes; mismatch should fail closed rather than silently picking one
