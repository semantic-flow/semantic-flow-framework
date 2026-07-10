---
id: srqux1lxfldslwu1z7pou5c
title: 2026 07 06 in Memory Support Chatg
desc: ''
updated: 1783397361059
created: 1783397352091
---

## Conceptual verdict

**Use a lower-level “realize artifact state” API. Do not model in-memory state as fake files.**

My verdict is **API change plus a small ontology clarification/addition**, not a large conceptual rewrite.

`hasWorkingLocatedFile` should be treated as **optional sparse authoring support**: a way to say, “this DigitalArtifact currently has a local working file that can be used as a content source.” It should not be the required model for “current content” of every `DigitalArtifact`.

For Stagecraft, the clean distinction is:

```text
DigitalArtifact
  = the authored state payload/package/dataset as an information artifact

ArtifactHistory
  = an ordered version line/branch for that artifact

HistoricalState
  = one exact state of that artifact in that history

ArtifactManifestation
  = a concrete serialization/representation of that state

LocatedFile
  = one file location containing a manifestation

Stagecraft character / checkpoint / weapon / narrative frame
  = referents described or captured by the artifact state, not DigitalArtifacts
```

An in-memory RDF dataset can be the **API input content source** for a `HistoricalState`. It does not need to become a persistent RDF resource merely because it exists in memory. RDF datasets already support default graphs and named graphs as abstract graph structures; concrete documents such as Turtle, TriG, and N-Quads are serialization forms, not the graph identity itself. RDF 1.2 describes datasets as a default graph plus zero or more named graphs, and Turtle as a concrete syntax for an RDF graph, while TriG and N-Quads are dataset-capable syntaxes. ([W3C][1])

So the core move is:

```text
do not invent:
  urn:stagecraft:graph:state-042.ttl

instead use:
  urn:stagecraft:graph:world-state:c7:s0042       # graph/state identity
  artifacts/stagecraft/c7/world-state/s0042.ttl  # located file realization
```

The `.ttl` extension belongs to the `LocatedFile` path, and possibly to manifestation/file metadata. It should not be baked into the identity of the in-memory graph or the `HistoricalState`.

---

## 1. `hasWorkingLocatedFile` should be optional

Yes. In the Semantic Flow model, `hasWorkingLocatedFile` should be documented and validated as **optional authoring support**, not as a required current-content model.

Recommended semantics:

```ttl
sf:hasWorkingLocatedFile
    a owl:ObjectProperty ;
    rdfs:domain sf:DigitalArtifact ;
    rdfs:range sf:LocatedFile ;
    rdfs:comment """
    Optional authoring-support relation identifying a local working file
    that may be used as a content source for versioning or realization.
    Absence of this property does not imply that the artifact lacks current
    content or cannot be versioned.
    """ .
```

Avoid constraints like:

```ttl
sf:DigitalArtifact
    owl:equivalentClass [
        owl:onProperty sf:hasWorkingLocatedFile ;
        owl:minCardinality 1
    ] .
```

That would make the file-based workflow ontologically mandatory, which is exactly the wrong outcome for Stagecraft’s in-memory RDF store.

---

## 2. Non-file current content: ontology concept or API input?

For Stagecraft’s flush path, **keep transient in-memory content outside the persistent RDF ontology** and pass it as an API input.

Do **not** add a core class like:

```ttl
sf:InMemoryWorkingGraph
```

as a sibling of `LocatedFile` or as a subclass of `DigitalArtifact`. “In memory” is a storage condition or implementation role. It is not an essential kind of authored artifact.

A stable named graph IRI may still be legitimate:

```ttl
<urn:stagecraft:graph:world-state:c7:s0042>
```

But that IRI should identify the graph/dataset state, not a file, and should not imply a serialization format.

Add ontology only if you need durable audit records of content sources. If so, model it generically:

