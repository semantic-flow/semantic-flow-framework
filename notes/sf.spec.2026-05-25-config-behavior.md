---
id: xrxqsvnhufq31p91p3ke4yw
title: 2026 05 25 Config Behavior
desc: ''
updated: 1779744117134
created: 1779744111582
---

## Purpose

This spec defines the portable behavior model for Semantic Flow configuration. It is meant to align the config ontology, Weave's effective-config runtime, and future implementations before implementation-specific CLI/API surfaces are added.

Semantic Flow config must let applications provide conservative defaults, meshes carry durable behavior choices, Knops inherit and override local behavior, and explicit operation requests override ordinary defaults for a single run. Config resolution must be deterministic, explainable, and fail closed when authored config is malformed, unsupported, or contradictory.

## Conceptual Model

Authored config is RDF that participates in behavior. `sfcfg:ApplicationConfig`, `sfcfg:MeshConfig`, `sfcfg:KnopConfig`, and config artifacts are authored or supplied inputs. Implementation- or service-specific runtime authority may also cap resolution, but it is not modeled as a Semantic Flow config level here. `sfcfg:ResolvedConfig` and `sfcfg:ConfigResolutionRecord` are derived runtime or diagnostic outputs; they are not trusted authored config sources and should not be required for ordinary mesh portability.

All authored config is potentially reusable. Reuse is not a separate kind of config and does not by itself define precedence. The effective scope and layer of config come from the attachment point that uses the config, such as application defaults, mesh-local config, mesh-inheritable config, Knop-local config, Knop-inheritable config, or a command override.

A config source may describe or constrain its intended use, but it must not expand the authority of the attachment that loaded it. For example, a config artifact stored under a Knop may participate at mesh scope only when a mesh-level attachment explicitly references it; Knop-local config must not promote itself to mesh-wide scope.

Config has two broad kinds of content:

- **Scoped settings** are direct facts about a config scope. They are appropriate for simple singleton or additive settings that are not target-selective policy, such as a mesh publication profile or the local relationship between a mesh root and workspace root.
- **Layered policies** are behavior choices that can vary by scope, target selector, policy slot/value predicate, and operation. They should be represented with explicit policy definitions, bindings, targets, and precedence semantics rather than direct context-free policy assertions on a config resource.

`sfcfg:ConfigResolutionConfig` is meta-config for the resolver itself: layer ordering, merge behavior, trust caps, reference policy, unknown-term policy, and cache/diagnostic behavior. It may be supplied by an application and may be narrowed by trusted or portable config where allowed, but portable config must not use resolver config to expand its own trust boundary. Under the default resolver model, portable config may request stricter resolver behavior but must not request looser behavior than the active application and runtime resolver policy. Implementations that do not yet support portable resolver narrowing should reject or ignore those portable resolver-config declarations according to the active unknown-term and resolver-policy rules.

## Config Layers

Implementations should treat config sources as ordered layers. The exact profile can be described by `sfcfg:ConfigResolutionConfig`, but the ordinary Semantic Flow precedence shape is:

1. built-in and application defaults
2. mesh-local config
3. mesh-inheritable config projected into a descendant scope
4. Knop-inherited config from ancestors
5. Knop-local config
6. explicit operation or command overrides

Implementation- or service-specific runtime authority may reject, cap, or narrow resolution before or during these layers, but it is not a portable Semantic Flow config layer in this spec.

Higher-precedence layers win over lower-precedence layers for the same policy slot where their targets overlap. An upper-layer binding masks lower-layer bindings only for resources its selector actually covers. Explicit operation overrides are volatile request inputs; they must not be silently persisted as mesh or Knop config.

Referenced config is not a universal layer. It is config content used through an attachment point. Its declarations participate at that attachment point's scope and layer, subject to trust policy and merge rules. Resolution records may still identify referenced config sources for provenance and debugging.

Knop-inheritable config is an outbound offer to descendant scopes, not local config for the declaring Knop. If the same policy should apply to the declaring Knop and its descendants, it should be attached both as Knop-local config and as Knop-inheritable config, or represented by explicitly referenced config used from both attachment points. Mesh-inheritable and Knop-inheritable config should be accepted, blocked, or propagated according to the active config inheritance policy.

