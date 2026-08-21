# Ritual Technical Review

## Scheduler

The Scheduler is the component that makes the market self-resolving.

Instead of requiring a backend service to call the contract at the correct
time, the contract schedules future executions.

This removes the dependency on a centralized keeper.

## Scheduled Attempts

The market schedules multiple attempts rather than relying on a single
execution.

The attempts are separated by a fixed number of blocks.

This gives the external execution path an opportunity to recover from a
temporary failure.

## Executor Selection

The contract does not permanently hardcode one executor.

At resolution time it requests an executor capable of performing the required
HTTP operation.

This is an important architectural decision.

It allows the execution environment to select an available service rather
than making the market depend on one fixed executor.

## HTTP Precompile

The HTTP precompile provides access to external HTTP data.

The prediction market uses this capability to retrieve information from the
configured oracle endpoint.

The contract itself does not implement an HTTP client.

Instead, it relies on the Ritual execution environment.

## JSON Extraction

The raw HTTP response is not necessarily the value required by the market.

The jq precompile extracts a specific value from the response.

The market therefore stores both:

- the oracle URL
- the JSON path

This allows the resolution rule to identify the required field.

## Comparison

After extracting the value, the contract compares it with the target.

The comparator determines whether the YES or NO outcome wins.

This keeps the final prediction rule inside the contract.

## Failure Handling

External execution can fail for many reasons.

The contract must distinguish between:

- successful response
- executor failure
- HTTP failure
- invalid response
- invalid JSON
- missing data

A failure should not be interpreted as a valid prediction.

## Retry Behavior

The Scheduler provides multiple opportunities to resolve the market.

A successful attempt should prevent unnecessary later executions.

The remaining scheduled calls can be cancelled after successful resolution.

## Idempotency

The callback should be safe against repeated execution.

This matters because scheduled calls can exist around the same resolution
window.

A repeated callback should not create a second payout or modify an already
resolved market.

## Block-Based Deadlines

The project uses block numbers for important timing decisions.

This is preferable to depending on timestamp assumptions when the Scheduler
itself operates around blocks.

The market converts the user-facing duration into a block-based resolution
point.

## Financial Safety

The market uses native RITUAL as the staking asset.

The payout calculation is performed when a user claims winnings.

This avoids a settlement transaction that has to iterate through every
participant.

## Pull-Based Claims

A pull-based claim system has several advantages.

Each user pays the gas required to claim their own reward.

The settlement transaction does not need to know the number of participants.

This makes the system more scalable than pushing rewards to every address.

## Invalid Markets

A market can become invalid when external resolution cannot produce a valid
outcome.

In that case the system should prioritize returning user funds instead of
creating an arbitrary result.

## Design Tradeoffs

The design intentionally avoids several features.

It does not attempt to implement:

- an order book
- an AMM
- governance
- a separate ERC-20 token
- a centralized resolver
- an upgrade proxy

This keeps the workshop focused on autonomous resolution.

## Main Learning

The most important lesson from the workshop is that smart contract
autonomy depends on infrastructure beyond the EVM contract itself.

Scheduler execution, trusted execution, HTTP access, and structured data
extraction form one complete resolution pipeline.

The prediction market is therefore a useful example of how Ritual Chain can
connect deterministic on-chain rules with external information.
