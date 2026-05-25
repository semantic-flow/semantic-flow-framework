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

Authored config is RDF that participates in behavior. `sfcfg:ApplicationConfig`, `sfcfg:MeshConfig`, `sfcfg:KnopConfig`, reusable config artifacts, and operational config are authored or supplied inputs. `sfcfg:ResolvedConfig` and `sfcfg:ConfigResolutionRecord` are derived runtime or diagnostic outputs; they are not trusted authored config sources and should not be required for ordinary mesh portability.

Config has two broad kinds of content:

- **Scoped settings** are direct facts about a config scope. They are appropriate for simple singleton or additive settings that are not target-selective policy, such as a mesh publication profile or the local relationship between a mesh root and workspace root.
- **Layered policies** are behavior choices that can vary by scope, target selector, policy family, and operation. They should be represented with explicit policy definitions, bindings, targets, and precedence semantics rather than direct context-free policy assertions on a config resource.

`sfcfg:ConfigResolutionConfig` is meta-config for the resolver itself: layer ordering, merge behavior, trust caps, reference policy, unknown-term policy, and cache/diagnostic behavior. It may be supplied by an application and may be narrowed by trusted or portable config where allowed, but portable config must not use resolver config to expand its own trust boundary.

## Config Layers

Implementations should treat config sources as ordered layers. The exact profile can be described by `sfcfg:ConfigResolutionConfig`, but the ordinary Semantic Flow precedence shape is:

1. built-in and application defaults
2. trusted machine-local and workspace operational config
3. mesh-local config
4. mesh-inheritable config projected into a descendant scope
5. Knop-inherited config from ancestors
6. reusable config at the attachment point that referenced it
7. Knop-inheritable config when a policy explicitly applies it to the declaring Knop
8. Knop-local config
9. explicit operation or command overrides

Higher-precedence layers win over lower-precedence layers for the same policy family and target. Explicit operation overrides are volatile request inputs; they must not be silently persisted as mesh or Knop config.

Knop-inheritable config is primarily an outbound offer to descendant scopes. If a config inheritance policy explicitly says it also applies to the declaring Knop, it should still be overrideable by Knop-local config. Mesh-inheritable and Knop-inheritable config should be accepted, blocked, or propagated according to the active config inheritance policy.

## Scoped Settings

Scoped settings should remain direct config terms when target selection and policy composition do not add value.

Examples:

- `sfcfg:hasPublicationProfile` records a mesh's publication-host profile, such as GitHub Pages. It may cause publication-surface files such as `.nojekyll` to be created, but it must not imply history tracking, ResourcePage generation, or ResourcePage presentation policy.
- `sfcfg:workspaceRootRelativeToMeshRoot` describes a portable local layout relationship. It must not grant arbitrary host access by itself.

Scoped settings still need validation. Unsupported values, duplicate singleton values, unsafe path values, or contradictory scoped settings must fail closed.

## Layered Policies

Layered policies should use explicit bindings. A binding associates one reusable policy definition with one explicit policy target selector within the layer and scope of the config source that declares the binding.

A policy binding has:

- a policy definition
- a policy family
- a policy value or profile
- an explicit target selector
- the declaring config source and layer
- an optional priority for same-layer conflict resolution

Policy definitions should be reusable. A definition can be bound to multiple targets or scopes without copying the policy value. Binding metadata, such as priority or attachment source, belongs on the binding rather than inside the reusable policy definition.

Policy families initially include:

- history tracking
- ResourcePage generation
- ResourcePage presentation defaults
- history, state, and manifestation naming

Additional policy families may be added when they need the same layer, target, and conflict-resolution behavior. Publication profiles are not policy bindings in this sense; they are scoped mesh settings.

## Policy Targets

Policy targets must be explicit. Omitted target fields do not mean "any".

Artifact policy targets should distinguish at least:

- any governed artifact in the current config scope
- artifacts with a specific `sfcfg:ArtifactRole`
- one exact artifact

