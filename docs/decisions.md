# Decision log

Append-only record of design decisions for the Tool Annotations IG's trust /
privacy extension work. Newest at the bottom.

## 2026-06-10 — Split SEP-1913 into independent extensions

**Decision.** Split the schema-bearing parts of SEP-1913 into separate
experimental extensions, each with its own `io.modelcontextprotocol/…`
identifier and reference implementation, rather than pursuing one broad
Standards Track SEP.

**Rationale.** @localden's review asked for a *narrower first cut*; the IG
[aligned 2026-05-28](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/2820)
on an extension-first strategy. Independent extensions can graduate on their own
clock and avoid hard-to-remove schema.

## 2026-06-10 — Three initial extensions

**Decision.** `trust-annotations` (primary), `action-metadata`, `ifc-fides`.

**Rationale.** These are the three pieces with either a reference implementation
or an existing SEP behind them: Kapil's SDK, SEP-2061 (Reichel), and the FIDES
model respectively.

## 2026-06-10 — FIDES is a profile, not a top-level extension

**Decision.** Information-flow control is `type: "ifc.fides.v1"`, a profile of
the `trust-annotations` `evidenceRef` slot — not a top-level `io.modelcontextprotocol/ifc`
extension.

**Rationale.** IFC is one enforcement model among several raised in review
(capability tokens — pshkv; cosigning — viftode4; sequence shape — marras0914).
A top-level `ifc/` root would foreclose those. Reviewer endorsement was for IFC
"if you use annotations" — i.e. as a profile.

## 2026-06-10 — `evidenceRef.type` is an open string

**Decision.** `type` MUST remain an open string with a non-binding registry of
well-known values; never a closed enum. Required fields are `digest` and
`canonicalization`; `schema` recommended; `ref` optional.

**Rationale.** Adapted from the vaaraio / Rul1an convergence in the SEP-1913
thread. An open `type` is what lets IFC, data-class, sequence-shape, and
attestation profiles share one slot.

## 2026-06-10 — `requiresReview` moves to `action-metadata`

**Decision.** `requiresReview` is an `action-metadata` field, not a
`trust-annotations` field.

**Rationale.** It is a workflow/consent signal, not a data-classification
property. Keeping it out of the trust taxonomy avoids reproducing SEP-1913's
"several concerns in one schema" problem at smaller scale.

## 2026-06-10 — DataClass demoted to a profile

**Decision.** The wire taxonomy keeps only the coarse `sensitive` boolean.
The four-level classification + regulatory scope becomes an `evidenceRef`
profile `type: "data-class.v1"`.

**Rationale.** Coarse binary is universally client-actionable and cheap on the
wire; the richer taxonomy can evolve behind a profile without a breaking schema
change.

## 2026-06-10 — Parked: maliciousActivityHint, propagation rules

**Decision.** Neither becomes an extension now; both stay on the SEP-1913
umbrella.

**Rationale.** `maliciousActivityHint` has unresolved structural objections
(fires pre-execution at `tools/resolve`; boolean granularity wrong for UX;
clients won't trust server self-attestation). Propagation/sequence-shape needs
the taxonomy and `evidenceRef` stable first.

## 2026-06-10 — Citations: public sources only

**Decision.** Reference implementations and motivating examples cite **public**
artifacts — [`github-mcp-server`](https://github.com/github/github-mcp-server),
[`kapil8811/mcp-trust-annotations`](https://github.com/kapil8811/mcp-trust-annotations),
[arXiv:2505.23643](https://arxiv.org/abs/2505.23643) — and index on the public
SEP-1913 review record (esp. @localden). Private/internal implementations are
not named or linked.

## 2026-06-10 — Pre-flight (SEP-1862) stays core

**Decision.** These extensions are response-level and do not depend on Tool
Resolution. SEP-1862 remains a core/Standards-Track protocol change.

**Rationale.** The 2026-05-28 IG meeting concluded pre-flight is inherently a
protocol-level change, not an extension.
