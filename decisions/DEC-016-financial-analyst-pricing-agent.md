# DEC-016: Financial Analyst & Pricing Agent

Status: Accepted

## Decision

Add a Financial Analyst & Pricing agent (`FINANCIAL`) to the agent team. It operates the startup financial model — revenue, costs, unit economics — monitors messaging costs against margins, and produces pricing-strategy analysis and proposals. Pricing and spend decisions remain human-owned (founder, DEC-011; AGENTS.md approval gates).

## Reason

- Messaging is the margin-killer of a queue SaaS: WhatsApp/SMS per-message charges scale with every notification (join confirmation, table-ready, reminders). Nobody currently owns cost-per-notification or per-restaurant messaging economics, so a silent margin erosion would go unnoticed until month-end invoicing.
- No one currently tracks actual costs (Fly, Neon, Vonage/Twilio, DeepSeek, GitHub, third-party SaaS) against a model, or computes unit economics.
- When Billing (RESQ-40) ships, revenue tracking (MRR/ARR/ARPU, by plan, by restaurant, expansion, discounts, failed payments) needs an owner from day one.
- Pricing strategy needs data and analysis before founder decisions; the agent provides the analysis, never the decision.

## Scope of the model

- Revenue: MRR, ARR, ARPU, revenue by plan, revenue by restaurant, expansion revenue, discounts, failed payments.
- Costs: AI/API, cloud, database, SMS, WhatsApp, payment gateway, infrastructure, third-party SaaS, support, acquisition.
- Unit economics: CAC, LTV, gross margin, contribution margin, payback period, revenue per restaurant, cost per restaurant — plus cost per notification and messaging cost as % of ARPU.

## Data discipline

Actuals come from provider invoices/dashboards only; estimates are always marked with stated assumptions; data gaps are flagged, never silently filled. This is the agent's non-negotiable rule — a financial agent that fabricates numbers is worse than no agent.

## Implication

- New Jira persona account: `financial.analyst` (label `agent-financial`, commit prefix `FINANCIAL`).
- Canonical spec: `.agents/specs/financial-analyst.md`; Claude Code adapter: `.claude/agents/financial-analyst.md`.
- The financial model lives in `queue-docs/FINANCIALS.md` (versioned, append-only historical months).
- Financial/pricing work routes through the Billing epic (RESQ-40) when it affects monetization; pricing changes stay `HUMAN APPROVAL REQUIRED`.
- Until Billing ships, revenue lines are zero; the model is a cost + unit-economics model ready to ingest revenue.

## Related

- `queue-docs/AGENTS.md` — agent team, approval gates (pricing changes are human-gated)
- `queue-docs/decisions/DEC-011-product-owner-owns-strategic-direction.md` — founder owns pricing decisions
- `queue-docs/decisions/DEC-008-start-with-five-ai-delivery-agents.md` — agent team evolution
