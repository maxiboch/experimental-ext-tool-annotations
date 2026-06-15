# Data-labelling schemes

A **scheme** is a concrete data-labelling or tool-annotation approach that fills
the [`trust-annotations`](../specification/draft/trust-annotations.mdx)
`evidenceRef` slot under an `evidenceRef.type` value. The extension defines a
small, stable wire vocabulary and an open `type` pointer; a scheme defines the
richer, out-of-band record that pointer resolves to.

Schemes are **not** extensions and **not** siblings of the extensions. They are
interchangeable: a deployment can adopt one, several, or none, and can swap them
without changing the extension. Modelling each labelling approach as a scheme keeps
any single academic model out of the wire root, which is the reason FIDES lives
here rather than as a top-level extension.

## Schemes here

| Scheme | `evidenceRef.type` | Status | Source |
| :--- | :--- | :--- | :--- |
| [FIDES information-flow control](./ifc-fides.md) | `ifc.fides.v1` | Draft skeleton | [arXiv:2505.23643](https://arxiv.org/abs/2505.23643) |

## Candidate schemes (not yet drafted)

The open `type` slot is designed to carry the range of models raised in the
[SEP-1913](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913)
review and the surrounding literature. Each is a candidate for its own scheme doc:

| Approach | Likely `type` | Source |
| :--- | :--- | :--- |
| Coarse data classification (level + regulatory scope) | `data-class.v1` | SEP-1913 taxonomy (e.g. `confidential:hipaa` shape) |
| Design-pattern controls (Plan-Then-Execute, Dual LLM, Map-Reduce) | — | [arXiv:2506.08837](https://arxiv.org/abs/2506.08837) |
| ShardGuard | — | cited in SEP-1913 |
| Capability-token constraints (SINT) | — | SEP-1913 review thread |
| Caller/tool cosigning | — | SEP-1913 review thread |
| Sequence-shape audit records | — | SEP-1913 review thread |
| Tool-call attestation (in-toto / OVERT envelopes) | — | [SEP-2787](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2787), in-toto, OVERT |

These are leads, not commitments. A candidate becomes a scheme when someone
drafts it to the bar below; until then the slot simply stays open for it.

## Bar for adding a scheme

A scheme doc should state:

1. **Identity** — the `evidenceRef.type` value it claims, and that it is selected
   by `evidenceRef.type == "<value>"` on a `trust-annotations` annotation.
2. **Payload** — the shape of the record the `evidenceRef` resolves to.
3. **Graceful degradation** — how a client that does not implement the scheme
   ignores it safely (the `trust-annotations` booleans and
   `digest`/`canonicalization` pair remain meaningful regardless).
4. **Producer/consumer** — at least a candidate emitter and consumer, so the
   scheme is validated against real implementations rather than asserted.

`type` values are coordinated through the non-binding `evidenceRef.type` registry
noted in [`trust-annotations`](../specification/draft/trust-annotations.mdx) so
they don't collide.
