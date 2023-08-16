## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Redundant call to external contract, result can be saved](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/43) 

# Handle

kenzo


# Vulnerability details

When using few times an unchanging value from external contract call, the result can be saved and used without recalling the external contract.

## Impact
Some gas can be saved.

## Proof of Concept
In settleAuction, the basket's totalSupply stays constant through the loop's iterations.
```
for (uint256 i = 0; i < pendingWeights.length; i++) {
            uint256 tokensNeeded = basketAsERC20.totalSupply() * pendingWeights[i] * newRatio / BASE / BASE;
            require(IERC20(pendingTokens[i]).balanceOf(address(basket)) >= tokensNeeded);
        }
```
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L97

## Tools Used
Manual analysis, hardhat

## Recommended Mitigation Steps
Save basketAsERC20.totalSupply() to a local variable outside the loop, and use that variable inside the loop.

