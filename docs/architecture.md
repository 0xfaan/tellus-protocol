# Tellus Protocol Architecture

Tellus Protocol is a Soroban prototype made of four contracts: pool, policy, oracle, and trigger. The contracts are currently tested together in Rust, but full production payout orchestration is not implemented.

## Contract Overview

```text
Farmer address ---- registers policy ----> Policy contract
LP address -------- deposits capital ----> Pool contract
Oracle submitter --- submits reading ----> Oracle contract
Caller ------------ simulated trigger ---> Trigger contract
```

## Pool Contract

Purpose: record liquidity provider capital, shares, coverage locks, and simulated payout releases.

Implemented methods:

- `initialize(admin, stablecoin_asset, min_collateral_ratio)`
- `deposit(provider, amount)`
- `withdraw(provider, shares)`
- `lock_coverage(policy_id, amount)`
- `release_payout(policy_id, farmer, amount)`
- `get_pool_stats()`
- `get_provider_shares(provider)`
- `get_provider_value(provider)`

Important limitation: the pool contract updates accounting values only. It does not transfer a Stellar token when deposits, withdrawals, or payouts occur.

## Policy Contract

Purpose: store farmer policy records.

Implemented methods:

- `initialize(admin, pool_contract)`
- `register_policy(farmer, farm_geohash, crop_type, coverage_amount, rainfall_threshold)`
- `get_policy(policy_id)`
- `list_policies_by_farmer(farmer)`
- `update_policy_state(policy_id, new_state)`

Current registration behavior:

- Requires farmer authorization.
- Rejects non-positive coverage amounts.
- Sets `season_start` to the current ledger timestamp.
- Sets `season_end` to 90 days after `season_start`.
- Sets `ndvi_baseline` to `0`.
- Sets state to `Active`.

## Oracle Contract

Purpose: store latest readings by location and reading type.

Implemented methods:

- `initialize(admin)`
- `submit_reading(geo_cell, reading_type, value)`
- `get_latest(geo_cell, reading_type)`

Important limitation: the oracle contract does not authenticate submitters, verify signatures, keep history, or aggregate multiple readings.

## Trigger Contract

Purpose: record a trigger event when simulated rainfall is below a simulated threshold.

Implemented methods:

- `initialize(admin, policy_contract, oracle_contract, pool_contract)`
- `evaluate_policy(policy_id, simulated_rainfall, simulated_threshold)`
- `get_trigger_event(policy_id)`
- `is_triggered(policy_id)`

Current trigger behavior:

- Rejects a policy that has already been triggered.
- Compares the supplied rainfall value with the supplied threshold.
- Stores a trigger event when rainfall is below threshold.
- Uses a hardcoded payout amount.

Important limitation: the trigger contract does not read policy data, read oracle data, update policy state, call the pool, or transfer funds.

## Storage Model

The contracts use Soroban typed storage keys.

Instance storage:

- Contract configuration.
- Pool totals.
- Policy ID counter.

Persistent storage:

- Provider positions.
- Policy records.
- Farmer policy lists.
- Latest oracle readings.
- Trigger events.
- Coverage locks.

## Security Notes

Implemented safeguards:

- Farmer authorization is required to register a policy.
- Liquidity providers authorize deposits and withdrawals.
- Contract initialization can only happen once.
- Invalid or non-positive amounts are rejected in core pool and policy paths.
- Duplicate trigger events for the same policy are rejected.

Known gaps:

- No token transfer integration.
- No oracle access control.
- No oracle data verification.
- No cross-contract authorization for pool payout release.
- No premium calculation or premium collection.
- No security audit.

## Development Guidance

Keep architecture documentation aligned with code. When adding cross-contract calls, token transfers, oracle validation, or payout automation, update this document in the same change.
