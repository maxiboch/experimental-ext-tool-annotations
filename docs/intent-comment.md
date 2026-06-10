# Intent comment (draft, pre-post review)

This is the comment we plan to post on
[SEP-1913](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913),
with an abbreviated pointer version for
[SEP-2061](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2061).
Kept here so it can be reviewed and stays in sync with
[sep-disposition.md](./sep-disposition.md).

---

## For SEP-1913

> **Intent: split this SEP and migrate to the Extensions Track**
>
> A note on direction for everyone following this thread. When SEP-1913 was
> first framed, the **Extensions Track** (SEP-2133) and the `experimental-ext-*`
> incubation process didn't exist in their current form. They now do, and
> they're a better fit for this work than a single Standards Track SEP.
>
> Two things pushed us here:
> - @localden's review ask for a **narrower first cut** — the concern that a
>   broad taxonomy with array-or-scalar polymorphism is hard to remove or change
>   once it lands.
> - The Tool Annotations IG's
>   [May 28 decision](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/2820)
>   to pursue trust/privacy as an **experimental extension first**, gather
>   adoption evidence, then ask core maintainers to absorb anything.
>
> So the plan is to **carve this proposal into a few small,
> independently-shippable extensions**, each with its own
> `io.modelcontextprotocol/…` identifier, reference implementation, and path to
> an Extensions Track SEP. Incubation is in
> [`experimental-ext-tool-annotations`](https://github.com/modelcontextprotocol/experimental-ext-tool-annotations):
>
> | Extension | Scope |
> |---|---|
> | `trust-annotations` | The narrow data-classification taxonomy (`sensitive`, `untrusted`) + an open-ended `evidenceRef` pointer for richer, out-of-band evidence. |
> | `action-metadata` | Tool I/O + outcome contract (folds in @rreichel3's SEP-2061). |
> | `ifc-fides` | Information-flow control ([arXiv:2505.23643](https://arxiv.org/abs/2505.23643)) as **one profile** of `evidenceRef`, not a wire root — the public/private-repo confidentiality case, with github-mcp-server as emitter candidate. |
>
> Deliberately **not** in the initial carve: `maliciousActivityHint` (the
> structural concerns raised here are unresolved) and session-level propagation
> rules. Those stay parked on this umbrella thread.
>
> This follows the same Standards-Track → Extensions-Track refactor pattern as
> SEP-2127 (#2893). This PR stays open as the umbrella / problem-framing thread;
> the schema-bearing pieces move out. Feedback on the carve is very welcome —
> particularly on whether any parked item deserves its own extension sooner.

---

## For SEP-2061 (abbreviated pointer)

> Cross-linking for visibility: the Tool Annotations IG is carving the trust /
> privacy / action-metadata work into small experimental extensions in
> [`experimental-ext-tool-annotations`](https://github.com/modelcontextprotocol/experimental-ext-tool-annotations)
> (background: SEP-1913 comment
> [here](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913),
> and the [May 28 IG decision](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/2820)).
>
> This proposal maps directly onto the
> [`action-metadata`](https://github.com/modelcontextprotocol/experimental-ext-tool-annotations/blob/main/specification/draft/action-metadata.mdx)
> extension — `inputMetadata` / `returnMetadata` / outcomes, plus a
> `requiresReview` signal we pulled out of the trust taxonomy because it's a
> workflow concern, not a data property. The intent is to carry SEP-2061 forward
> as that extension rather than run a parallel proposal. @rreichel3 — flagging
> so we co-own it rather than diverge; happy to keep this thread as the home for
> the field semantics.
