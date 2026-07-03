---
id: 1as5uvnhovdarxqglpw3zxn
title: Glossary
desc: ''
updated: 1779152129069
created: 1774245492640
---

## Primary Identifier

A primary identifier is an IRI minted for the resource a publisher primarily wants to name for external use: an ontology, ontology term, dataset, document, person, organization, concept, or other referent.

Primary identifiers are the IRIs a Semantic Mesh is *for*. They are the public handles users cite, dereference, put in RDF, and expect to remain meaningful over time.

A primary identifier is a role in a mesh context, not an eternal metaphysical class. A resource that is a support artifact in one mesh may become a primary subject in another mesh or in a later modeling pass.

## Support Identifier

A support identifier is an IRI minted for a mesh-managed resource that helps maintain, publish, inspect, or explain another identifier.

Common support identifiers name resources such as:

- `Knop`
- `ReferenceCatalog`
- `KnopSourceRegistry`
- `KnopInventory`
- `KnopMetadata`
- `ResourcePageDefinition`
- `ResourcePage`
- `ArtifactHistory`
- `HistoricalState`
- `ArtifactManifestation`
- `LocatedFile`

Support identifiers are still first-class IRIs. Their role is instrumental: they make primary identifiers dereferenceable, citable at useful specificity levels, and supported by inspectable RDF evidence.

## Sense-Finding

Sense-finding is the activity of working out what an identifier is being used to mean from available evidence.

Semantic Flow does not freeze identifier meaning. Identifier senses can evolve through use, documentation, revision, deprecation, and community practice. Semantic Flow instead tries to keep the support trail inspectable: pages, references, source records, histories, states, manifestations, files, digests, and observations.

## Semantic Mesh

A Semantic Mesh is a governed identifier surface plus the RDF-described support structure needed to publish, inspect, and maintain that surface.

A mesh usually has:

- a canonical base IRI for minting mesh identifiers
- one or more primary identifiers
- Knop-managed support for those identifiers
- generated `ResourcePage`s for dereferenceability
- support artifacts such as inventories, metadata, reference catalogs, source registries, and page definitions
- optional payload `DigitalArtifact`s and their histories, states, manifestations, and located files

A mesh can be materialized as ordinary files and hosted as a static site when the host returns `index.html` for directory-like IRIs.

## Knop

A Knop is the mesh-managed support object for one primary identifier or otherwise governed designator in a Semantic Mesh.

The Knop is not the referent of the identifier. It is the support anchor that can own the identifier's metadata, inventory, reference catalog, source registry, page definition, generated ResourcePage, and optional payload artifact.

The Knop pattern lets a mesh support very different referents with the same publication machinery: ontology terms, datasets, RDF documents, concepts, people, and support artifacts can all have explicit support without being collapsed into the same kind of resource.

## DigitalArtifact

A `DigitalArtifact` is a resource with byte-grounded artifact identity across time and representation.

Examples include ontology documents, RDF datasets, SHACL shapes files, generated HTML pages, Markdown source files, PDFs, bundles, and other digital objects where it is useful to distinguish the continuing artifact from its histories, states, representations, and retrievable byte locations.

Not every primary identifier identifies a `DigitalArtifact`. A term IRI may identify a class or property; a person IRI may identify a person. Those resources can still have support artifacts and ResourcePages.

## DigitalArtifact Facets

The main `DigitalArtifact` facet chain is:

`DigitalArtifact -> ArtifactHistory -> HistoricalState -> ArtifactManifestation -> LocatedFile`

The terms mean:

- `DigitalArtifact`: the continuing artifact-level resource
- `ArtifactHistory`: a named internal lineage for the artifact, such as release, draft, archive, source-import, or curation history
- `HistoricalState`: a settled state within one artifact history; it specializes DCAT versioning via `hasHistoricalState` / `dcat:hasVersion`
- `ArtifactManifestation`: a representation or byte-pattern identity for an artifact or state; it specializes `dcat:Distribution`
- `LocatedFile`: a retrievable location for bytes, such as a mesh path, URL, or hosted file

The distinction between `ArtifactManifestation` and `LocatedFile` is important. A manifestation identifies the representation or intended byte pattern; a located file identifies where bytes can be retrieved. The same manifestation may be available from several locations, and a location may need digest or observation evidence before an application trusts it as the intended manifestation.

The full chain is used when every layer carries useful evidence. Sparse modeling may skip layers through explicit shortcut properties when the omitted layers would be ceremonial.

## ArtifactHistory

An `ArtifactHistory` is a named lineage inside a `DigitalArtifact`.

One artifact can have several histories. Common examples are release history, draft history, archived predecessor history, editorial curation history, and source-import history.

`defaultArtifactHistory` identifies the history that ordinary write/version operations should extend when no history is explicitly selected. `currentArtifactHistory` remains available as compatibility or publisher-endorsed lineage vocabulary, but "current" is contextual and should not be treated as the core default-write rule.

`latestHistoricalState` points from an `ArtifactHistory` to its latest settled state within that lineage. It should not be confused with a global "current version" of the artifact.

## ResourcePage

A `ResourcePage` is a `LocatedFile` whose role is to present another resource.

On static hosts, a directory-like primary identifier can be made dereferenceable when the host returns that directory's `index.html`. The returned HTML file can identify itself as a `ResourcePage` and assert that it is the page linked from the requested resource with `hasResourcePage`.