```ttl
sf:ContentSource
    a owl:Class ;
    rdfs:comment """
    A transient or durable source from which artifact-state content can be
    read by an API operation. This is an operational support role, not a
    DigitalArtifact kind.
    """ .

sf:RdfDatasetContentSource
    a owl:Class ;
    rdfs:subClassOf sf:ContentSource ;
    rdfs:comment """
    A content source whose supplied value is an RDF dataset or graph.
    May be an in-process object, stream, or serialized dataset.
    """ .

sf:LocatedFileContentSource
    a owl:Class ;
    rdfs:subClassOf sf:ContentSource .
```

But I would not require these for Stagecraft. The cleaner v0.3.x design is API-level content input plus persistent realization metadata after the flush.

---

## 3. API shape

The API should be centered on **realizing a known artifact state**, not on discovering current content from working files.

Recommended primary API:

```ts
export interface RealizeArtifactStateInput {
  artifact: ArtifactDesignator;

  /**
   * Optional if the state object already gives an exact history IRI.
   * Required when artifact has multiple histories and state does not identify one.
   */
  history?: ArtifactHistoryDesignator;

  /**
   * Required for Stagecraft flush-all semantics.
   * This is the HistoricalState already minted/advanced in memory.
   */
  state: HistoricalStateDesignator;

  /**
   * Exact RDF content for the state.
   * Accept DatasetCore, async quad stream, string, Buffer, or Readable stream.
   */
  rdf: RdfContentInput;

  /**
   * Requested on-disk realization.
   */
  realization: RealizationRequest;

  /**
   * Optional links such as "this HistoricalState captures checkpoint X".
   * These are support/provenance assertions about the state.
   */
  captures?: CaptureLink[];

  provenance?: ProvenanceInput;

  validation?: RealizationValidationOptions;

  mesh?: MeshTargetOptions;

  idempotency?: IdempotencyOptions;
}

export type ArtifactDesignator =
  | string
  | { iri: string }
  | { slug: string; base?: string };

export type ArtifactHistoryDesignator =
  | string
  | {
      iri: string;
      expectedArtifact?: string;
      name?: string;
    };

export type HistoricalStateDesignator =
  | string
  | {
      iri: string;
      history?: string;
      ordinal?: number;
      segment?: string;
      previousState?: string;
      alreadyAdvanced?: true;
    };

export interface RdfContentInput {
  source:
    | DatasetCore
    | AsyncIterable<Quad>
    | Iterable<Quad>
    | string
    | Buffer
    | NodeJS.ReadableStream;

  /**
   * Required when source is string, Buffer, or byte stream.
   * Not needed for DatasetCore / Quad stream.
   */
  inputFormat?:
    | "text/turtle"
    | "application/trig"
    | "application/n-quads"
    | "application/ld+json"
    | "application/rdf+xml";

  /**
   * For graph serializations such as Turtle.
   * If omitted and output is Turtle, the planner should require the dataset
   * to contain exactly one selected graph or default graph.
   */
  graph?: string;

  baseIRI?: string;
}

export interface RealizationRequest {
  format:
    | "text/turtle"
    | "application/trig"
    | "application/n-quads"
    | "application/ld+json";

  /**
   * Optional explicit path. If absent, mesh path policy chooses one.
   */
  locatedFile?: {
    localRelativePath?: string;
    mediaType?: string;
  };

  pathPolicy?: "mesh-default" | "artifact-history-ordinal" | "content-addressed";

  /**
   * Important for idempotent flush.
   */
  overwrite?: "never" | "ifSameDigest" | "replaceAndRecordSupersession";
}

export interface CaptureLink {
  predicate?: string; // default sf:captures
  object: string;    // checkpoint/frame/etc.
  role?: string;
}

export interface ProvenanceInput {
  activity?: string;
  generatedAtTime?: string;
  agent?: string;
  wasDerivedFrom?: string[];
  wasInformedBy?: string[];

  /**
   * Example: Stagecraft checkpoint/event/store transaction identifier.
   */
  sourceEvent?: string;
}

export interface RealizationValidationOptions {
  artifactGuard?: boolean;
  requireKnownArtifact?: boolean;
  requireKnownHistory?: boolean;
  requireAlreadyAdvancedState?: boolean;
  rejectHistoryAdvance?: boolean;
  shapes?: string[];
  failOnReferentDigitalArtifactInference?: boolean;
}

export interface MeshTargetOptions {
  root: string;
  inventory?: "update" | "planOnly" | "none";
  knop?: "update" | "planOnly" | "none";
}
```

