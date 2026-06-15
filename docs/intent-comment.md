# Intent comments (posted)

Both comments below have been **posted**. Kept here as the source of record,
in sync with [sep-disposition.md](./sep-disposition.md).

- **SEP-1913** umbrella comment — [posted 2026-06-10](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913#issuecomment-4675047154).
- **SEP-2061** coordination note — [posted 2026-06-10](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/2061#issuecomment-4675049171);
  @localden then **closed SEP-2061 on 2026-06-13** in favour of the
  `action-metadata` extension.

---

## For SEP-1913

> **Intent: split this SEP and migrate to the Extensions Track**
> >
> A note on direction for everyone following this thread. When SEP-1913 was
> first framed, the **Extensions Track** (SEP-2133) and the `experimental-ext-*`
> incubation process didn't exist in their current form. They now do, and
> they're a better fit for this work than a single Standards Track SEP.
> >
> Two things pushed us here:
> >
> - @localden's review ask for a **narrower first cut** — the concern that a
>   broad taxonomy with array-or-scalar polymorphism is hard to remove or change
>   once it lands.
> - The Tool Annotations IG's
>   [May 28 decision](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/2820)
>   to pursue trust/privacy as an **experimental extension first**, gather
>   adoption evidence, then ask core maintainers to absorb anything.
> >
> So the plan is to \*\*split this proposal into a few small,
> independently-shippable extensions\*\*, each with its own
> `io.modelcontextprotocol/…` identifier, reference implementation, and path to
> an Extensions Track SEP. Incubation is in
> [`experimental-ext-tool-annotations`](https://github.com/modelcontextprotocol/experimental-ext-tool-annotations).
> >
> Kicking off with a few draft extensions in the tool annotations repo — not
> sure yet whether they'd each need separate repos eventually, or whether
> grouping them in one is fine. That's part of what incubation is for.
> >
> | Extension | Scope |
> |---|---|
> | `trust-annotations` | The narrow data-classification taxonomy (`sensitive`, `untrusted`) + an open-ended `evidenceRef` pointer for richer, out-of-band evidence. |
> | `action-metadata` | Tool I/O + outcome contract (folds in @rreichel3's SEP-2061). |
> | `ifc-fides` | Information-flow control ([arXiv:2505.23643](https://arxiv.org/abs/2505.23643)) as **one profile** of `evidenceRef`, not a wire root — the public/private-repo confidentiality case, with github-mcp-server as an emitter example. |
> >
> Deliberately removed: `maliciousActivityHint` (the structural concerns raised
> here are unresolved) and session-level propagation rules.
> >
> This follows the same Standards-Track → Extensions-Track refactor pattern as
> SEP-2127 (#2893). This PR will eventually pivot to the `trust-annotations`
> piece itself, with the other schema-bearing pieces moving out into their own
> extensions. Everything is still in the incubation phase, so naming, design,
> and the choice of what to put forward as an extension are all open for
> discussion in the IG.

---

## For SEP-2061 (coordination note)

> @rreichel3 — splitting this out into an independent extension as discussed: https://github.com/modelcontextprotocol/modelcontextprotocol/pull/1913#issuecomment-4675047154