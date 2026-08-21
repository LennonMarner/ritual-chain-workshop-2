# Testing Notes

## Testing Goals

The test suite should verify the complete lifecycle of a prediction market.

The main goal is to make sure that a market behaves correctly under both
normal and abnormal conditions.

## Market Creation

The first test category covers market creation.

A newly created market should contain the parameters supplied by the creator.

The test should verify:

- target value
- comparator
- oracle URL
- JSON path
- resolution block
- initial market state

## Betting

The second category covers user betting.

Users should be able to select either YES or NO.

The amount supplied by the user should be reflected in the corresponding pool.

Tests should verify that the total YES and NO pools change correctly.

## Deadline

Betting should stop after the configured deadline.

This is an important property because the resolution block is fixed when the
market is created.

Allowing bets after the deadline would make the settlement process unfair.

## Resolution

The resolution path is the most important part of the workshop.

The test design should distinguish between:

- successful oracle response
- failed oracle request
- invalid response
- malformed data
- unsuccessful executor response

A failed oracle request should not automatically become a NO outcome.

## Invalid Resolution

If the resolution process cannot determine a valid result after the allowed
attempts, the market should enter an invalid state.

Participants should then be able to recover their original stake according
to the contract rules.

## Reward Calculation

Winning users should receive a proportional share of the winning pool.

The calculation should depend on:

- user's winning stake
- total market pool
- total winning pool

The test suite should verify that users cannot claim more than once.

## Access Control

Administrative and callback functions should be tested against unauthorized
callers.

A user should not be able to perform privileged actions simply by calling
the contract directly.

## Events

Events are useful for verifying state transitions.

Tests should check important events related to:

- market creation
- betting
- scheduling
- resolution
- invalid markets
- claims

## Regression Testing

Whenever the contract is modified, existing tests should continue to pass.

A regression test suite provides confidence that a new feature has not
silently changed the behavior of an existing market.

## Future Testing

A stronger test suite could add:

- randomized betting amounts
- multiple participants
- multiple markets
- boundary block numbers
- zero-value edge cases
- oracle failure simulations
- repeated callback attempts
- gas usage measurements

## Conclusion

Testing this project requires thinking about both financial state and
external execution.

The most important principle is that external data failure must remain
different from a valid negative prediction.

That distinction is essential for a self-resolving prediction market.