Recommended function split:

```ts
export async function planRealizeArtifactState(
  input: RealizeArtifactStateInput
): Promise<RealizationPlan>;

export async function applyRealizationPlan(
  plan: RealizationPlan
): Promise<RealizationResult>;

export async function realizeArtifactState(
  input: RealizeArtifactStateInput
): Promise<RealizationResult>;
```

`realizeArtifactState()` can simply be:

```ts
const plan = await planRealizeArtifactState(input);
return applyRealizationPlan(plan);
```

That gives Weave a dry-run/planning phase, which is useful for mesh inventory updates, digest checks, path collision detection, and validation reports.

---

## 4. Should the API advance history or merely realize?

For Stagecraft, the primary API should **merely realize already-advanced in-memory history**.

Do not make the flush path mint a new `HistoricalState`. Stagecraft already has one ordinal space in memory. Disk should be a faithful realization of that state, not a second versioning engine.

Recommended modes:

```ts
type VersioningMode =
  | "realize-existing-state"
  | "advance-and-realize";
```

Default for the lower-level API:

```ts
mode: "realize-existing-state"
```

Behavior:

| Mode                     | Meaning                                                                                                        | Stagecraft use                                  |
| ------------------------ | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `realize-existing-state` | Caller supplies exact `HistoricalState` IRI/ordinal/content. Weave writes manifestations/files/inventory only. | Primary flush path                              |
| `advance-and-realize`    | Weave creates a new `HistoricalState` from supplied content, then realizes it.                                 | Convenience path for file-based `weave version` |
| `plan-only`              | Validate and return planned writes without touching disk.                                                      | Debug/dry-run                                   |

For Stagecraft flush-all semantics, use:

```ts
validation: {
  requireAlreadyAdvancedState: true,
  rejectHistoryAdvance: true
}
```

Conflict policy should be strict:

```text
same state IRI + same digest
  => idempotent success

same state IRI + different digest
  => error by default

same ordinal + different state IRI
  => error

same file path + different digest
  => error unless explicit repair/supersession mode
```

This preserves the rule: **one ordinal space, no reminting, disk is a realization of memory**.

---

## 5. Validation model

Validation should target **the supplied RDF content plus planned support metadata**, not discovered working files.

Recommended validation phases:

```text
1. Parse/normalize supplied RDF content.
2. Validate DigitalArtifact identity and guard.
3. Validate History/State membership and ordinal invariants.
4. Validate RDF payload constraints.
5. Validate planned manifestation/file metadata.
6. Validate mesh/Knop inventory mutations.
7. Compute digests and compare with existing files/manifests.
```

Important validation rules:

```text
artifact IRI must resolve to or be declared as sf:DigitalArtifact

HistoricalState must belong to the selected ArtifactHistory

ArtifactHistory must belong to the selected DigitalArtifact

state ordinal must match the in-memory history ordinal

capture links must not imply that captured referents are DigitalArtifacts

characters, checkpoints, frames, weapons, places, etc. may be described in
the payload, but must not be inferred as sf:DigitalArtifact merely because
they have pages, captures, state records, or named graphs

if output format is Turtle, input must be a single graph or a selected graph

if input is a true multi-graph RDF dataset and graph names must be preserved,
use TriG or N-Quads
```

That last rule matters. Turtle serializes an RDF graph, while TriG and N-Quads are appropriate when the realization must preserve an RDF dataset with named graphs. ([W3C][2])

---

## 6. API output

Return both the plan and the applied result. The caller needs enough detail to reconcile memory, disk, mesh inventory, and Knop inventory.

Suggested result shape:

