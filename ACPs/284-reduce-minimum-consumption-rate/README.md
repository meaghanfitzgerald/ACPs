| ACP | 284 |
| :--- | :--- |
| **Title** | Reduce Minimum Consumption Rate |
| **Author(s)** | Martin Eckardt ([@martineckardt](https://github.com/martineckardt)), Meaghan FitzGerald ([@meaghanfitzgerald](https://github.com/meaghanfitzgerald)), Eric Lu ([@ericlu-avax](https://github.com/ericlu-avax)) |
| **Status** | Proposed ([Discussion](https://github.com/avalanche-foundation/ACPs/discussions/XXX)) |
| **Track** | Standards |

## Abstract

This proposal reduces the `MinConsumptionRate` parameter governing Primary Network staking rewards from 10% to 7.5%. `MinConsumptionRate` sets the annualized reward rate for a validator whose stake duration approaches zero. Lowering it steepens the APY gradient between minimum-duration and maximum-duration staking — widening the spread from approximately 1.02 percentage points to 2.30 percentage points — without changing the maximum reward available to long-term validators. This change makes inflation-funded rewards more targeted, strengthening the economic incentive for validators to commit capital for longer periods and improving the long-term alignment between staking duration and network security.

## Motivation

Primary Network staking rewards follow a linear interpolation between `MinConsumptionRate` and `MaxConsumptionRate` based on a validator's stake duration relative to the `MintingPeriod` (365 days):

```
APY(d) = (RemainingSupply / CurrentSupply) × (MinRate + (MaxRate − MinRate) × d / MintingPeriod)
```

Where:
- `d` = validator stake duration in days
- `MinRate` = `MinConsumptionRate / PercentDenominator` = 10% (current)
- `MaxRate` = `MaxConsumptionRate / PercentDenominator` = 12% (unchanged)
- `MintingPeriod` = 365 days
- `RemainingSupply` = `SupplyCap − CurrentSupply`

This formula produces APY that increases linearly with duration. `MinConsumptionRate` sets the floor: the reward a validator receives if it stakes for the shortest possible period. `MaxConsumptionRate` sets the ceiling: the reward for a full 365-day commitment.

### Problem 1: A Flat Reward Curve Offers Weak Duration Incentives

At current network parameters (470,134,316 AVAX circulating, 720,000,000 AVAX supply cap), the APY schedule across the full staking duration range is:

| Duration | APY (MinRate = 10%) |
|----------|---------------------|
| 14 days (current minimum) | 5.36% |
| 90 days  | 5.58% |
| 182 days | 5.85% |
| 365 days | 6.38% |

A validator that stakes for a full year earns only 1.02 percentage points more than one that stakes for the minimum 14 days. This narrow spread means that the economic cost of choosing the shortest permissible staking duration is very small.

[ACP-236](https://github.com/avalanche-foundation/ACPs/blob/main/ACPs/236-auto-renewed-staking/README.md) compounds this further: by enabling automatic stake renewal, rolling 14-day positions become operationally equivalent to a perpetual stake. The on-chain convenience of auto-renewal makes the incremental yield from committing to longer durations even less compelling relative to the optionality of frequent roll periods.

[ACP-273](https://github.com/avalanche-foundation/ACPs/blob/main/ACPs/273-reduce-minimum-staking-duration/README.md) explicitly posed this question: *"Should the rewards rate differ for shorter vs. longer validation periods to incentivize network stability?"* This proposal answers affirmatively and provides the parameter change that implements that incentive.

### Problem 2: Undifferentiated Inflation Transfers Value Without Targeting Behavior

`MinConsumptionRate` at its current setting ensures that even a validator staking for the minimum duration receives a substantial base reward. This functions as untargeted inflation: a near-uniform subsidy paid regardless of the degree of commitment to network security.

As noted in a [recent public discussion](https://x.com/frostLedger/status/2039683238285713557) by the Avalanche Foundation: *"Inflation should be used surgically. To reward specific behaviors: uptime, performance, ecosystem contribution. Not as a blanket payment for passively existing on the network."*

The current `MinConsumptionRate` of 10% creates exactly the opposite: a floor that is nearly as valuable as the ceiling. Staking for 14 days currently earns at 84% of the reward rate a validator would earn for 365 days. Duration, the primary proxy for long-term commitment to network security, accounts for only 16% of the reward variation.

Reducing `MinConsumptionRate` to 7.5% preserves the maximum reward for long-term stakers while lowering the floor, expanding the portion of the reward that is earned through duration commitment.

### Problem 3: Inflation as a Security Subsidy Has an Expiration Date

Primary Network validators currently earn rewards almost entirely from token issuance against a fixed supply cap of 720,000,000 AVAX. As circulating supply grows, `RemainingSupply` shrinks and all APY values scale downward proportionally. When the supply cap is reached, inflation-funded rewards converge to zero.

This is a structural limitation of the current model: the security budget is finite and counting down. Given this constraint, the remaining reward issuance should be distributed with the intention of concentrating rewards to validators who provide the strongest security guarantees. Lowering the `MinConsumptionRate` moves in this direction.

### Proposed Fix: Lower the Floor, Keep the Ceiling

Reducing `MinConsumptionRate` from 10% to 7.5% produces the following APY schedule:

| Duration | APY (MinRate = 7.5%) | Compounding APY (MinRate) = 7.5% |
|----------|----------------------|----------------------------------|
| 2 days   | 4.00%                | 4.08%                            |
| 14 days  | 4.08%                | 4.16%                            |
| 90 days  | 4.58%                | 4.65%                            |
| 182 days | 5.18%                | 5.25%                            |
| 365 days | 6.38%                | 6.38%                            |

> All values calculated using live reward formula with current circulating supply of 470,134,316 AVAX and supply cap of 720,000,000 AVAX. Compound APY assumes the user utilizes ACP-236 auto-renewal for 365 days.

The maximum reward for a 365-day validator is unchanged. The incentive gradient more than doubles, going from a spread of 1.02 percentage points to 2.30 percentage points difference, an increase of over 126%.

## Specification

Modify `MinConsumptionRate` in the Primary Network genesis configuration for all network environments:

**`genesis/genesis_mainnet.go`**
```go
// Current
MinConsumptionRate: .10 * reward.PercentDenominator,

// Proposed
MinConsumptionRate: .075 * reward.PercentDenominator,
```

**`genesis/genesis_fuji.go`**
```go
// Current
MinConsumptionRate: .10 * reward.PercentDenominator,

// Proposed
MinConsumptionRate: .075 * reward.PercentDenominator,
```

### Unchanged Parameters

The following staking parameters are explicitly **not modified** by this proposal:

| Parameter | Value |
|-----------|-------|
| `MaxConsumptionRate` | 12% |
| `MinStakeDuration` | Governed separately by [ACP-273](https://github.com/avalanche-foundation/ACPs/blob/main/ACPs/273-reduce-minimum-staking-duration/README.md) |
| `MaxStakeDuration` | 365 days |
| Minimum validator stake | 2,000 AVAX |
| Minimum delegator stake | 25 AVAX |
| Uptime requirement | 90% (per [ACP-267](https://github.com/avalanche-foundation/ACPs/blob/main/ACPs/267-uptime-requirement-increase/README.md)) |
| Reward formula structure | Unchanged |

## Backwards Compatibility

This is a non-backwards compatible change to P-Chain reward calculation. It requires a network upgrade to activate.

The change affects only new staking periods initiated after activation. Validators and delegators with active staking positions at the time of activation continue to earn rewards under the original parameters for the remainder of their existing stake periods. No migration or re-registration is required.

## Reference Implementation

The complete change is a single floating-point literal update in three genesis files, as specified above. No changes are required to `vms/platformvm/reward/calculator.go`, `vms/platformvm/reward/config.go`, or any other reward logic.

The reward formula itself is not modified. The implementation does not introduce new code paths, state transitions, or P-Chain transaction types.

## Security Considerations

### Interaction With ACP-273 and ACP-236

This proposal is designed to work in conjunction with [ACP-273](https://github.com/avalanche-foundation/ACPs/blob/main/ACPs/273-reduce-minimum-staking-duration/README.md) and [ACP-236](https://github.com/avalanche-foundation/ACPs/blob/main/ACPs/236-auto-renewed-staking/README.md). ACP-273 reduces the minimum staking duration to 48 hours; ACP-236 enables zero-friction auto-renewal. Without a change to `MinConsumptionRate`, these two proposals together make minimum-duration rolling stakes economically near-equivalent to long-duration commitments (cost of choosing 2-day rolling vs. 90-day: ~0.50 pp). This proposal provides the complementary reward gradient that ACP-273's own Security Considerations section identified as potentially necessary.

If ACP-273 activates without this proposal, the combination of 48-hour minimum duration and auto-renewal represents a meaningful security regression. This proposal should be considered a companion to ACP-273.

## Open Questions

1. Is 7.5% the right value, or should `MinConsumptionRate` be reduced further?

   The 7.5% floor was selected to produce a meaningful but not extreme shift in the APY gradient. A lower floor (e.g., 5%) would widen the spread further, but might reduce minimum-duration validator participation to levels that negatively affect decentralization. Community input on the right balance is welcome.

2. Should `MaxConsumptionRate` change simultaneously?

   Reducing `MinConsumptionRate` without touching `MaxConsumptionRate` preserves the full reward for 365-day stakers. An alternative framing would reduce `MaxConsumptionRate` as well to slow total issuance, accepting lower returns for long-term stakers in exchange for a slower approach to the supply cap. This proposal does not pursue that direction but it is worth discussing.

3. Should this proposal activate concurrently with ACP-273?

   Given the Security Considerations above, simultaneous activation with ACP-273 is strongly preferred. The two proposals address the same underlying tension of short staking duration versus network stability, from complementary angles.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
