## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Floating pragma used in Uniswap*.sol ](https://github.com/code-423n4/2021-04-basedloans-findings/issues/19) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the floating pragma, i.e. by not using ^ in pragma solidity ^0.6.10, ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

For reference, see https://swcregistry.io/docs/SWC-103


## Proof of Concept

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/UniswapOracle/UniswapAnchoredView.sol#L3

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/UniswapOracle/UniswapConfig.sol#L3

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/UniswapOracle/UniswapLib.sol#L3


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Remove ^ in “pragma solidity ^0.6.10” and change it to “pragma solidity 0.6.12” to be consistent with the rest of the contracts.