```ts
export interface RealizationResult {
  status: "applied" | "planned" | "no-op" | "failed";

  artifact: {
    iri: string;
  };

  history: {
    iri: string;
  };

  state: {
    iri: string;
    ordinal?: number;
    segment?: string;
    existedBefore: boolean;
    advancedByThisCall: boolean;
  };

  content: {
    inputKind: "dataset" | "quadStream" | "string" | "buffer" | "stream";
    graph?: string;
    inputFormat?: string;
    outputFormat: string;
    quadCount?: number;
    byteLength?: number;
  };

  manifestations: ManifestationResult[];

  locatedFiles: LocatedFileResult[];

  digests: DigestResult[];

  inventory: {
    mesh?: InventoryMutationResult;
    knop?: InventoryMutationResult;
  };

  validation: {
    conforms: boolean;
    reports: ValidationReport[];
    warnings: string[];
  };

  provenance: {
    activity?: string;
    generatedAtTime?: string;
    captures?: CaptureLink[];
  };

  plan?: RealizationPlan;
}

export interface ManifestationResult {
  iri: string;
  state: string;
  mediaType: string;
  serializationFormat: string;
  created: boolean;
  replaced: boolean;
}

export interface LocatedFileResult {
  iri: string;
  manifestation: string;
  localRelativePath: string;
  absolutePath?: string;
  created: boolean;
  updated: boolean;
  byteLength: number;
}

export interface DigestResult {
  subject: string;
  algorithm: "sha256" | "sha384" | "sha512";
  value: string;
  canonicalization?: string;
}

export interface InventoryMutationResult {
  status: "updated" | "planned" | "none";
  createdEntries: string[];
  updatedEntries: string[];
  unchangedEntries: string[];
}
```

The output should make it explicit whether Weave created a new `HistoricalState` or merely recorded/realized an existing one. For Stagecraft flush, this should always be:

```ts
advancedByThisCall: false
```

---

## 7. Coexistence with current file-based `weave version`

File-based `weave version` should remain, but become a convenience wrapper over the same lower-level planner.

Current flow:

```text
DigitalArtifact
  -> hasWorkingLocatedFile
  -> workingLocalRelativePath
  -> read file
  -> create HistoricalState
  -> write version support
```

Refactored flow:

```text
weave version
  -> resolve DigitalArtifact
  -> resolve optional hasWorkingLocatedFile
  -> read file bytes/RDF
  -> maybe advance ArtifactHistory
  -> call planRealizeArtifactState()
  -> applyRealizationPlan()
```

Stagecraft flow:

```text
Stagecraft store
  -> already has DigitalArtifact + ArtifactHistory + HistoricalState
  -> supplies RDF dataset/quad stream directly
  -> call planRealizeArtifactState()
  -> applyRealizationPlan()
```

So both paths share:

```text
validation
digesting
manifestation planning
LocatedFile planning
mesh inventory update
Knop inventory update
support metadata generation
```

Only the content-source step differs.

Backcompat story:

```text
Existing hasWorkingLocatedFile data remains valid.

Existing weave version commands keep working.

No fake file IRIs are required.

No migration is required for existing file-backed artifacts.

Docs/schema should clarify that hasWorkingLocatedFile has cardinality 0..n.

Code that assumed every DigitalArtifact has a working file should move to
the lower-level content-source abstraction.

ResourcePage generation remains outside the flush/version realization path.
```

---

## Proposed vocabulary additions

Use these only if the equivalent properties do not already exist.

### Core realization properties

