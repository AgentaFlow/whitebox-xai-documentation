---
description: The WhiteBoxXAI Model Register — every governed AI system with lens-specific columns, filters and CSV export, and the per-system dossier with verdicts, signals, model card, control coverage, trust score history and audit trail.
---

# Model Register

The **Model Register** is the inventory of every AI system you govern: models, LLM apps and agent
systems. It's where an auditor's first question, *what AI do you run, and who's accountable for
it?*, gets its answer. Each row opens the system's **dossier**.

Press `g` then `r` to get here. **Register a system** adds one from the dashboard; to add one from
code, see [Getting Started](../get-started/getting-started.md#4-register-a-model-and-log-predictions).

## The register table

Which columns lead depends on your [lens](index.md#lenses-view-as). Available columns:

| Column | Shows |
| --- | --- |
| **Model**, **Type**, **Status**, **Owner**, **Registered** | Identity and lifecycle |
| **Trust score** | The system's current [Trust Score](trust-score.md) |
| **EU AI Act tier** | The risk classification recorded for the system |
| **Lifecycle stage** | Where the system is in its lifecycle |
| **Open findings** | Unresolved findings against the system |
| **Evidence** | Whether approved evidence exists |
| **Last attested** | When the system was last attested |
| **Drift**, **Fairness gap** | Latest drift status and fairness gap |
| **Last seen** | When data last arrived from the system |

### Filters

Search by name, plus **Risk tier**, **Lifecycle**, **Drift**, **Findings**, **Evidence** (**Has
approved** / **Missing evidence**), **Status** (Active, Deprecated, Archived), **Owner** and **Tag**.
Filters are kept in the page address, so you can share a filtered view by sharing the link.
**Clear filters** resets them.

### Toolbar

- **Saved views**: save the current filters and columns ([how](index.md#saved-views))
- **Columns**: choose which columns to show, or **Reset to default**
- **Export CSV**: exports *every* row that matches your filters, not just the visible page

## The dossier

Click a system to open its dossier in a slide-over. The page address includes the system, so you
can link to a dossier directly.

### Governance verdict

Four verdict cards summarise the system's state in plain language. Each has **How was this
computed?** and, for analysts and administrators, **Mark reviewed**. Marking a verdict reviewed
records who reviewed it and when. It's refused if the underlying figures have changed since the
page loaded, so nobody signs off on numbers they didn't see.

### Assurance timeline

Assurance events for the system (audits, drift reports, reviews and decisions) in date order.

### Signals

| Tab | Contents |
| --- | --- |
| **Drift** | Latest drift results for the system |
| **Fairness** | Latest bias audit, including *"Recommended mitigations, as the audit recorded them"* |
| **Explainability** | Recent explanations for the system's predictions |
| **LLM & agents** | LLM usage, safety and agent activity for LLM-based systems |
| **Lineage & versions** | Every registered version, with **Compare versions and see the code diff** |
| **Model card** | The system's model card: card version, publication status and freshness |

A model card marked **Stale — needs refresh** says why under **Why this model card is marked
stale**: a newer source commit was registered, or a newer fairness audit or drift report was
recorded, after the card was last updated.

### Findings, coverage and history

- **Open findings** for the system
- **Control coverage**: **N of M** in-scope controls with evidence (a count, never a percentage)
- **Trust score history**, with its three components
- **Audit trail**: every recorded action on this system

### System settings

- **Edit details**: description, status and tags, then **Save changes**
- **Archive** / **Restore**: archiving asks *"Archive this model?"*. Archived systems keep their
  history, stop counting towards your plan's [Governed AI Systems](../get-started/governance-events.md#what-is-one-governed-ai-system),
  and can be restored later.

## Versions and code diffs

Register a new version of a system by registering it under the **same name** with a new version
string. It stays one Governed AI System. From **Lineage & versions**, compare two versions to see
what changed between them, including the code changes when the versions were registered with Git
context. Model-diff architecture reviews are included on Business Cloud and Enterprise Edition. See
[GitHub Diff Reviews](../integrations/github-diff.md).

## From code

```python
client.models.list(search="credit", limit=50)
client.models.get(model_id)
client.models.get_versions("credit_scoring")
client.models.compare_versions(model_id_a, model_id_b)
client.models.archive(model_id)
client.models.restore(model_id)
```

See [SDK & API](../sdk/index.md).
