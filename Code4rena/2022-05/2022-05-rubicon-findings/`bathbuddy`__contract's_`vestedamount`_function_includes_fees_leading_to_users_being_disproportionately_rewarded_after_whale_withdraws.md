## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`BathBuddy`  contract's `vestedAmount` function includes fees leading to users being disproportionately rewarded after whale withdraws](https://github.com/code-423n4/2022-05-rubicon-findings/issues/191) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L103-L104
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L133


# Vulnerability details


## Impact

When a whale withdraws their tokens and receives rewards from the `BathBuddy` contract the fees they pay will erroneously become part of the calculation performed in function `vestedAmount`. This means that any subsequent withdrawer of funds may receive a disproportionate amount of tokens. The fees paid by a whale could still be much larger than the amount of tokens invested by a minnow.

Althought similar to the issue "When `BathToken` contract is recipient of fees then users can make disproportionate returns after whales withdraw" it is not the same issue since fees are always accrued in the `BathBuddy` contract and this cannot be changed. Also, the calculations in are subtly different

However, the outcome is the same. A minnow can receive a disproportionate reward and drain much of the fees from the contract.

The intention of setting the pool as the recipient of the fees was to reward HODLers but, in fact, they will be incentivised to withdraw after a whale does.

## Proof of Concept

Consider the following scenario.

1. fee is set to 50 BPS (i.e. 0.50%)
2. A whale deposits 200 tokens
3. A minnow deposits 0.01 tokens
4. A `BathBuddy` contract is set up for the `BathToken` contract.
5. The whale withdraws their funds
6. The minnow then withdraws their funds

After step 5, the function `vestedAmount` will return a value that includes the fees paid by the whale. This is because the `BathBuddy` contract is the recipient of all fees. They are not transferred anywhere.

Thus, when the minnow withdraws their funds `releasable` is much larger than the amount they otherwise would have expected. Further [sharesWithdrawn](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L103) is equal to [initialTotalSupply](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L104) in this particular scenario so `mul(sharesWithdrawn).div(initialTotalSupply)` evaluates to `1`. This means that `amount = releaseable`.

A [test](https://github.com/sseefried/codearena-rubicon-2022-05/blob/f5010d845d3713b07a00f3bb96a5608c6d09b047/test/BugsBathBuddy.js#L55-L145) has been written in the private fork that exhibits this behaviour.

## Tools Used

Manual inspection

## Recommended Mitigation Steps

Keep a tally of the fees accrued in a separate variable and work out a fairer system for distributing rewards to HODLers.