```ttl
sf:hasArtifactHistory
    a owl:ObjectProperty ;
    rdfs:domain sf:DigitalArtifact ;
    rdfs:range sf:ArtifactHistory .

sf:historyOf
    a owl:ObjectProperty ;
    rdfs:domain sf:ArtifactHistory ;
    rdfs:range sf:DigitalArtifact .

sf:hasHistoricalState
    a owl:ObjectProperty ;
    rdfs:domain sf:ArtifactHistory ;
    rdfs:range sf:HistoricalState .

sf:stateOfHistory
    a owl:ObjectProperty ;
    rdfs:domain sf:HistoricalState ;
    rdfs:range sf:ArtifactHistory .

sf:stateOrdinal
    a owl:DatatypeProperty ;
    rdfs:domain sf:HistoricalState ;
    rdfs:range xsd:integer .

sf:hasManifestation
    a owl:ObjectProperty ;
    rdfs:domain sf:HistoricalState ;
    rdfs:range sf:ArtifactManifestation .

sf:manifestationOfState
    a owl:ObjectProperty ;
    rdfs:domain sf:ArtifactManifestation ;
    rdfs:range sf:HistoricalState .

sf:hasLocatedFile
    a owl:ObjectProperty ;
    rdfs:domain sf:ArtifactManifestation ;
    rdfs:range sf:LocatedFile .

sf:localRelativePath
    a owl:DatatypeProperty ;
    rdfs:domain sf:LocatedFile ;
    rdfs:range xsd:string .

sf:serializationFormat
    a owl:DatatypeProperty ;
    rdfs:domain sf:ArtifactManifestation ;
    rdfs:range xsd:string .

sf:mediaType
    a owl:DatatypeProperty ;
    rdfs:range xsd:string .
```

### Capture/provenance links

```ttl
sf:captures
    a owl:ObjectProperty ;
    rdfs:domain sf:HistoricalState ;
    rdfs:range rdfs:Resource ;
    rdfs:comment """
    Relates a HistoricalState to a referent, event, checkpoint, frame, or
    other domain thing whose state is captured or represented by the artifact
    state. The object is intentionally not constrained to sf:DigitalArtifact.
    """ .
```

For provenance, reuse PROV-O where possible. PROV-O is intended for representing provenance information about entities, activities, and agents, and can be specialized for domain-specific provenance. ([W3C][3])

Example:

```ttl
sf:RealizationActivity
    a owl:Class ;
    rdfs:subClassOf prov:Activity .

sf:realizedBy
    a owl:ObjectProperty ;
    rdfs:domain sf:ArtifactManifestation ;
    rdfs:range sf:RealizationActivity .
```

### Digest metadata

```ttl
sf:Digest
    a owl:Class .

sf:hasByteDigest
    a owl:ObjectProperty ;
    rdfs:range sf:Digest .

sf:digestAlgorithm
    a owl:DatatypeProperty ;
    rdfs:domain sf:Digest ;
    rdfs:range xsd:string .

sf:digestValue
    a owl:DatatypeProperty ;
    rdfs:domain sf:Digest ;
    rdfs:range xsd:string .
```

---

## OntoClean sanity check

| Term                                                  | OntoClean reading                                                                                                                                             | Verdict                                                      |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `sf:DigitalArtifact`                                  | Rigid type for authored information artifacts/payloads/packages. Identity comes from authorship and intended artifact identity, not from the thing described. | Good                                                         |
| `sf:ArtifactHistory`                                  | Dependent on a `DigitalArtifact`; identity is a version line/branch.                                                                                          | Good                                                         |
| `sf:HistoricalState`                                  | Dependent state snapshot in a history; not the domain referent itself.                                                                                        | Good                                                         |
| `sf:ArtifactManifestation`                            | Realization of a state in a concrete syntax/representation. Multiple manifestations may realize one state.                                                    | Good                                                         |
| `sf:LocatedFile`                                      | Storage/location bearer for manifestation bytes. File extension/path belong here.                                                                             | Good                                                         |
| `sf:InMemoryWorkingGraph` as core class               | Anti-rigid implementation/storage role; not an essential artifact kind.                                                                                       | Avoid                                                        |
| `sf:captures`                                         | Relates a state to arbitrary referents. Range must not be `sf:DigitalArtifact`.                                                                               | Good                                                         |
| `stage:Character`, `stage:Weapon`, `stage:Checkpoint` | Domain referents described by the payload.                                                                                                                    | Must not become `DigitalArtifact` because pages/states exist |

The important guard is:

```text
A page about X, a graph describing X, or a HistoricalState that captures X
does not make X a DigitalArtifact.

Only the authored dataset/package/state payload is the DigitalArtifact.
```

