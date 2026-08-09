---
name: whitebox-voice-tone
description: Keeps this documentation site (docs.whiteboxxai.com) and WhiteBoxXAI's marketing site (whiteboxxai.com, separate repo whitebox-xai-azure) speaking in one consistent voice — grounded, evidence-based, skeptic-aware — calibrated to the buyer and end-user personas rather than generic SaaS copy. TRIGGER — read BEFORE: writing or editing any page under docs/**; drafting release notes, FAQ, or troubleshooting content; reviewing existing pages for tone. SKIP when the task has no reader-facing copy angle (config/build changes, mkdocs.yml navigation-only edits, asset updates).
user-invocable: true
---

# WhiteBoxXAI Voice & Tone

Full source of truth: [`.claude/Voice_And_Tone_Guide.md`](../../Voice_And_Tone_Guide.md) — read
it directly before writing anything reader-facing. **This file and its source doc are
hand-maintained copies of the ones in the product repo, `whitebox-xai-azure`** (see the banner
at the top of the guide) — there is no automated sync; re-copy by hand when the source changes
materially, and treat the azure repo's version as authoritative on conflict.

## The core rules

- **Register:** grounded, evidence-based, skeptic-aware. The buyer persona has been burned by
  "compliance theater" vendors — show *how* something is computed rather than asserting *what*
  it does.
- **Lead with the deadline or gap**, not the category name ("AI governance platform").
- **No Fortune-500-coded pricing/packaging language** on pages like Plans & Limits or Getting
  Started.
- **Technical depth is a trust signal for the compliance buyer, not just engineer content** —
  don't dumb down *how a number is computed*. What changes by audience is framing
  (evidence-of-authenticity vs. feature-pitch), not rigor.
- **SDK & API / integration pages** skip compliance jargon entirely and lead with integration
  mechanics.
- Full rationale and before/after phrasing examples are in the source doc — read it rather than
  working from this summary alone when the stakes are a real published page.

## One voice across two sites

The same reader who lands on whiteboxxai.com to evaluate the product reads these docs next to
check whether the pitch holds up. A tone mismatch between the marketing site and this site
undermines the docs even when each page is individually fine on its own. If you're editing
marketing copy in the `whitebox-xai-azure` repo instead, that repo carries the authoritative
copy of this same skill and guide.

## How to apply this when working forward

- **Before publishing a page:** check register (evidence-based, not asserted), opening
  (deadline/gap, not category name), and audience (SDK/integration pages get mechanics + no
  jargon; user-guide/compliance pages get the same technical depth framed as authenticity
  evidence).
- **Reviewing existing pages:** if a page reads as confidently vague ("best-in-class",
  "seamless", "enterprise-grade" with no mechanism named), that's the failure mode this skill
  exists to catch.
- **Feature/scope questions** (which frameworks are in scope, whether a feature described here
  is actually shipped) are not this skill's job — check the product repo's
  `Key_Compliance_Frameworks.md` and its `scripts/check_marketing_features_shipped.py` /
  `scripts/check_docs_site_feature_sync.py` instead. This skill is about *how* to phrase
  something already known to be true and in scope.
- **Delegating to a subagent:** subagents start with no memory of this conversation. Paste
  `.claude/Voice_And_Tone_Guide.md`'s path or its key rules directly into the subagent's prompt
  — don't assume it will load this skill on its own.
