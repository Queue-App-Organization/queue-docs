# DEC-010: Continuous Work Needs Gates

Status: Accepted

## Decision

Agents can work continuously, but high-risk actions require human approval.

## Reason

Unbounded autonomous work can cause business, security, data, or production risk.

## Implication

Human approval is required for destructive migrations, security-sensitive work, pricing/payment changes, deleting customer data, major architecture changes, external communication, and early production deployments.

## Related

See `queue-docs/AGENTS.md`.