---

## Small Stagecraft example

### In-memory RDF graph identity

No file extension:

```ttl
<urn:stagecraft:graph:world-state:campaign-7:s0042>
```

### DigitalArtifact and already-advanced HistoricalState

```ttl
@prefix sf:    <https://semantic-flow.example/ns#> .
@prefix stage: <https://stagecraft.example/ns#> .
@prefix prov:  <http://www.w3.org/ns/prov#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .

<urn:stagecraft:artifact:world-state:campaign-7>
    a sf:DigitalArtifact ;
    sf:hasArtifactHistory
        <urn:stagecraft:artifact:world-state:campaign-7/history/main> .

<urn:stagecraft:artifact:world-state:campaign-7/history/main>
    a sf:ArtifactHistory ;
    sf:historyOf
        <urn:stagecraft:artifact:world-state:campaign-7> ;
    sf:hasHistoricalState
        <urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042> .

<urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042>
    a sf:HistoricalState ;
    sf:stateOfHistory
        <urn:stagecraft:artifact:world-state:campaign-7/history/main> ;
    sf:stateOrdinal 42 ;
    sf:captures
        <urn:stagecraft:checkpoint:campaign-7:after-ambush> ;
    prov:generatedAtTime
        "2026-07-07T18:30:00Z"^^xsd:dateTime .
```

The checkpoint is not a `DigitalArtifact`:

```ttl
<urn:stagecraft:checkpoint:campaign-7:after-ambush>
    a stage:Checkpoint ;
    stage:label "After the north-road ambush" .
```

### In-memory world-state payload

This is the graph content Stagecraft has in memory:

```ttl
@prefix stage: <https://stagecraft.example/ns#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .

<urn:stagecraft:world:campaign-7>
    a stage:World ;
    stage:currentFrame
        <urn:stagecraft:frame:campaign-7:north-road-after-ambush> ;
    stage:currentCheckpoint
        <urn:stagecraft:checkpoint:campaign-7:after-ambush> .

<urn:stagecraft:character:aria>
    a stage:Character ;
    stage:displayName "Aria" ;
    stage:health 78 ;
    stage:location <urn:stagecraft:place:north-road> ;
    stage:hasWeapon <urn:stagecraft:weapon:shadeglass-dagger> .

<urn:stagecraft:weapon:shadeglass-dagger>
    a stage:Weapon ;
    stage:displayName "Shadeglass dagger" ;
    stage:condition "chipped" .

<urn:stagecraft:frame:campaign-7:north-road-after-ambush>
    a stage:NarrativeFrame ;
    stage:summary "The ambush has ended; two riders fled north." .
```

### Realization call

```ts
import { realizeArtifactState } from "@semantic-flow/weave";

const result = await realizeArtifactState({
  artifact: {
    iri: "urn:stagecraft:artifact:world-state:campaign-7"
  },

  history: {
    iri: "urn:stagecraft:artifact:world-state:campaign-7/history/main"
  },

  state: {
    iri: "urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042",
    history: "urn:stagecraft:artifact:world-state:campaign-7/history/main",
    ordinal: 42,
    alreadyAdvanced: true
  },

  rdf: {
    source: stagecraftStore.datasetForState(42),
    graph: "urn:stagecraft:graph:world-state:campaign-7:s0042"
  },

  realization: {
    format: "text/turtle",
    locatedFile: {
      localRelativePath:
        "artifacts/stagecraft/campaign-7/world-state/history/s0042.ttl",
      mediaType: "text/turtle"
    },
    overwrite: "ifSameDigest"
  },

  captures: [
    {
      predicate: "https://semantic-flow.example/ns#captures",
      object: "urn:stagecraft:checkpoint:campaign-7:after-ambush"
    }
  ],

  provenance: {
    generatedAtTime: "2026-07-07T18:30:00Z",
    sourceEvent: "urn:stagecraft:event:campaign-7:tick-000042"
  },

  validation: {
    artifactGuard: true,
    requireKnownArtifact: true,
    requireKnownHistory: true,
    requireAlreadyAdvancedState: true,
    rejectHistoryAdvance: true,
    failOnReferentDigitalArtifactInference: true
  },

  mesh: {
    root: "./mesh",
    inventory: "update",
    knop: "update"
  }
});
```

