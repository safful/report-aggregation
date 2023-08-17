## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [BathBuddy locks up Ether it receives](https://github.com/code-423n4/2022-05-rubicon-findings/issues/78) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/peripheral_contracts/BathBuddy.sol#L69


# Vulnerability details

## Impact
The BathBuddy contract is able to receive ETH. But, there's no way of ever retrieving that ETH from the contract. The funds will be locked up.

Currently, there seems to be no logic in the protocol where ETH is sent to the contract. But, it might happen in the future. So I'd say it's a MED issue.

## Proof of Concept
`receive()`  function: https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/peripheral_contracts/BathBuddy.sol#L69

## Tools Used
none

## Recommended Mitigation Steps
Remove the `receive()` function if the contract isn't supposed to handle ETH. Otherwise, add the necessary logic to release the ETH it gets.