## Scoped Settings

Scoped settings should remain direct config terms when target selection and policy composition do not add value.

Examples:

- `sfcfg:hasPublicationProfile` records a mesh's publication-host profile, such as GitHub Pages. It may cause publication-surface files such as `.nojekyll` to be created, but it must not imply history tracking, ResourcePage generation, or ResourcePage presentation policy.
- `sfcfg:workspaceRootRelativeToMeshRoot` describes a portable local layout relationship. It must not grant arbitrary host access by itself and is not a policy binding.

Scoped settings still need validation. Unsupported values, duplicate singleton values, unsafe path values, or contradictory scoped settings must fail closed.

## Layered Policies

Layered policies should use explicit bindings. A binding associates one policy definition with one explicit policy target selector within the layer and scope of the config source that declares the binding.

A policy binding has:

- a policy definition
- one or more policy slots/value predicates set by that definition
- an explicit target selector
- the declaring config source and layer
- an optional priority for same-layer conflict resolution

Policy definitions should be reusable. A definition can be bound to multiple targets or scopes without copying the policy value. Binding metadata, such as priority or attachment source, belongs on the binding rather than inside the policy definition.

A policy slot is the supported value predicate whose value participates in resolution, such as `sfcfg:hasHistoryTrackingPolicy` or `sfcfg:hasResourcePageGenerationPolicy`. Conflict detection and merge behavior are evaluated per policy slot, not by a broad policy-family label. If a policy definition sets multiple supported slots, each slot is resolved independently after the applicable binding and target are selected. An implementation may identify the slot by inspecting supported value predicates directly or by using an explicit slot/property marker, but `PolicyFamily` is not part of the normative resolution model.

Initial policy slots include:

- `sfcfg:hasHistoryTrackingPolicy`
- `sfcfg:hasResourcePageGenerationPolicy`
- the ResourcePage presentation default slot, such as `sfcfg:hasDefaultResourcePagePresentationConfig` or its successor if the ontology gives config-scope presentation defaults a narrower policy predicate

Additional policy slots may be added when they need the same layer, target, and conflict-resolution behavior. History, state, and manifestation naming may remain ordinary layered scoped settings unless they become target-selective. Publication profiles are not policy bindings in this sense; they are scoped mesh settings. Broad grouping metadata can be added later for UI, logging, or documentation, but it must not drive normative conflict detection.

## Policy Targets

Policy targets must be explicit. Omitted target fields do not mean "any".

Artifact policy targets should distinguish at least:

- any governed artifact in the current config scope
- artifacts with a specific `sfcfg:ArtifactRole`
- one exact artifact

A governed artifact is a Semantic Flow-managed `sflo:DigitalArtifact` governed by the current config scope through the mesh or Knop's support, inventory, payload, reference, page-definition, or config structure. Mesh support artifacts such as mesh metadata, mesh inventory, and mesh config artifacts are governed by their mesh. Knop support artifacts and payload artifacts are governed by their Knop and by the containing mesh. A merely referenced external artifact is not governed by a scope just because config in that scope points to it.

"Any artifact role" is not itself an artifact role. It is an explicit selector shape or selector value. A config that intends to match every artifact role must say so explicitly.

Incomplete or ambiguous target selectors must fail validation. A target selector with no explicit matching shape must not be broadened by interpretation.

ResourcePage policy targets may also need page-kind, target-class, artifact-role, exact-page, or authored/generated-page selectors. These should follow the same rule: wildcard behavior must be explicit, not implied by missing fields. ResourcePage selector specificity should be defined by containment when possible: selector A is more specific than selector B when every target matched by A is also matched by B. If two ResourcePage selectors overlap but neither contains the other, they are incomparable; priority may resolve same-layer conflicts, otherwise conflicting values must fail closed.

## Policy Resolution

Under the default/application-level `sfcfg:ConfigResolutionConfig`, resolution for a given policy query proceeds in this order:

