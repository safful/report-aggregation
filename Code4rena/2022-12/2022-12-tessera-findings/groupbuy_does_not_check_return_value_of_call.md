## Tags

- bug
- 3 (High Risk)
- disagree with severity
- primary issue
- selected for report
- sponsor confirmed
- H-01

# [GroupBuy does not check return value of call](https://github.com/code-423n4/2022-12-tessera-findings/issues/6) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/1e408ebc1c4fdcc72678ea7f21a94d38855ccc0b/src/modules/GroupBuy.sol#L265
https://github.com/code-423n4/2022-12-tessera/blob/1e408ebc1c4fdcc72678ea7f21a94d38855ccc0b/src/modules/GroupBuy.sol#L283


# Vulnerability details

## Impact
Both usages of `call` do not check if the transfer of ETH was succesful:
```solidity
payable(msg.sender).call{value: contribution}("");
...
payable(msg.sender).call{value: balance}("");
```
This can become very problematic when the recipient is a smart contract that reverts (for instance, temporarily) in its `receive` function. Then, `GroupBuy` still assumes that this ETH was transferred out and sets the balance to 0 or deletes `userContributions[_poolId][msg.sender]`, although no ETH was transferred. This leads to a loss of funds for the recipient.

## Proof Of Concept
We assume that the recipient is a smart contract that performs some logic in its `receive` function. For instance, it can be a nice feature for some people to automatically convert all incoming ETH into another token using an AMM. However, it can happen that the used AMM has too little liquidity at the moment or the slippage of a swap would be too high, leading to a revert in the receing contract. In such a scenario, the `GroupBuy` contract still thinks that the call was succesful, leading to lost funds for the recipient.

## Recommended Mitigation Steps
`require` that the call was succesful.