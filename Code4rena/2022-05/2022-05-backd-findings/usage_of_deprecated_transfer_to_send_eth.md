## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Usage of deprecated transfer to send ETH](https://github.com/code-423n4/2022-05-backd-findings/issues/180) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/swappers/SwapperRouter.sol#L140
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/swappers/SwapperRouter.sol#L280


# Vulnerability details

## Impact
  Usage of deprecated transfer  Swap can revert. 

## Proof of Concept

The original `transfer` used to send eth uses a fixed stipend 2300 gas.   This was used to prevent reentrancy.   However this limit your protocol to interact with others contracts that need more than that to proceess the transaction
A good article about that
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

## Recommended Mitigation Steps

Used call instead.  For example

        (bool success, ) = msg.sender.call{amount}("");
        require(success, "Transfer failed.");

