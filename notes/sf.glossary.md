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
- a target `LocatedFile`
- another explicit packaged target if the vocabulary later grows one

This means a target artifact IRI is allowed but not required. If a direct `LocatedFile` is sufficient, that is a complete and valid resolution target on its own.

Typical consequences:

- if the target is a `DigitalArtifact` in `Current` mode, resolution usually follows that artifact's current `hasWorkingLocatedFile`
- if the target is a `DigitalArtifact` in `Pinned` mode, resolution follows the requested history or state subject to the allowed fallback policy
- if the target is already a direct `LocatedFile`, no artifact-history lookup is needed; resolution can use that file directly
- direct workspace-relative file targets are a specialized case of `LocatedFile` resolution, not a separate third system beside artifact resolution and import
- imported content is not a separate resolution kind once imported; after import it participates in governed artifact resolution like any other managed `DigitalArtifact`

So the important split is not “artifact source vs imported source.” The important split is:

- governed artifact resolution
- direct located-file resolution

This is both a runtime term and now also an ontology term through `ArtifactResolutionTarget`.
