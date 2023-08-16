## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Beneficiary cant get `fairSideConviction` NFT unless they only claim once, and only after it's fully vested](https://github.com/code-423n4/2021-11-fairside-findings/issues/62) 

# Handle

WatchPug


# Vulnerability details

Based on the context, once the beneficiary claimed all their vesting tokens, they should get the `fairSideConviction` NFT.

However, in the current implementation, if the beneficiary has claimed any amounts before it's fully vested, then they will never be able to get the `fairSideConviction` NFT, because at L138, it requires the `tokenbClaim` to be equal to the initial vesting amount.

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/token/FSDVesting.sol#L124-L142

```solidity=124
function claimVestedTokens() external override onlyBeneficiary {
    uint256 tokenClaim = calculateVestingClaim();
    require(
        tokenClaim > 0,
        "FSDVesting::claimVestedTokens: Zero claimable tokens"
    );

    totalClaimed = totalClaimed.add(tokenClaim);
    lastClaimAt = block.timestamp;

    fsd.safeTransfer(msg.sender, tokenClaim);

    emit TokensClaimed(msg.sender, tokenClaim, block.timestamp);

    if (amount == tokenClaim) {
        uint256 tokenId = fsd.tokenizeConviction(0);
        fairSideConviction.transferFrom(address(this), msg.sender, tokenId);
    }
}
```

### Recommendation

Change to:

```solidity=124
function claimVestedTokens() external override onlyBeneficiary {
    uint256 tokenClaim = calculateVestingClaim();
    require(
        tokenClaim > 0,
        "FSDVesting::claimVestedTokens: Zero claimable tokens"
    );

    totalClaimed = totalClaimed.add(tokenClaim);
    lastClaimAt = block.timestamp;

    fsd.safeTransfer(msg.sender, tokenClaim);

    emit TokensClaimed(msg.sender, tokenClaim, block.timestamp);

    if (amount == totalClaimed) {
        uint256 tokenId = fsd.tokenizeConviction(0);
        fairSideConviction.transferFrom(address(this), msg.sender, tokenId);
    }
}
```

