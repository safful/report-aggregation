## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method
- selected for report

# [Wrong balanceOf user after minting legendary gobbler](https://github.com/code-423n4/2022-09-artgobblers-findings/issues/333) 

# Lines of code

https://github.com/code-423n4/2022-09-artgobblers/blob/d2087c5a8a6a4f1b9784520e7fe75afa3a9cbdbe/src/ArtGobblers.sol#L458


# Vulnerability details

## Impact

In `ArtGobblers.mintLegendaryGobbler()` function, line 458 calculates the number of gobblers user owned after minting
```solidity
// We subtract the amount of gobblers burned, and then add 1 to factor in the new legendary.
getUserData[msg.sender].gobblersOwned = uint32(getUserData[msg.sender].gobblersOwned - cost + 1);
```

It added 1 to factor in the new legendary. But actually, this new legendary is accounted in `_mint()` function already
```solidity
function _mint(address to, uint256 id) internal {
    // Does not check if the token was already minted or the recipient is address(0)
    // because ArtGobblers.sol manages its ids in such a way that it ensures it won't
    // double mint and will only mint to safe addresses or msg.sender who cannot be zero.

    unchecked {
        ++getUserData[to].gobblersOwned;
    }

    getGobblerData[id].owner = to;

    emit Transfer(address(0), to, id);
}
```

So the result is `gobblersOwned` is updated incorrectly. And `balanceOf()` will return wrong value.


## Proof of Concept

Script modified from `testMintLegendaryGobbler()`
```solidity
function testMintLegendaryGobbler() public {
    uint256 startTime = block.timestamp + 30 days;
    vm.warp(startTime);
    // Mint full interval to kick off first auction.
    mintGobblerToAddress(users[0], gobblers.LEGENDARY_AUCTION_INTERVAL());
    uint256 cost = gobblers.legendaryGobblerPrice();
    assertEq(cost, 69);
    setRandomnessAndReveal(cost, "seed");
    uint256 emissionMultipleSum;
    for (uint256 curId = 1; curId <= cost; curId++) {
        ids.push(curId);
        assertEq(gobblers.ownerOf(curId), users[0]);
        emissionMultipleSum += gobblers.getGobblerEmissionMultiple(curId);
    }

    assertEq(gobblers.getUserEmissionMultiple(users[0]), emissionMultipleSum);

    uint256 beforeSupply = gobblers.balanceOf(users[0]);
    vm.prank(users[0]);
    uint256 mintedLegendaryId = gobblers.mintLegendaryGobbler(ids);

    // Check balance
    assertEq(gobblers.balanceOf(users[0]), beforeSupply - cost + 1);
}
```

## Tools Used
Foundry

## Recommended Mitigation Steps
Consider remove adding 1 when calculating `gobblersOwned`
```solidity
getUserData[msg.sender].gobblersOwned = uint32(getUserData[msg.sender].gobblersOwned - cost);
```