1. Select policy bindings in applicable config scopes and layers whose selectors cover the queried target.
2. Prefer higher-precedence layers over lower-precedence layers for the queried target.
3. Within the winning layer, prefer more specific target selectors over broader target selectors.
4. Within the same layer, policy slot, and effective selector specificity, prefer higher `sfcfg:policyPriority`.
5. Treat omitted `sfcfg:policyPriority` as `0`.
6. If two still-applicable bindings tie and produce incompatible values, reject the resolved config as ambiguous.

Selector specificity is structural. For artifact policies, exact artifact is more specific than artifact role, and artifact role is more specific than any artifact. `sfcfg:policyPriority` is not the mechanism that makes a role-specific policy beat an any-artifact policy; priority only resolves conflicts at the same effective layer and selector specificity.

Higher layers should be able to establish a new baseline for a policy slot. For example, a mesh-local history policy for any artifact in that mesh overrides lower application-level role defaults, while a same-layer mesh-local role-specific history policy can still override that mesh baseline for a role such as runtime metadata.

## ResourcePage Behavior

ResourcePage generation policy controls whether ResourcePages are generated, suppressed, deferred, or produced on request for matching targets.

ResourcePage presentation policy controls generated page chrome, panel selection, templates, stylesheets, and related presentation choices. Presentation defaults may be supplied at application, mesh, inherited, referenced, or Knop-local scopes like other layered policies.

Mesh-wide ResourcePage presentation must be able to select generated panels without adding `sfcfg:hasGeneratedResourcePagePanelSelection` to every authored `sflo:ResourcePageDefinition`. Page-local selections remain useful as local composition or override hooks, but they are not the only way to choose generated panels.

Implementations may initially restrict portable config to known supported ResourcePage presentation profiles. Unsupported profile IRIs or unsupported panel/template/style references must fail closed rather than falling back silently.

Publication profile and ResourcePage presentation are separate concerns. A GitHub Pages publication profile may create `.nojekyll`; it must not imply an all-panels ResourcePage layout or Semantic Flow metadata panel inclusion.

## Command Overrides

Operation request fields and command-line flags are explicit operator intent for one operation. They participate as the highest ordinary precedence layer unless a stricter invariant or the active `sfcfg:OperationRequestOverridePolicy` rejects the request.

Command overrides should be represented internally as policy bindings or scoped settings in a command-override layer, but they must not be persisted into portable mesh config unless the user explicitly invokes a config-editing operation.

## Validation

Config resolution must fail closed for:

- malformed RDF
- unknown config terms when the active unknown-term policy rejects them
- unsupported controlled-vocabulary values
- duplicate singleton settings in the same effective scope
- unsafe scoped-setting values, such as local paths that escape the active trust boundary
- incomplete or ambiguous policy targets
- policy-slot conflicts that cannot be resolved by layer, selector specificity, or priority
- unsafe config references or cycles under the active resolver policy
- portable config that attempts to expand host trust or resolver authority beyond the active runtime boundary

Failing closed means the operation stops before applying behavior that depends on the invalid config.

## Runtime Shape

Implementations should compile RDF config into an internal structure that is efficient to query. A useful runtime shape has separate sections for scoped settings, layered policy indexes, and resolver/meta-policy:

```ts
interface CompiledConfig {
  settings: ScopedConfigSettings;
  policies: PolicyIndex;
  resolution: ConfigResolutionRuntimeProfile;
}
```

The policy index should support ordinary resolution and explanation. Implementations should be able to report participating config sources, command overrides, selected policy values by slot, and rejected or ignored config sources without serializing the entire effective config by default. `sfcfg:ConfigResolutionRecord` is the RDF vocabulary for materializing this explanation when an implementation deliberately persists or exports resolver diagnostics.

## Initial Weave Slice

The first Weave implementation slice should honor `_mesh/_config/config.ttl` as mesh-local config for existing-mesh commands, compile it over application defaults, and then apply command overrides.

That slice should establish the policy-binding vocabulary needed for history tracking, ResourcePage generation, and ResourcePage presentation defaults. It may leave remote config retrieval, arbitrary referenced config resolution, inheritable-config projection, persisted `sfcfg:ResolvedConfig`, and full resolver diagnostic materialization for later.
