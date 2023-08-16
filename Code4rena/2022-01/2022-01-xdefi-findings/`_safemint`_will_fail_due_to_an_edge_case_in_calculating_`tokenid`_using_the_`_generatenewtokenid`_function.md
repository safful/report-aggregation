## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [`_safeMint` Will Fail Due To An Edge Case In Calculating `tokenId` Using The `_generateNewTokenId` Function](https://github.com/code-423n4/2022-01-xdefi-findings/issues/17) 

# Handle

leastwood


# Vulnerability details

## Impact

NFTs are used to represent unique positions referenced by the generated `tokenId`. The `tokenId` value contains the position's score in the upper 128 bits and the index wrt. the token supply in the lower 128 bits.

When positions are unlocked after expiring, the relevant position stored in the `positionOf` mapping is deleted, however, the NFT is not. The `merge()` function is used to combine points in unlocked NFTs, burning the underlying NFTs upon merging. As a result, `_generateNewTokenId()` may end up using the same `totalSupply()` value, causing `_safeMint()` to fail if the same `amount_` and `duration_` values are used.

This edge case only occurs if there is an overlap in the `points_` and `totalSupply() + 1` values used to generate `tokenId`. As a result, this may impact a user's overall experience while interacting with the `XDEFI` protocol, as some transactions may fail unexpectedly.

## Proof of Concept

```
function _lock(uint256 amount_, uint256 duration_, address destination_) internal returns (uint256 tokenId_) {
    // Prevent locking 0 amount in order generate many score-less NFTs, even if it is inefficient, and such NFTs would be ignored.
    require(amount_ != uint256(0) && amount_ <= MAX_TOTAL_XDEFI_SUPPLY, "INVALID_AMOUNT");

    // Get bonus multiplier and check that it is not zero (which validates the duration).
    uint8 bonusMultiplier = bonusMultiplierOf[duration_];
    require(bonusMultiplier != uint8(0), "INVALID_DURATION");

    // Mint a locked staked position NFT to the destination.
    _safeMint(destination_, tokenId_ = _generateNewTokenId(_getPoints(amount_, duration_)));

    // Track deposits.
    totalDepositedXDEFI += amount_;

    // Create Position.
    uint96 units = uint96((amount_ * uint256(bonusMultiplier)) / uint256(100));
    totalUnits += units;
    positionOf[tokenId_] =
        Position({
            units: units,
            depositedXDEFI: uint88(amount_),
            expiry: uint32(block.timestamp + duration_),
            created: uint32(block.timestamp),
            bonusMultiplier: bonusMultiplier,
            pointsCorrection: -_toInt256Safe(_pointsPerUnit * units)
        });

    emit LockPositionCreated(tokenId_, destination_, amount_, duration_);
}
```

```
function _generateNewTokenId(uint256 points_) internal view returns (uint256 tokenId_) {
    // Points is capped at 128 bits (max supply of XDEFI for 10 years locked), total supply of NFTs is capped at 128 bits.
    return (points_ << uint256(128)) + uint128(totalSupply() + 1);
}
```

```
function merge(uint256[] memory tokenIds_, address destination_) external returns (uint256 tokenId_) {
    uint256 count = tokenIds_.length;
    require(count > uint256(1), "MIN_2_TO_MERGE");

    uint256 points;

    // For each NFT, check that it belongs to the caller, burn it, and accumulate the points.
    for (uint256 i; i < count; ++i) {
        uint256 tokenId = tokenIds_[i];
        require(ownerOf(tokenId) == msg.sender, "NOT_OWNER");
        require(positionOf[tokenId].expiry == uint32(0), "POSITION_NOT_UNLOCKED");

        _burn(tokenId);

        points += _getPointsFromTokenId(tokenId);
    }

    // Mine a new NFT to the destinations, based on the accumulated points.
    _safeMint(destination_, tokenId_ = _generateNewTokenId(points));
}
```

## Tools Used

Manual code review.
Discussions with Michael.

## Recommended Mitigation Steps

Consider replacing `totalSupply()` in `_generateNewTokenId()` with an internal counter. This should ensure that `_generateNewTokenId()` always returns a unique `tokenId` that is monotomically increasing .

