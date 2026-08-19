# DEC-012: Backward Compatibility Before Customers

Status: Accepted

## Decision

Before real customers are using the product, optimize for the best product and system design instead of preserving backward compatibility.

## Reason

The project is still pre-customer. Compatibility layers, legacy flows, and migration constraints can slow down iteration and preserve weaker early decisions. Until restaurants and customers depend on the system, the better default is to simplify, correct, and improve the design aggressively.

## Compatibility Rule

Pre-customer:

- Breaking internal APIs is allowed when it improves clarity, correctness, or long-term architecture.
- Database schemas may be changed aggressively if data is disposable or can be regenerated.
- UI flows may be replaced when a better workflow is found.
- Old compatibility paths should be removed when they add confusion.

Post-customer:

- Preserve compatibility unless a documented migration, rollout plan, and approval exist.
- Treat QR links, WhatsApp flows, production data, staff workflows, APIs, and integrations as contracts.

## Decision Checklist

Before choosing whether to keep backward compatibility, ask:

- Are real customers or staff using the affected behavior?
- Does production data depend on the old schema or format?
- Are external systems, WhatsApp templates, QR links, APIs, or integrations using the current contract?
- Will removing compatibility break active restaurants during service?
- Is a migration path available?
- Can the change be rolled out safely behind a feature flag?
- Is the old behavior blocking a materially better product or architecture?

## Implication

Agents should default to clean design while pre-customer. Once customer usage begins, breaking changes require explicit decision records, migration plans, rollout strategy, and approval.

## Related

See `queue-docs/AGENTS.md`.
