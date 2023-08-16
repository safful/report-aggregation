## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Saving by changing the visibility of initialize function from public to externa in basket.sol](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/4) 

# Handle

jah


# Vulnerability details

## Impact
the initialize function will not be called from the contract and  it  doesn't require public visibility so the visibility should be changed to external to save gas as  described in https://mudit.blog/solidity-gas-optimization-tips/: “For all the public functions, the input parameters are copied to memory automatically, and it costs gas. If your function is only called externally, then you should explicitly mark it as external. External function’s parameters are not copied into memory but are read from calldata directly. This small optimization in your solidity code can save you a lot of gas when the function input parameters are huge.”   

## Proof of Concept
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L36

## Tools Used
manual analysis 
## Recommended Mitigation Steps
change the visibility to external

