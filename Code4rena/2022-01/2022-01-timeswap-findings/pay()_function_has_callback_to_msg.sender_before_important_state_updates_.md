## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [pay() function has callback to msg.sender before important state updates ](https://github.com/code-423n4/2022-01-timeswap-findings/issues/7) 

# Handle

jayjonah8


# Vulnerability details

## Impact
In TimeswapPair.sol, the pay() function has a callback to the msg.sender in the middle of the function while there are still updates to state that take place after the callback.  The lock modifier guards against reentrancy but not against cross function reentrancy.  Since the protocol implements Uniswap like functionality,  this can be extremely dangerous especially with regard to composability/interacting with other protocols and contracts.  The callback before important state changes (updates to reserves collateral and reserves assets) also violates the Checks Effects Interactions best practices further widening the attack surface. 

## Proof of Concept
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L369

https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html

cross function reentrancy
https://medium.com/coinmonks/protect-your-solidity-smart-contracts-from-reentrancy-attacks-9972c3af7c21

## Tools Used
Manual code review 

## Recommended Mitigation Steps
The callback "if (assetIn > 0) Callback.pay(asset, assetIn, data);"  should be placed at the end of the pay() function after all state updates have taken place.


