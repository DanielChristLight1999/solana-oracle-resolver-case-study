# Engineering Decisions

## Decision 1 — Idempotency before throughput

The resolver's first responsibility is safe state transitions, not maximum event-processing throughput.

Repeated observations are expected. Therefore every state-changing path checks whether the intended transition is still necessary.

## Decision 2 — Validate close to the transaction

An automated service should not assume that an earlier observation remains valid when a transaction is finally submitted.

The resolution path therefore performs relevant authorization and state checks immediately before the state-changing operation.

## Decision 3 — Separate protocol observation from business decisions

Raw Solana account data is not the same thing as a resolver decision.

The conceptual separation is:

`Protocol data → Protocol parsing → Normalized state → Decision logic → Transaction`

This reduces coupling and makes the decision engine easier to test.

## Decision 4 — Bounded retries

Network and RPC failures are not necessarily application failures.

Retries should be bounded, delayed using exponential backoff, applied only where retrying is safe, and combined with idempotency/state checks.

## Decision 5 — Operational visibility

Automated infrastructure needs a way to answer basic operational questions:
- What state is being observed?
- What was processed?
- What was resolved?
- When was the last observation?
- Is the service making progress?

A normalized status endpoint provides a lightweight operational surface without exposing internal implementation details.

## Decision 6 — Public documentation without private implementation

The public repository demonstrates architecture and engineering judgment while keeping the production implementation private.

That boundary is intentional: the value of the case study is the engineering reasoning, not disclosure of confidential source code or credentials.