### Support metadata written by realization

```ttl
@prefix sf:    <https://semantic-flow.example/ns#> .
@prefix prov:  <http://www.w3.org/ns/prov#> .
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .

<urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042>
    sf:hasManifestation
        <urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042/manifestation/turtle> .

<urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042/manifestation/turtle>
    a sf:ArtifactManifestation ;
    sf:manifestationOfState
        <urn:stagecraft:artifact:world-state:campaign-7/history/main/state/000042> ;
    sf:serializationFormat "text/turtle" ;
    sf:mediaType "text/turtle" ;
    sf:serializesGraph
        <urn:stagecraft:graph:world-state:campaign-7:s0042> ;
    sf:hasLocatedFile
        <urn:stagecraft:file:artifacts/stagecraft/campaign-7/world-state/history/s0042.ttl> ;
    sf:hasByteDigest [
        a sf:Digest ;
        sf:digestAlgorithm "sha256" ;
        sf:digestValue "b1946ac92492d2347c6235b4d2611184..."
    ] .

<urn:stagecraft:file:artifacts/stagecraft/campaign-7/world-state/history/s0042.ttl>
    a sf:LocatedFile ;
    sf:localRelativePath
        "artifacts/stagecraft/campaign-7/world-state/history/s0042.ttl" ;
    sf:mediaType "text/turtle" ;
    sf:byteLength 612 .
```

The graph/state identity remains:

```ttl
<urn:stagecraft:graph:world-state:campaign-7:s0042>
```

The file realization is:

```text
artifacts/stagecraft/campaign-7/world-state/history/s0042.ttl
```

Those are intentionally different things.

---

## Recommended implementation direction for Weave npm >=0.3.x

Implement this layering:

```text
Layer 1: Content readers
  - from working file
  - from RDF DatasetCore
  - from quad stream
  - from string/Buffer/Readable

Layer 2: State resolver
  - resolve artifact
  - resolve history
  - resolve exact HistoricalState
  - optionally advance history only in convenience APIs

Layer 3: Realization planner
  - choose serialization
  - choose path
  - generate Manifestation and LocatedFile IRIs
  - compute planned support metadata
  - compute digests
  - prepare inventory mutations

Layer 4: Validator
  - payload RDF
  - artifact/history/state invariants
  - DigitalArtifact guard
  - serialization compatibility
  - path/digest conflicts

Layer 5: Applier
  - write files
  - write support metadata
  - update mesh inventory
  - update Knop inventory

Convenience wrapper:
  weave version
    = file content reader
    + optional history advance
    + same realization planner
```

The Stagecraft flush path should call Layer 3+ directly with already-minted states.

Final policy recommendation:

```text
hasWorkingLocatedFile is optional.

In-memory RDF content is an API input, not a fake LocatedFile.

HistoricalState identity is minted by the in-memory Stagecraft store.

Flush realizes that state to Manifestations and LocatedFiles.

File extensions live on LocatedFiles/paths, not graph or state IRIs.

weave generate remains outside the flush path.

Referents described or captured by the state remain referents, not DigitalArtifacts.
```

I did not find useful public material for the specific internal Semantic Flow/Weave vocabulary names in your prompt; public search results for “Semantic Flow” point to unrelated projects or papers, so the analysis above is based on the Stagecraft context and the vocabulary semantics you provided.

[1]: https://www.w3.org/TR/rdf12-concepts/?utm_source=chatgpt.com "RDF 1.2 Concepts and Abstract Data Model"
[2]: https://www.w3.org/TR/rdf12-turtle/?utm_source=chatgpt.com "RDF 1.2 Turtle"
[3]: https://www.w3.org/TR/prov-o/?utm_source=chatgpt.com "PROV-O: The PROV Ontology"
