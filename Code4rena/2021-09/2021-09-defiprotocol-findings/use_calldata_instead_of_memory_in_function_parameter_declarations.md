## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use calldata instead of memory in function parameter declarations](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/75) 

# Handle

chasemartin01


# Vulnerability details

## Impact
Gas optimisation

## Example
As an example, you can change the declaration of `inputTokens`, `inputWeights`, `outputTokens`, `outputWeights` to be `calldata` as a gas optimisation

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Auction.sol#L69-L75

There's other instances of this in `Basket.sol` and`Factory.sol`
## Explanation
When you specify `memory` for a function param for an external function, the following happens: the compiler copies elements from `calldata` to `memory` (using the opcode `calldatacopy`.) Note that there is also the opcode `calldataload` to read an offset from `calldata`. By changing the location from `memory` to `calldata`, you avoid this expensive copy from `calldata` to `memory`, while managing to do exactly what's needed.

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Change all instances of `memory` to `calldata` where the function parameter isn't being modified

