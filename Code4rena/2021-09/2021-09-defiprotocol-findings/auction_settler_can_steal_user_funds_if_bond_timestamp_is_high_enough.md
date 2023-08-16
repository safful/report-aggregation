## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Auction settler can steal user funds if bond timestamp is high enough](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/45) 

# Handle

kenzo


# Vulnerability details

After an auction has started, as time passes and according to the bondTimestamp, newRatio (which starts at 2*ibRatio) gets smaller and smaller and therefore less and less tokens need to remain in the basket.
This is not capped, and after a while, newRatio can become smaller than current ibRatio.

## Impact
If for some reason nobody has settled an auction and the publisher didn't stop it, a malicious user can wait until newRatio < ibRatio, or even until newRatio ~= 0 (for an initial ibRatio of ~1e18 this happens after less than 3.5 days after auction started), and then bond and settle and steal user funds.

## Proof of Concept
These are the vulnerable lines:
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L89:#L99
```
        uint256 a = factory.auctionMultiplier() * basket.ibRatio();
        uint256 b = (bondTimestamp - auctionStart) * BASE / factory.auctionDecrement();
        uint256 newRatio = a - b;

        for (uint256 i = 0; i < pendingWeights.length; i++) {
            uint256 tokensNeeded = basketAsERC20.totalSupply() * pendingWeights[i] * newRatio / BASE / BASE;
            require(IERC20(pendingTokens[i]).balanceOf(address(basket)) >= tokensNeeded);
        }
```
The function verifies that ```pendingTokens[i].balanceOf(basket) >= basketAsERC20.totalSupply() * pendingWeights[i] * newRatio / BASE / BASE```. This is the formula that will be used later to mint/burn/withdraw user funds.
As bondTimestamp increases, newRatio will get smaller, and there is no check on this.
After a while we'll arrive at a point where ```newRatio ~= 0```, so ```tokensNeeded = newRatio*(...) ~= 0```, so the attacker could withdraw nearly all the tokens using outputTokens and outputWeights, and leave just scraps in the basket.

## Tools Used
Manual analysis, hardhat.

## Recommended Mitigation Steps
Your needed condition/math might be different, and you might also choose to burn the bond while you're at it, but I think at the minimum you should add a sanity check in settleAuction:
```
require (newRatio > basket.ibRatio());
```

