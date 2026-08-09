# Voice & Tone Guide

> **This is a hand-maintained copy.** The source of truth lives in the product repo,
> `whitebox-xai-azure`, at `.claude/Voice_And_Tone_Guide.md` (also copied alongside
> `Primary_Marketing_Persona.md` and `Investor_Acquirer_Persona.md`, which are not duplicated
> here). There is no automated sync between the two repos — if the source changes materially,
> re-copy it here by hand. Don't edit this file and the azure repo's copy independently and
> expect them to reconcile; treat the azure repo's version as authoritative on conflict.

This file exists because nothing previously said how WhiteBoxXAI copy should *sound* — only who
it's for. The rules below apply to **both** whiteboxxai.com marketing copy (product repo) and
this site's documentation. The same buyer reads both — a compliance officer evaluating the
product reads marketing copy to decide whether to take a call, then reads these docs to decide
whether to trust what the sales conversation claimed. A tone mismatch between the two ("confident
and vague" on the marketing site, "hedged and precise" here, or vice versa) reads as either
oversold or under-engineered. The tone-vs-substance shift between docs written for the ML end
user and docs written for the compliance buyer is also intentional — those readers are
different, but neither is worse than the other.

## Register: grounded, evidence-based, skeptic-aware

The economic-buyer personas for this product (compliance/GRC officers — see the product repo's
`Primary_Marketing_Persona.md` for full avatars) have been burned by vendors selling "compliance
theater": capability claims that don't survive audit scrutiny. Their default posture toward any
AI-governance vendor is skepticism, not excitement. Copy earns trust by showing *how* something
is computed, not by asserting confidence about *what* it does.

- **Don't:** "Best-in-class explainability for every model." (asserts, doesn't show)
- **Do:** "SHAP and LIME computed from your live model, not templated — every value in an
  export traces back to a real computation you can re-run." (shows the mechanism, addresses the
  skepticism directly)

Don't overclaim to sound impressive, and don't over-hedge to sound careful — state exactly what
is true, because the technical buyer persona will test the boundary of the claim.

## Lead with the deadline or gap, not the category name

"Your GRC platform can't produce this evidence for auditors" converts better than "AI governance
platform" as an opener, because the reader doesn't wake up wanting a category of software — a
specific deadline, questionnaire, or audit creates the urgency. Open pages (Get Started, feature
pages, page introductions) with the concrete gap the reader has, not the label for the product.

## No Fortune-500-coded pricing or packaging language

The secondary compliance-buyer avatar is explicitly price-sensitive relative to enterprise tools
and is "specifically searching for something not built for Fortune 500 budgets." Avoid
packaging language that reads as enterprise-tier by default on pages this persona is likely to
land on (Plans & Limits, Getting Started).

## Technical depth is a trust signal, not just engineer content

The compliance buyer persona will actually read documentation on how SHAP explanations are
computed, not just take a sales pitch at face value. Don't dumb down the technical explanation
of *how a number is computed* on the theory that the compliance buyer doesn't want detail — that
detail is exactly what earns trust. What changes by audience is framing, not rigor:

- **SDK & API docs, integration guides:** lead with integration mechanics — decorators,
  `wrap_model`, framework names. Skip compliance jargon entirely; this reader doesn't care about
  ISO 42001 and forcing it in front of them reads as irrelevant, not thorough.
- **Audit & Explanation Reports, Trust Score, Governance pages:** the same computational detail
  belongs here too, but framed as *evidence of authenticity* ("here is exactly how this number
  is derived") rather than a feature pitch ("here is a cool capability").

Never let technical depth be the *headline* on a compliance-buyer-facing page, even when it's
present — technical content is a means to the compliance buyer's end (faster or more defensible
audit evidence), not the narrative itself.

## Jargon rules

- Don't open compliance-buyer-facing pages with the category name ("AI governance platform").
- Don't use compliance/regulatory jargon in SDK/integration pages.
- Regulatory framework names (ISO/IEC 42001, GDPR, CCPA, EU AI Act, NIST AI RMF) should only
  appear where accurate — see the product repo's `Key_Compliance_Frameworks.md` for which
  frameworks are actually in scope. Naming a framework the product doesn't map to is a trust
  failure, not a minor inaccuracy.

See the product repo's `Primary_Marketing_Persona.md`, `Investor_Acquirer_Persona.md`, and
`Key_Compliance_Frameworks.md` for who the audience is and what's in scope — this file is about
how to write for them once that's settled, not a replacement for reading those.
