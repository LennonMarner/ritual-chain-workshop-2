# Development Notes

## Workshop Objective

The goal of this workshop is to understand how a self-resolving prediction
market can be implemented on Ritual Chain.

The important part of the project is not only the prediction market itself.
The interesting part is the automated resolution mechanism.

The contract combines several Ritual Chain capabilities:

- Scheduler
- HTTP precompile
- jq precompile
- TEE service selection
- Native RITUAL staking
- Automatic resolution

## Market Lifecycle

A market follows a predictable lifecycle.

1. A user creates a market.
2. The contract records the resolution parameters.
3. Users place YES or NO bets.
4. The contract schedules future resolution attempts.
5. The Scheduler calls the contract.
6. The contract selects an HTTP-capable TEE executor.
7. The executor retrieves the configured oracle response.
8. The jq precompile extracts the required value.
9. The contract compares the observed value with the target.
10. The market becomes resolved.
11. Winners can claim their proportional reward.

## Resolution Design

The resolution parameters are fixed when a market is created.

Important parameters include:

- target
- comparator
- oracle URL
- JSON path
- resolution block

Keeping these parameters immutable makes the resolution process easier to
reason about.

A market should not be able to silently change its oracle or target after
users have already deposited funds.

## Scheduler Design

The Scheduler is responsible for waking the contract at a predetermined block.

The contract does not depend on a centralized backend cron job.

The initial schedule contains multiple future attempts.

This design provides retry behavior when an oracle request fails.

If an attempt succeeds, remaining scheduled calls can be cancelled.

If all attempts fail, the market can become invalid and users can receive
their stakes back.

## Oracle Failure

An oracle failure must not automatically become a NO result.

Examples of failure include:

- HTTP request failure
- non-success HTTP response
- invalid response data
- malformed JSON
- missing JSON path
- executor error

Treating these cases as invalid resolution protects users from incorrect
settlement.

## Payout Model

The project uses a pull-based payout model.

Users do not receive rewards automatically through a loop over all
participants.

Instead, each participant calls the claim function independently.

This reduces the amount of work required by the settlement transaction.

The payout calculation is proportional to the user's winning stake.

## Invalid Markets

A market can become invalid when a valid winning side cannot be determined.

For example, if nobody backed the winning side, there is no meaningful
pari-mutuel denominator.

In that situation the project can refund participants instead of distributing
an undefined reward.

## Local Development

The repository can be studied without relying on a live Ritual Chain.

A local Hardhat environment is useful for understanding:

- contract deployment
- market creation
- betting
- resolution state
- invalid state
- reward claims
- event emission

## Testing Philosophy

Tests should cover both successful and unsuccessful paths.

Important cases include:

- successful market creation
- valid YES bet
- valid NO bet
- betting after the deadline
- successful resolution
- failed oracle resolution
- invalid market
- reward claim
- repeated reward claim
- unauthorized actions

## Security Considerations

Prediction markets handle user funds, so state transitions should be explicit.

The contract should make sure that:

- a market cannot be resolved twice
- a user cannot claim twice
- betting cannot continue after the deadline
- resolution parameters cannot be changed unexpectedly
- invalid markets return user funds correctly

## Future Improvements

Possible future extensions include:

- a browser-based interface
- richer market statistics
- event indexing
- additional comparison operators
- multiple oracle formats
- improved error messages
- automated integration tests

## Personal Learning

This workshop helped me understand that autonomous smart contracts require
more than ordinary contract logic.

The Scheduler, TEE execution environment, HTTP access, and data extraction
pipeline all need to work together.

The most interesting part of the project is the separation between market
logic and external data retrieval.

That separation makes the prediction market more autonomous while keeping
the settlement rules inside the contract.