"Any artifact role" is not itself an artifact role. It is an explicit selector shape or selector value. A config that intends to match every artifact role must say so explicitly.

Incomplete or ambiguous target selectors must fail validation. A target selector with no explicit matching shape must not be broadened by interpretation.

ResourcePage policy targets may also need page-kind, target-class, artifact-role, exact-page, or authored/generated-page selectors. These should follow the same rule: wildcard behavior must be explicit, not implied by missing fields.

## Policy Resolution

For a given policy query, resolution proceeds in this order:

1. Select policy bindings in applicable config scopes and layers.
2. Prefer higher-precedence layers over lower-precedence layers.
3. Within the winning layer, prefer more specific target selectors over broader target selectors.
4. Within the same layer, policy family, and effective selector specificity, prefer higher `sfcfg:policyPriority`.
5. Treat omitted `sfcfg:policyPriority` as `0`.
6. If two still-applicable bindings tie and produce incompatible values, reject the resolved config as ambiguous.

Selector specificity is structural. For artifact policies, exact artifact is more specific than artifact role, and artifact role is more specific than any artifact. `sfcfg:policyPriority` is not the mechanism that makes a role-specific policy beat an any-artifact policy; priority only resolves conflicts at the same effective layer and selector specificity.

Higher layers should be able to establish a new baseline for a policy family. For example, a mesh-local history policy for any artifact in that mesh overrides lower application-level role defaults, while a same-layer mesh-local role-specific history policy can still override that mesh baseline for a role such as runtime metadata.

## ResourcePage Behavior

ResourcePage generation policy controls whether ResourcePages are generated, suppressed, deferred, or produced on request for matching targets.

ResourcePage presentation policy controls generated page chrome, panel selection, templates, stylesheets, and related presentation choices. Presentation defaults may be supplied at application, mesh, inherited, reusable, or Knop-local scopes like other layered policies.

Mesh-wide ResourcePage presentation must be able to select generated panels without adding `sfcfg:hasGeneratedResourcePagePanelSelection` to every authored `sflo:ResourcePageDefinition`. Page-local selections remain useful as local composition or override hooks, but they are not the only way to choose generated panels.

Implementations may initially restrict portable config to known supported ResourcePage presentation profiles. Unsupported profile IRIs or unsupported panel/template/style references must fail closed rather than falling back silently.

Publication profile and ResourcePage presentation are separate concerns. A GitHub Pages publication profile may create `.nojekyll`; it must not imply an all-panels ResourcePage layout or Semantic Flow metadata panel inclusion.

## Command Overrides

Operation request fields and command-line flags are explicit operator intent for one operation. They participate as the highest ordinary precedence layer unless a stricter invariant or operation-request override policy rejects the request.

Command overrides should be represented internally as policy bindings or scoped settings in a command-override layer, but they must not be persisted into portable mesh config unless the user explicitly invokes a config-editing operation.

## Validation

Config resolution must fail closed for:

- malformed RDF
- unknown config terms when the active unknown-term policy rejects them
- unsupported controlled-vocabulary values
- duplicate singleton settings in the same effective scope
- incomplete or ambiguous policy targets
- policy-family conflicts that cannot be resolved by layer, selector specificity, or priority
- unsafe config references or cycles under the active resolver policy
- portable config that attempts to expand host trust or resolver authority beyond the active operational boundary

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

The policy index should support ordinary resolution and explanation. Implementations should be able to report participating config sources, command overrides, selected policy values by family, and rejected or ignored config sources without serializing the entire effective config by default.

## Initial Weave Slice

The first Weave implementation slice should honor `_mesh/_config/config.ttl` as mesh-local config for existing-mesh commands, compile it over application defaults, and then apply command overrides.

That slice should establish the policy-binding vocabulary needed for history tracking, ResourcePage generation, ResourcePage presentation defaults, and naming policy. It may leave remote config retrieval, arbitrary reusable config references, persisted `sfcfg:ResolvedConfig`, and full resolver diagnostic materialization for later.
