## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Call to swapExactTokensForETH in liquidateDai() will always fail ](https://github.com/code-423n4/2021-05-fairside-findings/issues/21) 

# Handle

0xRajeev


# Vulnerability details

## Impact

liquidateDai() calls Uniswap’s swapExactTokensForETH to swap Dai to ETH. This will work if msg.sender, i.e. FSD contract, has already given the router an allowance of at least amount on the input token Dai. 

Given that there is no prior approval, the call to UniswapV2 router for swapping will fail because msg.sender has not approved UniswapV2 with an allowance for the tokens being attempted to swap.

The impact is that updateCostShareRequest() will fail and revert while working with stablecoin Dai.

## Proof of Concept

https://uniswap.org/docs/v2/smart-contracts/router02/#swapexacttokensfortokens

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/token/FSD.sol#L191

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/token/FSD.sol#L182-L198

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/network/FSDNetwork.sol#L323

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/network/FSDNetwork.sol#L307-L329

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/network/FSDNetwork.sol#L280

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/network/FSDNetwork.sol#L297


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add FSD approval to UniswapV2 with an allowance for the tokens being attempted to swap.

