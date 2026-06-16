---
title: Data classification (data-labelling scheme)
---

> ⚠️ **Experimental scheme skeleton.** This is one **data-labelling scheme** that
> fills the [`trust-annotations`](../specification/draft/trust-annotations.mdx)
> `evidenceRef` slot via `type: "data-class.v1"`. It is **not** an MCP extension
> and **not** a sibling of the extensions — it is the home for the richer
> classification taxonomy that the extension's coarse `sensitive` boolean
> deliberately leaves out. See [Why a scheme](#why-a-scheme-not-a-wire-field).

**Scheme** — fills `trust-annotations`'s `evidenceRef` slot, selected by
`evidenceRef.type == "data-class.v1"`. Not an extension.

## Abstract

This scheme defines `data-class.v1`, an entry for the `trust-annotations`
`evidenceRef` slot that carries a **structured data classification**: a coarse
sensitivity level plus zero or more regulatory scopes and optional org-defined
labels. It is the structured counterpart to the extension's single
`sensitive: boolean`, for deployments that need to express *which* regulated
category a result falls under (e.g. HIPAA, GDPR, PCI-DSS) without putting that
taxonomy on the wire for every server.

## Why a scheme, not a wire field

The `trust-annotations` extension keeps only `sensitive: boolean` on the wire.
That coarse signal is universally client-actionable and cheap, and — critically
— it cannot become wrong as regulation changes. A richer classification was
asked for repeatedly in review, in three different shapes:

- a **linear** `sensitiveHint: low | medium | high`, which
  [@JustinCappos](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/711#issuecomment-2967516811)
  rejected: sensitivity is set-theoretic (a card number and a medical record are
  both sensitive but to *different* readers), not a single scale;
- **org-defined vocabularies** rather than a fixed enum
  ([@olaservo](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/711#issuecomment-2968743154);
  [@Mossaka](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/711#issuecomment-2971788308)
  proposed reverse-DNS `classification.labels`);
- a **class + regulatory scope** pairing such as `confidential:hipaa`
  ([@krubenok](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913#discussion_r3103485194);
  carried into SEP-2061 by @rreichel3).

Against all three, [@localden](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913#issuecomment-4037623595)
warned that a taxonomy baked into the protocol is very hard to remove if it turns
out wrong. A scheme resolves the tension: the taxonomy evolves behind a `type`
value, on its own clock, while the wire keeps the boolean that can never rot.

## Motivation

A regulated deployment needs to know not just *that* a result is sensitive but
*under which regime*, so the host can apply the matching control (HIPAA minimum-
necessary, GDPR purpose limitation, PCI-DSS storage rules). The boolean cannot
carry that, and encoding it inline would force every server and client to agree
on a regulatory taxonomy. This scheme lets a server that already knows the
classification (a healthcare record store, a payments API) attach it as evidence,
while a client that does not implement the scheme still sees `sensitive: true`.

## Specification

### Scheme identity

A `data-class.v1` record is selected by `evidenceRef.type == "data-class.v1"` on
a `trust-annotations` annotation whose `sensitive` is `true`. The
`digest`/`canonicalization` pair over the record is required exactly as for any
other scheme.

### Payload (skeleton)

The record is a JSON object. Shapes below are **open questions**, not settled:

```json
{
  "class": "confidential",
  "regulatory": ["hipaa"],
  "labels": ["com.example.pii.ssn"],
  "policyRef": "https://policy.example.com/classes/confidential"
}
```

- `class` — a coarse level. Candidate set `public | internal | confidential |
  restricted` (four levels, from the original SEP-1913 taxonomy). Whether the set
  is fixed or registry-curated is open.
- `regulatory` — zero or more regulatory scopes the result falls under. Open
  strings, not an enum (so a new regime doesn't need a schema change). This is
  the `:scope` half of @krubenok's `confidential:hipaa`.
- `labels` — optional org-defined tags, reverse-DNS namespaced per @Mossaka, for
  vocabularies a deployment defines for itself.
- `policyRef` — optional pointer to the policy that defines the classes, so the
  record is interpretable by a host that hasn't pre-agreed the vocabulary.

### Graceful degradation

A client that does not implement `data-class.v1` ignores the record and relies on
the `trust-annotations` `sensitive` boolean, which remains meaningful on its own.
The scheme only ever *refines* the boolean; it never contradicts it (a
`data-class.v1` record only appears where `sensitive` is already `true`). Because
the boolean is the lowest-common-denominator floor, a producer of this scheme
SHOULD emit **both** the boolean and the record, never the record alone.

## Producer / consumer

- **Producer (candidate).** A server fronting a regulated store (health records,
  payments) that already classifies its data internally.
- **Consumer (candidate).** A host policy engine that maps `class` + `regulatory`
  to a control (block, redact, prompt). The same host-side resolution model as
  the FIDES scheme applies: the wire marker is advisory; the regulated semantics
  are host-resolved against the deployment's policy.

## Open questions

- Is the `class` set fixed (`public/internal/confidential/restricted`) or
  registry-curated? @localden's removability concern argues against fixing it.
- Do `regulatory` and `labels` overlap enough to collapse into one open-string
  list, or are "named regime" and "org-defined tag" usefully distinct?
- Should `data-class.v1` ever be expressible **on the wire** as a structured
  escape hatch (tracked in [open-questions.md](../docs/open-questions.md)),
  or is keeping it strictly behind `evidenceRef` the right boundary?
- How does this relate to SEP-2061's `DataClass` / `RegulatoryScope` — is this
  scheme the migration target for that work, or a parallel encoding?

## Changelog

- **2026-06-16** — Initial skeleton. Payload shapes are candidates pending the
  open questions above.
