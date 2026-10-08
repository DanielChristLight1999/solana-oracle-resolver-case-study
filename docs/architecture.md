# Architecture Notes

## Processing Pipeline

```text
Protocol Account Change
        │
        ▼
      Parse
        │
        ▼
    Validate
        │
        ▼
  Normalize State
        │
        ▼
 Resolver Decision
        │
        ▼
Pre-Transaction Checks
        │
        ▼
     Submit
        │
        ▼
Record / Observe Result
```

Each stage has a distinct responsibility.

### Parse
Convert raw account observations into application-level values.
### Validate
Reject incomplete, malformed, stale, unauthorized, or otherwise invalid observations.
### Normalize
Represent protocol state in a predictable internal model so decision logic does not depend directly on raw account layouts.
### Decide
Determine whether a valid state transition should occur.
### Pre-transaction checks
Verify expected market/range state and oracle authorization immediately before a state-changing transaction.
### Submit
Create and submit the blockchain transaction.
### Record / observe
Track resulting state so duplicate observations do not trigger duplicate actions.

## Range Closure

```text
Treasury update → Normalize value → Find surpassed ranges
        → Validate range state → Already processed?
        → Submit close → Record
```

## Market Resolution

```text
Round update → Motherlode hit? → Market exists?
        → Already resolved? → Oracle authorized?
        → Calculate winning range → Range valid?
        → Submit resolution → Record resolved state
```

## Why Subscription-Driven Processing

The resolver reacts to protocol accounts relevant to its decisions.

Compared with blind polling, the intended relationship is:

`Protocol change → observation → decision`

rather than:

`Timer → query everything → compare everything → maybe act`

This reduces unnecessary reads and makes the trigger for a decision easier to reason about.