Ordinary `LocatedFile`s usually do not need ResourcePages of their own because their identifiers already identify concrete retrievable content. A ResourcePage can itself be modeled as a `DigitalArtifact` when the page's own history, states, or bytes matter.

## Artifact Resolution

Artifact resolution is the general process of taking a policy-bearing target description and resolving it to the concrete bytes an application should use.

In the current core ontology this pattern is modeled by `ArtifactResolutionSpec`.

An `ArtifactResolutionSpec` may resolve through:

- a target `DigitalArtifact`
- a direct mesh-local path string such as `targetLocalRelativePath`
- a direct remote/external URL such as `targetAccessUrl`
- a target `LocatedFile`
- another explicit packaged target if the vocabulary later grows one

This means a target artifact IRI is allowed but not required. If a direct mesh-local path, direct access URL, or direct `LocatedFile` is sufficient, that is a complete and valid resolution target on its own.

Typical consequences:

- if the target is a `DigitalArtifact` in `Working` mode, resolution usually follows mutable working bytes through `workingLocalRelativePath`, allowed `workingAccessUrl`, or `hasWorkingLocatedFile`
- if the target is a `DigitalArtifact` in `Working` mode and that artifact also declares `workingLocalRelativePath`, local runtime resolution should follow `workingLocalRelativePath` first and treat `hasWorkingLocatedFile` as the semantic `LocatedFile` facet when present
- if the target is a `DigitalArtifact` in `Working` mode and that artifact declares `workingAccessUrl`, a runtime may use that URL only when its operational profile explicitly permits remote working-byte access
- if the target is a `DigitalArtifact` in `LatestState` mode, resolution follows the latest settled `HistoricalState`; a requested `ArtifactHistory` bounds that search to that history, and a no-history request should use `defaultArtifactHistory` or fail closed rather than guessing across all histories
- if the target declares an exact `HistoricalState`, `LocatedFile`, manifestation/distribution, commit, or digest, the target is exact by default without needing an additional resolution mode
- if the target is a direct `targetLocalRelativePath`, resolution uses that exact path relative to mesh root with fail-closed behavior, subject to any configured allowed-directory boundary
- if the target is a direct `targetAccessUrl`, resolution may use that URL only when its operational profile explicitly permits remote target access
- if the target is already a direct `LocatedFile`, no artifact-history lookup is needed; resolution can use that file directly
- imported content is not a separate resolution kind once imported; after import it participates in governed artifact resolution like any other managed `DigitalArtifact`

Use `Working` for mutable working/source bytes and `LatestState` for the latest settled historical state. Exact target coordinates fix identity by themselves.

So the important split is not “artifact source vs imported source.” The important split is:

- governed artifact resolution
- direct mesh-path resolution
- direct access-URL resolution
- direct located-file resolution

This is both a runtime term and now also an ontology term through `ArtifactResolutionSpec`.

Related current-byte rule:

- `workingLocalRelativePath` is the operational local-path hook for a `DigitalArtifact`
- `workingAccessUrl` is the operational remote/external current-byte hook for a `DigitalArtifact`
- `hasWorkingLocatedFile` remains the semantic `LocatedFile` hook
- when multiple current-byte locators are present for the same working surface, they should identify the same current bytes; mismatch should fail closed rather than silently picking one

## Source Provenance

Source provenance records where bytes used by a Semantic Flow operation came from. This is usually carried in a Knop-owned `_sources` registry, materialized as `_knop/_sources/sources.ttl`.

Two common source-provenance cases are distinct:

- payload source provenance records how a governed artifact was materialized from repository/ref/path/digest-shaped source bytes
- extracted-terms provenance records which RDF artifact provided the source facts used to ground an extracted Knop-managed resource

Extraction provenance is not the same thing as a curated `ReferenceLink`. A reference says something intentionally curated about a resource. Extraction provenance says which source bytes justified creating or rendering that extracted identifier. A future operation may derive a curated reference from extraction provenance, but the two records have different meanings.

## Integrate And Import

`integrate` links available source bytes to a target designator and payload artifact while leaving those source bytes where they are. The source may be mesh-local, adjacent under explicit local-path policy, or repository-backed with working, latest-state, or exact source policy.

`import` copies a working file into the mesh or publication tree so the copy becomes governed local working content. Import is not the normal operation for sidecar or branch-published ontology release sources; those should generally be integrated from their source lane.

## Sidecar Mesh

A sidecar mesh is a `SemanticMesh` that rides alongside the primary source files in a repository rather than being the repository's main subject.

A common form is a docs-rooted sidecar mesh: the mesh root is a publishable directory such as `docs/`, while working payload files remain elsewhere in the same repo under explicit repo-local path policy. The mesh governs public identifiers, generated resource pages, and historical snapshots without requiring the source tree itself to be organized as the public mesh.

Typical consequences:

- `workingLocalRelativePath` may point from the mesh root to adjacent source files such as `../ontology/example-ontology.ttl`
- repo-local operational policy decides which adjacent paths the runtime may read
- when versioning is enabled, woven historical snapshots should still be materialized inside the mesh by default
- immutable external or remote `LocatedFile` references may be modeled separately, but they are not the default replacement for mesh-owned historical snapshots
