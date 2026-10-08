# Solana Oracle & Prediction-Market Resolver

> **Sanitized portfolio case study of proprietary backend infrastructure.** The original implementation remains private.

This case study describes a NestJS-based Solana service that observed protocol state and acted as an automated oracle/resolver for prediction markets.

The central engineering problem was turning **repeated, asynchronous blockchain observations into safe, deterministic state transitions**.

## At a Glance

| Area | Details |
|---|---|
| Runtime | Node.js / TypeScript |
| Framework | NestJS |
| Blockchain | Solana |
| SDKs | @solana/web3.js, Anchor |
| Pattern | Event/subscription-driven processing |
| Core concerns | Observation, validation, idempotency, resolution |
| Source | Private / proprietary |

## System Architecture

```text
Solana / ORE Protocol
          │
          ▼
Account Subscriptions
          │
          ▼
State Services
Parse + Validate + Normalize
          │
          ▼
Resolver Decision Engine
       │       │
       ▼       ▼
Range Closure  Market Resolution
       │       │
       └───┬───┘
           ▼
   Solana Transactions
```

## Design Principles

- **Trusted** — decisions follow explicit validation and authorization rules.
- **Idempotent** — repeated observations must not create duplicate state transitions.
- **Subscription-driven** — react to relevant account changes instead of blindly scanning everything.
- **Replaceable** — resolver behavior is separated from the protocol itself.

## Observation Layer

### Treasury state
The service observed treasury-related protocol state, including values such as current epoch, jackpot/motherlode balance, and formatted accumulated value.

Observed updates were handled so stale or decreasing observations would not silently move application state backwards.

### Round state
The ORE integration tracked information such as current round, epoch, motherlode amount, and motherlode-hit status.

The service followed the active round as protocol state advanced.

## Resolution Flow

### Range closure
1. Receive updated state.
2. Convert the value into the comparison unit.
3. Identify ranges whose upper bounds were surpassed.
4. Validate each candidate.
5. Submit the close transaction.
6. Record the action for idempotency.

### Market resolution
1. Detect a motherlode hit from round state.
2. Verify the market exists and is unresolved.
3. Verify the configured oracle is authorized.
4. Calculate the winning range.
5. Verify that range exists and remains eligible.
6. Submit the resolution transaction.
7. Record the market as resolved.

## Idempotency

Blockchain listeners can receive the same observation more than once. Retries can also repeat work.

The resolver therefore combined:
- in-memory processed-state tracking
- on-chain state checks before actions
- transaction retry logic with exponential backoff
- oracle authorization checks
- explicit market/range validation

> **The same observation should not accidentally produce the same state transition twice.**

## Failure Handling

The service accounted for transient failures such as WebSocket subscription errors, RPC/network failures, missing accounts, invalid or incomplete observations, and transaction submission failures.

Transient transaction failures could be retried using bounded exponential backoff instead of immediately terminating the service.

## Operational Visibility

A status API exposed a normalized snapshot of relevant observed state, including treasury observation, current ORE round state, current market state, closed ranges, resolved markets, and processing timestamps.

## Security Boundary

Automated blockchain services are security-sensitive because a resolver may be authorized to submit state-changing transactions.

The design therefore validates authorization and relevant on-chain state before attempting a state transition.

No signing keys, production credentials, internal wallet addresses, or proprietary implementation details are included in this repository.

## Engineering Takeaways

### Event-driven processing reduces unnecessary reads
Subscribing to relevant accounts allows the service to react to state changes without continuously polling the entire protocol.

### Idempotency is a core blockchain requirement
Duplicate observations are normal. Safe automation must make repeated delivery harmless.

### Validate immediately before signing
Authorization and state checks should happen as close as possible to the state-changing operation.

### Separate observation from decisions
Protocol parsing, normalized state, and resolver decisions should remain distinct. This makes the system easier to test, reason about, and replace.

## Public / Private Boundary

This repository is a documentation-only case study. It does not publish oracle private keys, production RPC credentials, proprietary program source, sensitive internal wallet addresses, client-specific infrastructure, or private database contents.

The original implementation remains private.

## Documentation

- [Architecture](docs/architecture.md)
- [Engineering Decisions](docs/engineering-decisions.md)
- [Security & Public Disclosure Boundary](docs/security-and-boundaries.md)
