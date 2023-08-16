## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Bypass or reduction on the lockup period of Pool FDTs.](https://github.com/code-423n4/2021-04-maple-findings/issues/117) 

# Handle

shw


# Vulnerability details

** Editing on a previous submission to clarify more details **

## Impact

In `Pool.sol`, the lockup restriction of withdrawal (`Pool.sol#396`) can be bypassed or reduced if new liquidity providers cooperate with existing ones.

## Proof of Concept

1. A liquidity provider, Alice, deposits liquidity assets into the pool and minted some FDTs. She then waits for `lockupPeriod` days and calls `intendToWithdraw` to pass her withdrawal window. Now she is available to receive FDTs from others.
2. A new liquidity provider, Bob, deposits liquidity assets into the pool and minted some FDTs. Currently, he is not allowed to withdraw his funds by protocol design.
3. Bob and Alice agree to cooperate with each other to reduce Bob's waiting time for withdrawal. Bob transfers his FDT to Alice via the `_transfer` function.
4. Alice calls `intendToWithdraw` and waits for the `withdrawCooldown` period. Notice that Alice's `depositDate` is updated after the transfer; however, since it is calculated using a weighted timestamp, the increased amount of lockup time should be less than `lockupPeriod`. In situations where the deposit from Alice is much larger than that from Bob, Alice could only even need to wait for the `withdrawCooldown` period before she could withdraw any funds.
5. Alice then withdraws the amount of FDT that Bob transferred to her and transfers the funds (liquidity assets) to Bob. Bob successfully reduces (or bypasses) the lockup period of withdrawal.

## Tools Used

None

## Recommended Mitigation Steps

Force users to wait for the lockup period when transferring FDT to others. Or let the `depositDate` variable record the timestamp of the last operation instead of a weighted timestamp.

