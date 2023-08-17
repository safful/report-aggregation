## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed

# [Strategists can take more rewards than they should using the function strategistBootyClaim().](https://github.com/code-423n4/2022-05-rubicon-findings/issues/157) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/tree/main/contracts/rubiconPools/BathPair.sol#L591-L625


# Vulnerability details

## Impact
Strategists can take more rewards than they should using the function strategistBootyClaim().
Even though the owner trusts strategists fully I think it's recommended to remove such flaws.
I think there would be 2 methods to claim more rewards.


## Proof of Concept
Method 1.
A strategist can call the function using same asset/quote parameters.
Then both of fillCountA and fillCountQ will be same positive values.
The first code block for fillCountA(L597-L610) will work same as expected but the second block for fillCountQ(L611-L624) will be executed for the same asset again.
Two mappings(totalFillsPerAsset, strategist2Fills) that save rewards will be updated for asset already after the first block 
but totalFillsPerAsset and balance of this contract for quote would be still positive as there would be remaining rewards for other strategiets.
So the strategist can get paid once more for the same asset.

Method 2.
I think a reentrancy attack is possible also because two mappings are updated after transfer funds.

## Tools Used
Solidity Visual Developer of VSCode

## Recommended Mitigation Steps
For Method 1.
You can add this require() at the beginning of function.(L595)
require(asset != quote, "asset = quote");

For Method 2.
You can update the state of 2 mappings before transfer.
Move L608-L609 to L601
Move L622-L623 to L615

So final code will look like this.(pseudocode)

function strategistBootyClaim(address asset, address quote)
    external
    onlyApprovedStrategist(msg.sender)
{
    require(asset != quote, "asset = quote");

    uint256 fillCountA = strategist2Fills[msg.sender][asset];
    uint256 fillCountQ = strategist2Fills[msg.sender][quote];
    if (fillCountA > 0) {
        uint256 booty = (
            fillCountA.mul(IERC20(asset).balanceOf(address(this)))
        ).div(totalFillsPerAsset[asset]);

        totalFillsPerAsset[asset] -= fillCountA;
        strategist2Fills[msg.sender][asset] -= fillCountA;

        IERC20(asset).transfer(msg.sender, booty);
        emit LogStrategistRewardClaim(
            msg.sender,
            asset,
            booty,
            block.timestamp
        );
    }
    if (fillCountQ > 0) {
        uint256 booty = (
            fillCountQ.mul(IERC20(quote).balanceOf(address(this)))
        ).div(totalFillsPerAsset[quote]);

        totalFillsPerAsset[quote] -= fillCountQ;
        strategist2Fills[msg.sender][quote] -= fillCountQ;

        IERC20(quote).transfer(msg.sender, booty);
        emit LogStrategistRewardClaim(
            msg.sender,
            quote,
            booty,
            block.timestamp
        );
    }
}

