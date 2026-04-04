# Teardown — VoiceAI Analytics

A production-grade analytics platform analysing Voice AI interaction data to evaluate accessibility, efficiency, adoption, and error reduction. Built to demonstrate 6+ years of Analytics Engineering maturity through a real analytical pipeline.

---

## Stack Choices & Rationale

| Component | Decision Rationale |
|---|---|
| **PostgreSQL** | Relational warehouse for both raw and analytics data. Mature, well-supported by dbt and Metabase, and realistic for the scale of a Voice AI session dataset. |
| **dbt** | Handles all transformation from raw PostgreSQL tables through staging models to the session-level fact table. dbt tests enforce data quality before any metric is exposed to dashboards. |
| **Metabase** | Open-source BI tool that connects directly to the dbt-produced analytics schema. Satisfies the governance requirement: Metabase only queries certified dbt models, not raw tables. |
| **Python (Pandas, sklearn)** | Used only where SQL is insufficient — logistic regression for abandonment modelling, statistical segmentation, and confidence interval calculations. Python is not the transformation layer. |
| **Docker Compose + Dev Containers** | Reproducible environment across machines. The VS Code Dev Containers extension eliminates "works on my machine" issues for anyone cloning the repo. |
| **One fact table, session grain** | `fact_voice_ai_sessions` models sessions as the natural decision unit. Avoids fan-out joins and metric inconsistency caused by mixed grains. |

---

## Key Design Decisions

- **Business logic lives in dbt, not in dashboards.** Metabase charts are thin: they select from pre-computed dbt models, not define metrics ad hoc.
- **Python (Logistic Regression)** is used to identify ASR confidence as the strongest abandonment predictor — a finding that cannot be derived from SQL aggregations alone.
- **PII protection is enforced at the model level**: no raw audio, user IDs anonymised, aggregated reporting only. This is a design constraint, not an afterthought.
- **Fact table grain is session-level**, not turn-level. Applications data was explicitly excluded after validation — a documented modelling decision, not an omission.
- **dbt tests** cover not-null, uniqueness, relationships, accepted ranges (0–1 for proportions), and custom business logic (error proportions ≤ 100%, sessions must have turns).

---

## Trade-offs

| Decision | Benefit | Cost |
|---|---|---|
| Session-grain fact table | Clean, consistent metrics — one row per decision unit | Turn-level analysis requires going back to raw tables or building a separate turn-level model |
| Metabase as BI layer | Open-source, quick to set up, dbt-aware | Less powerful than Tableau or Looker for enterprise self-serve; limited semantic layer capabilities |
| Logistic Regression | Interpretable, fast to train, confidence intervals available | Does not capture non-linear interactions; a gradient boosted model would likely have higher AUC |
| Docker Compose dev environment | Fully reproducible, no local installs required | Slower iteration cycle than a native environment; resource-heavy on lower-spec machines |

---

## Extensions & Real-World Use Cases

- Replace Metabase with **Superset or Lightdash** for richer semantic layer integration with dbt metrics.
- Add a **dbt snapshot model** to track changes in session outcomes over time — useful for measuring the impact of ASR model updates longitudinally.
- Build a **feature store** from the session-level features for use in a real-time scoring API (natural bridge to the Fraud Detection Platform pattern).
- Extend to **multi-language ASR evaluation** — segment confidence scores and error rates by language to identify which languages need model improvement.
- Apply the same architecture pattern to any **conversational AI product**: chatbot sessions, IVR logs, or customer service transcripts.

---

## Portfolio Signal

The explicit BI governance boundary — "if Metabase can see it, it has already been tested" — is a production engineering philosophy, not a portfolio talking point. It signals maturity in how analytics systems are designed for trust and reliability.
