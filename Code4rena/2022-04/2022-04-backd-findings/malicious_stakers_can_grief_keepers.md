## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- reviewed

# [Malicious Stakers can grief Keepers](https://github.com/code-423n4/2022-04-backd-findings/issues/194) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L727-L729


# Vulnerability details

## Impact

A Staker -- that has their top-up position removed after `execute` is called by a Keeper -- can always cause the transaction to revert. They can do this by deploying a smart contract to the `payer` address that has implemented a `receive()` function that calls `revert()`. The revert will be triggered by the following [lines](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L727-L729) in `execute`

```sol
if (vars.removePosition) {
    gasBank.withdrawUnused(payer);
}
```

This will consume some gas from the keeper while preventing them accruing any rewards for performing the top-up action.

## Proof of Concept

I have implemented a [PoC](https://github.com/sseefried/codearena-backd-2022-04/blob/4d3c3ba7a0139bea01a0bdee9e84a7921572a9fd/backd/tests/top_up_action/sseefried_test_staker_grief.py) in a fork of the contest repo. The attacker's contract can be found [here](https://github.com/sseefried/codearena-backd-2022-04/blob/4d3c3ba7a0139bea01a0bdee9e84a7921572a9fd/backd/contracts/AliceAttacker.sol).

## Tools Used
Manual inspection

## Recommend Mitigation Steps

To prevent this denial of service attack some way of blacklisting badly behaved Stakers should be added.


