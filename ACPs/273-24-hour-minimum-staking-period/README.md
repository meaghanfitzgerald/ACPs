| ACP | 273 |
| :--- | :--- |
| **Title** | Reduce Minimum Validator Staking Period to 24 Hours |
| **Author(s)** | Meaghan FitzGerald ([@meaghanfitzgerald](https://github.com/meaghanfitzgerald)) |
| **Status** | Proposed |
| **Track** | Standards |

## Abstract

This proposal reduces the minimum validator staking period on Avalanche's Primary Network from 2 weeks (336 hours) to 1 day (24 hours). The change lowers barriers to validator participation while maintaining network security, enabling more flexible staking strategies and improving capital efficiency across the ecosystem.

## Motivation

### Current State

The Avalanche Primary Network currently requires validators and delegators to stake for a minimum of 2 weeks (336 hours) to be eligible for staking rewards. While this ensures validator commitment to network security, it creates friction for participants who require more flexible liquidity access.

### Benefits of a Shorter Minimum Period

1. Increased Validator Participation: A 24-hour minimum removes a significant barrier to entry. Validators who cannot commit capital for 2 weeks can now participate, increasing overall network stake and decentralization.

2. Improved Capital Efficiency: Stakers gain reliable access to liquidity after 24 hours rather than 14 days. This enables more dynamic capital allocation strategies while maintaining commitment to network security during the staking period.

### Reward Impact Analysis

Using Avalanche's reward formula at the time of writing, the APY difference between 24-hour and 2-week staking periods is approximately 0.04 percentage points (6.08% vs 6.12%). This minimal difference preserves reward incentives while enabling liquidity flexibility.

## Specification

### Technical Changes

Update the minimum staking duration by modifying `MinStakeDuration` in the genesis configuration:

Current:
```go
MinStakeDuration = 336 * time.Hour
```

Proposed:
```go
MinStakeDuration = 24 * time.Hour
```

### Implementation Details

The staking mechanism remains unchanged. Validators must still meet all existing requirements:

- Minimum stake: 2,000 AVAX for Primary Network validators
- Uptime requirement: 90% (per ACP-267)
- Hardware requirements: As specified in ACP-256

Only the minimum duration parameter changes, with all other validation rules and reward calculations preserved.

## Backwards Compatibility

This change affects only new staking periods initiated after activation. Existing validators with active staking periods remain unaffected and will complete their original durations under the previous rules.

The change is non-breaking: all existing validation infrastructure, reward mechanisms, and consensus operations continue functioning without modification.

## Security Considerations

Validators remain subject to the same accountability standards during the 24-hour period. Network consensus sampling assumes the same validator availability model. Historical data demonstrates that validator uptime patterns remain consistent regardless of staking duration length, as infrastructure quality and operational commitment drive uptime, not duration requirements alone.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
