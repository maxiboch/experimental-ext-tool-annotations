# Tool Annotations Interest Group — Experimental Extensions

> ⚠️ **Experimental** — This repository is an incubation space for the
> [Tool Annotations Interest Group](https://modelcontextprotocol.io/community/tool-annotations/charter).
> Contents are exploratory drafts intended to feed future Extensions Track SEPs
> ([SEP-2133](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2133)).
> They do not represent official MCP specifications or recommendations.

**Charter:** [modelcontextprotocol.io/community/tool-annotations/charter](https://modelcontextprotocol.io/community/tool-annotations/charter)
**Discord:** [#tool-annotations-ig](https://discord.com/channels/1358869848138059966/1482836798517543073)
**Open work:** [Pull requests](https://github.com/modelcontextprotocol/experimental-ext-tool-annotations/pulls)

## Why split the work?

[SEP-1913 (Trust and Sensitivity Annotations)](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913)
bundles four concerns that have proven hard to evaluate as a single unit: a
client-facing trust taxonomy, action-security metadata for tool I/O, a
malicious-activity signal, and propagation rules across session boundaries.

The sponsor, [@localden](https://github.com/localden), asked the central
question directly in review: the SEP "adds a few schema modifications and a
thorny array-or-scalar polymorphism on enum fields. If the taxonomy turns out
to be wrong, I worry that we can't remove it or easily modify it. Can we do a
potential narrower first cut?" The subsequent design discussion converged on a
layered answer: a small, stable annotation surface on the wire, with richer
evidence kept out-of-band and referenced by a bounded pointer.

This repo follows that steer. Each concern becomes a **separate experimental
extension** with its own [reverse-DNS identifier](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/seps/2133-extensions.md#definition),
its own reference implementation, and its own path to a future Extensions
Track SEP. Drafts can graduate independently — directly addressing the "narrower
first cut" ask without throwing away the combinatoric value of the full set.

See [docs/decisions.md](docs/decisions.md) for the decision record and
[docs/trust-model.md](docs/trust-model.md) for the shared enforcement model.

## Extensions

| Identifier | Status | What it specifies | Reference implementation(s) |
| :--- | :--- | :--- | :--- |
| [`io.modelcontextprotocol/trust-annotations`](specification/draft/trust-annotations.mdx) | Draft skeleton | **Primary extension.** A small, scheme-agnostic client-facing data-classification vocabulary (`sensitive`, `untrusted`) on result `_meta`, plus an optional `evidenceRef` pointer slot that carries richer payloads out-of-band. | Python SDK: [`kapil8811/mcp-trust-annotations`](https://github.com/kapil8811/mcp-trust-annotations) (138-test suite, healthcare demo, LLM usability study). |
| [`io.modelcontextprotocol/action-metadata`](specification/draft/action-metadata.mdx) | Draft skeleton | `inputMetadata` / `returnMetadata` / outcome classifiers (incl. `requires_review`) on `ToolAnnotations`, describing where inputs go, where outputs originate, and what real-world effects a tool can cause. | Carries forward [SEP-2061 (Action Security Metadata)](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2061) by [@rreichel3](https://github.com/rreichel3); reference impl per that proposal (`read_drafts` / `list_inbox` / `send_email`). |
| [`io.modelcontextprotocol/ifc-fides`](specification/draft/ifc-fides.mdx) | Draft skeleton | A **profile** of the `trust-annotations` `evidenceRef` slot: `type: "ifc.fides.v1"` carrying an integrity + confidentiality label for deterministic information-flow control, following the FIDES paper ([arXiv:2505.23643](https://arxiv.org/abs/2505.23643)). | Emitter candidate: [`github-mcp-server`](https://github.com/github/github-mcp-server) (does not emit IFC labels today — closing that gap is the proof point). |

### Why FIDES is a profile, not a top-level extension

Information-flow control is modelled as a profile rather than the namespace
root because IFC (an integrity × confidentiality lattice) is one enforcement
model among several that reviewers raised — capability tokens, caller/tool
cosigning, and sequence-shape audit records. A top-level `ifc/` root would bake
one academic model into the namespace and foreclose the others. As one reviewer
put it, IFC "fits relatively well if you use annotations" — an endorsement of
IFC *as a profile*, not as the wire root. As a `type` value under
`trust-annotations`'s open-ended `evidenceRef` slot, the FIDES work stays
first-class while every other model can occupy the same slot.

## Relationship to SEP-1913

SEP-1913 remains the canonical place to discuss the overall problem framing.
This repository develops the schema-bearing parts of that proposal as
independently shippable extensions. When an extension here is ready to graduate,
an Extensions Track SEP can reference this repo as the prior art and the working
implementation that SEP-2133 [requires](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/seps/2133-extensions.md#creation).

For the full per-SEP plan — what happens to SEP-1913, SEP-2061, SEP-1862 and
others, and the SEP-2127 refactor precedent — see
[docs/sep-disposition.md](docs/sep-disposition.md).

**Out of scope for these extensions** (see [docs/open-questions.md](docs/open-questions.md)):

- **`maliciousActivityHint`** — reviewer concerns are structural (it fires at
  `tools/resolve` before execution can produce evidence; a boolean is the wrong
  granularity for client UX; clients won't trust server self-attestation). If it
  returns, it is per-`ContentBlock` with spans, on a different clock. It stays
  on the SEP-1913 umbrella rather than in an extension here.
- **Propagation rules** — sensitivity escalation across session boundaries, and
  the sequence-shape gap (an annotation surface for "this was call N in a
  flagged sequence") remain open. Likely a future extension once the taxonomy
  and `evidenceRef` shape are stable.

## Repository layout

This repo mirrors the structure of official extension repositories such as
[`ext-auth`](https://github.com/modelcontextprotocol/ext-auth):

```
specification/draft/<extension-name>.mdx   # one spec per extension
docs/                                       # decision log, open questions, related work
MAINTAINERS.md                              # IG facilitators
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Substantive design discussion happens
on PRs against the relevant `specification/draft/*.mdx` file, in the IG
Discord, and (for cross-extension concerns) on [SEP-1913](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913).
