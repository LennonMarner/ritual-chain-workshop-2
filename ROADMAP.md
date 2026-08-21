# Development Roadmap

## Phase 1 - Understand

The first phase is understanding the existing implementation.

Goals:

- read the main contract
- identify market state
- identify betting logic
- identify resolution logic
- identify claim logic
- understand Scheduler interaction
- understand HTTP execution
- understand jq extraction

## Phase 2 - Verify

The second phase is verification.

Goals:

- compile the project
- run the existing tests
- inspect deployment configuration
- inspect mock contracts
- verify important state transitions
- document observed behavior

## Phase 3 - Improve Tests

The next phase focuses on testing.

Planned tests:

- market creation
- YES betting
- NO betting
- deadline enforcement
- successful resolution
- failed resolution
- invalid market
- refunds
- reward claims
- duplicate claims

## Phase 4 - Improve Developer Experience

The project can be made easier to understand by improving documentation.

Planned improvements:

- setup guide
- architecture explanation
- testing guide
- troubleshooting guide
- contract comments
- deployment notes

## Phase 5 - Frontend

A lightweight frontend could provide:

- market creation
- market discovery
- current pool size
- deadline information
- current resolution state
- betting
- reward claiming

The frontend should remain a thin client.

The contract remains responsible for all important financial rules.

## Phase 6 - Market Analytics

A future analytics layer could calculate:

- total volume
- YES percentage
- NO percentage
- number of participants
- historical outcomes
- average stake

These values could be derived from emitted events.

## Phase 7 - Multiple Oracle Sources

A more advanced version could support different oracle endpoints.

Each market could specify a different data source while preserving the same
resolution architecture.

This would make the market more general-purpose.

## Phase 8 - Additional Comparators

Future versions could support additional comparison operators.

Examples include:

- greater than
- greater than or equal
- less than
- less than or equal
- equal
- not equal

This would allow more types of prediction questions.

## Phase 9 - Failure Simulation

A local development environment could simulate:

- HTTP failure
- invalid JSON
- missing JSON path
- executor failure
- repeated callback
- unavailable oracle

This would make failure handling easier to validate.

## Phase 10 - Production Readiness

Before production deployment, the project should receive:

- security review
- gas analysis
- integration testing
- oracle reliability review
- Scheduler review
- edge case testing
- documentation review

## Final Goal

The final goal is a prediction market that is easy to understand,
deterministic on-chain, and capable of resolving without a centralized
operator.

The workshop provides the foundation for this architecture.

Future work should improve usability and testing without weakening the
autonomous resolution model.
