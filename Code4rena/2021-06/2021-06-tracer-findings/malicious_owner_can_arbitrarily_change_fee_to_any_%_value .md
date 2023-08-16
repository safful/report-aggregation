## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Malicious owner can arbitrarily change fee to any % value ](https://github.com/code-423n4/2021-06-tracer-findings/issues/66) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Tracer protocol like any other allows market creators to charge fees for trades. However, a malicious/greedy owner can arbitrarily change fee to any % value and without an event to observe this change or a timelock to react, there is no easy way for users to monitor this via front-end or off-chain monitoring tools.

Impact: Users trade on a market with 0.1% fees. The owner suddenly changes this to 100%. Users realise this only after their trades are executed. Market loses confidence. Protocol takes a reputational hit.

## Proof of Concept

See similar Medium-severity finding in ConsenSys's Audit of 1inch Liquidity Protocol (https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unpredictable-behavior-for-users-due-to-admin-front-running-or-general-bad-timing

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/TracerPerpetualSwaps.sol#L548-L550

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/lib/LibBalances.sol#L198-L214

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Emit event, provide time lock for users to react and establish an upper threshold for fees that is decided across markets by governance.

