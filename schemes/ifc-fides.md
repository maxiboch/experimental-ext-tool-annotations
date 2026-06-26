---
title: FIDES information-flow control (data-labelling scheme)
---

> ⚠️ **Experimental scheme skeleton.** This is one **data-labelling scheme** that
> fills the [`trust-annotations`](../specification/draft/trust-annotations.mdx)
> `evidenceRef` slot via `type: "ifc.fides.v1"`. It is **not** an MCP extension
> and **not** a sibling of the extensions — it is one interchangeable way to
> populate the evidence a `trust-annotations` annotation carries. See
> [Why a scheme](#why-a-scheme-not-an-extension).

**Scheme** — fills `trust-annotations`'s `evidenceRef` slot, selected by
`evidenceRef.type == "ifc.fides.v1"`. Not an extension.

## Abstract

This scheme defines `ifc.fides.v1`, an entry for the `trust-annotations`
`evidenceRef` slot that carries an **information-flow-control label** — integrity
plus confidentiality — following the FIDES model
([arXiv:2505.23643](https://arxiv.org/abs/2505.23643)). A host that implements
deterministic information-flow control can consume these labels to decide whether
a tool call is permitted, without baking the IFC model into the core protocol or
into the `trust-annotations` wire surface.

## Why a scheme, not an extension

Information-flow control is one enforcement model among several that reviewers of
SEP-1913 raised — capability tokens, caller/tool cosigning, and sequence-shape
audit records were all put forward, and the literature adds more (ShardGuard, the
"Design Patterns for Securing LLM Agents" controls). A top-level extension
(`io.modelcontextprotocol/ifc`) would make the FIDES integrity × confidentiality
lattice the namespace root and silently foreclose those other models.

As a `type` value behind the open-ended `evidenceRef` slot, the FIDES label is
first-class while the slot stays free for every other scheme. One reviewer's
framing captured it: IFC "fits relatively well *if you use annotations*" — an
endorsement of IFC as a scheme behind the slot, not as the wire root. The
[`schemes/`](./README.md) folder is where these interchangeable approaches live.

## Motivation

The motivating case is the one raised in the
[2026-05-28 IG meeting](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/2820):
**a model often cannot tell whether a repository is public or private**, and
lacking that signal it may push private content to a public destination. An IFC
label lets the host track confidentiality (who may read this data) and
integrity (is this data trusted) as context accumulates across tool calls, and
deny or prompt before a flow violates policy.

A public MCP server is the natural emitter. [`github-mcp-server`](https://github.com/github/github-mcp-server)
returns repository data whose confidentiality follows from repository visibility
and collaborator sets — the same public/private signal — but does **not** emit
IFC labels today. Closing that emitter gap is the concrete proof point for this
scheme: a host-side consumer of the label shape already exists, so the missing
half is a server willing to emit it, classifying each resource it returns (see
[per-resource classification](#reference-implementation)).

## Specification

### Scheme identity

This scheme is selected by `evidenceRef.type == "ifc.fides.v1"` on a
`trust-annotations` annotation. A client that does not implement IFC MUST be
able to ignore it safely (the surrounding `sensitive` / `untrusted` booleans and
the `digest`/`canonicalization` pair remain meaningful).

### Label payload

The record referenced by the `evidenceRef` (and, for low-friction adoption, MAY
be inlined by deployments that accept the wire cost) has the shape:

```jsonc
{
  "integrity": "trusted",            // "trusted" | "untrusted"  (FIDES §4.1 two-level lattice)
  "confidentiality": "public"        // "public" | "private"
}
```

| Field | Meaning |
| :--- | :--- |
| `integrity` | Two-level integrity lattice (`trusted` ⊑ `untrusted`): trusted data may flow to untrusted sinks, not vice versa. |
| `confidentiality` | `"public"` = world-readable; `"private"` = an opaque marker meaning "restricted to some reader set". The concrete reader set is resolved host-side at policy-decision time (see [Reader-set resolution](#reader-set-resolution)). |

> **Confidentiality is `public` / `private` only — never a reader list on the
> wire.** Emitting concrete reader identities (e.g. logins) is out of scope: user
> identity is not uniform across servers using different auth methods, the
> identities are themselves access-restricted data, and a single resource can
> have hundreds of readers. The opaque marker keeps the wire shape stable and the
> sensitive resolution host-side.

### Label semantics

The load-bearing distinction is between the wire and the host: **wire markers are
advisory hints; reader-set semantics are host-resolved.** The asymmetry between
the two joins below follows from that one cut.

- **Join on accumulation.** As a session ingests labeled results, the context
  label is the *join* of what it has seen: integrity degrades toward
  `untrusted`, confidentiality narrows toward the smallest permitted reader set.
  The two joins differ in *where* they can be computed, and the difference is
  principled rather than incidental:
  - **Integrity join is total and wire-computable.** The integrity lattice is
    small and closed (`trusted ⊑ untrusted`), so `untrusted` dominates and the
    join needs nothing beyond the wire values.
  - **Confidentiality join is partial and host-resolved.** Reader sets are open
    and host-knowledge-dependent. `public` is the one wire-computable case,
    because its reader set is universal (`⊤`): `public ⊔ anything = public`.
    `private ⊔ private`, by contrast, is the *intersection* of two reader sets
    that the opaque markers don't carry, so it is **not** computable from the
    wire — see [Reader-set resolution](#reader-set-resolution).
- **Policy check before egress.** Before a write/egress tool call, the host
  checks whether the current context label may flow to the call's target. When
  a label is absent, the host falls back to its default (trusted-action)
  policy rather than assuming the worst — labels are an *additive* signal.

> The normative integrity/confidentiality lattice definitions follow the FIDES
> paper, §4.1 and §4.3. This scheme references the model rather than restating
> the proofs.

### Reader-set resolution

`"private"` is intentionally opaque on the wire — and that opaqueness is a
property of the security model, not a limitation of the scheme. A reader set is not
transmissible without policy context, so the wire shape correctly declines to
carry it. Two distinct `"private"` markers (e.g. file contents from two different
private repositories) are **not equal**, and their confidentiality join is **not**
the same `"private"` token: data derived from both may flow only to principals who
can read *both* sources — the intersection of their reader sets. The opaque marker
cannot express this intersection, so a host that needs to make a precise
cross-source flow decision MUST resolve each `"private"` marker to a concrete
reader set before joining.

Resolution is a host-side concern, performed at policy-decision time:

1. The host maps each contributing `"private"` label back to its source (e.g.
   via the `evidenceRef.ref` locator, or its own record of which tool result
   carried the label).
2. The host queries the originating system for the current reader set (e.g. a
   repository collaborators lookup) using its own credentials.
3. The host computes the flow decision over the resolved sets (intersection for
   a join of multiple private sources) and then discards them.

**When resolution is unavailable** — the `ref` is absent, the label is
digest-only, or the originating system is unreachable at decision time — the host
MUST NOT treat two opaque labels as equal, and MUST NOT treat `"private"` as
`"public"`. It denies, prompts, or applies its configured fail-closed policy. Two
`"private"` labels are equal only once resolution proves their sources are; until
a source is established, unknown or mixed provenance classifies as `"private"`,
never defaulted to `"public"` from a repository-level shortcut.

The resolved reader set is a decision-time read performed under the host's own
credentials. It is not a durable grant: a host SHOULD NOT cache it as one or
serialize it back into annotations or evidence unless a deployment explicitly opts
in. This keeps the wire free of user identities while still letting the host make a
precise decision when it holds source provenance plus its own credentials.

### Relationship to `trust-annotations`

The `ifc.fides.v1` label never appears without a host `trust-annotations`
annotation carrying the `evidenceRef`. The booleans are the universally-actionable
signal; the IFC label is the precise, host-checkable evidence behind them.

## Reference implementation

- **Consumer:** a host-side IFC engine that parses the `{integrity,
  confidentiality}` label, maintains a context label across tool results, and
  applies a flow policy before egress operations already exists in practice.
  (Linked once a public reference is available.)
- **Emitter (gap / proof point):** [`github-mcp-server`](https://github.com/github/github-mcp-server)
  is the candidate — it already knows repository visibility and collaborator
  sets, which are the confidentiality inputs. Repository visibility is only a
  *default* hint, not the whole story: a public repository can serve
  sub-resources that are **not** world-readable (draft security advisories,
  draft releases, the collaborator roster itself, authenticated-user fields), so
  a correct emitter MUST classify **per resource returned**, not per repository.
  That makes the emitter a non-trivial proof point rather than a one-line
  `repo.private` read.

## Open questions

- Should the label be inlinable on `_meta.ifc` directly for low-friction
  adoption, or always behind `evidenceRef` for schema minimalism? (Lean:
  permit both; `evidenceRef` is canonical, inline is a convenience.)
- How does GitHub Enterprise `internal` repository visibility map onto the
  `public` / `private` confidentiality model? (Audience is the whole org,
  strictly broader than collaborators — likely classified `private` and resolved
  host-side, or falls back to default policy.)
- Registry coordination with other evidence schemes (e.g. SEP-2787) so
  `evidenceRef.type` values don't collide.

## Changelog

| Date       | Change                                                       |
| ---------- | ------------------------------------------------------------ |
| 2026-06-10 | Initial draft skeleton. Reframed from a top-level `ifc` extension to a `trust-annotations` `evidenceRef` entry (`ifc.fides.v1`). |
| 2026-06-15 | Confidentiality limited to `public` / `private` on the wire (dropped reader-list); added Reader-set resolution section; emitter classifies per resource, not per repository. (Review: @JoannaaKL.) |
| 2026-06-16 | Lead the semantics with the wire-hint / host-resolved split; state the integrity-total vs confidentiality-partial asymmetry as principled (`public` = `⊤` is wire-computable, `private ⊔ private` is not); add fail-closed handling when resolution is unavailable and a no-durable-grant rule for resolved sets. (Review: @Rul1an.) |
| 2026-06-16 | Moved out of `specification/draft/` into `schemes/`; reframed from a sibling extension draft into a data-labelling scheme — one filler of the `trust-annotations` `evidenceRef` slot, not a peer extension. |
