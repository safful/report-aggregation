## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [`addFee` will stop accumulating fee once `rewardToken` has reached max supply](https://github.com/code-423n4/2022-07-golom-findings/issues/320) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L98-L138


# Vulnerability details

## Impact
`RewardDistributor` will stop accumulating fees for staker rewards once `rewardToken` supply has reached the maximum supply (1 billion).

## Vulnerability Details
[RewardDistributor.sol#L98-L138](https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/rewards/RewardDistributor.sol#L98-L138)
```
function addFee(address[2] memory addr, uint256 fee) public onlyTrader {
    if (rewardToken.totalSupply() > 1000000000 * 10**18) {
        // if supply is greater then a billion dont mint anything, dont add trades
        return;
    }
    
    ...
    
    feesTrader[addr[0]][epoch] = feesTrader[addr[0]][epoch] + fee;
    feesExchange[addr[1]][epoch] = feesExchange[addr[1]][epoch] + fee;
    epochTotalFee[epoch] = epochTotalFee[epoch] + fee;
}
```
The check at the beginning of `addFee` is supposed to stop `RewardDistributor` from minting additional rewardToken once it has reached 1 billion supply. However, the current implementation has a side effect of causing the function to skip recording accumulated trading fees (the last 3 lines of the function). This will cause stakers to lose their trading fee rewards once the max supply has been reached, and the funds will be permanently locked in the contract.

## Proof of Concept
- Alice staked `GOLOM` to receive fee rewards from `RewardDistributor`.
- `GOLOM` supply reaches 1 billion token.
- Traders keep trading on `GolomTrader`, sending protocol fees to `RewardDistributor`. However, `RewardDistributor.addFee` does not update the fee accounting.
- Alice won't receive any fee reward and protocol fees are stuck in the contract.

## Recommended Mitigation Steps
Modify `addFee` so that the check won't skip accruing trade fees:
```
function addFee(address[2] memory addr, uint256 fee) public onlyTrader {
    if (block.timestamp > startTime + (epoch) * secsInDay) {
        uint256 previousEpochFee = epochTotalFee[epoch];
        epoch = epoch + 1;

        if (rewardToken.totalSupply() > 1000000000 * 10**18) {
            emit NewEpoch(epoch, 0, 0, previousEpochFee);
        } else {
            uint256 tokenToEmit = (dailyEmission * (rewardToken.totalSupply() - rewardToken.balanceOf(address(ve)))) /
                rewardToken.totalSupply();
            uint256 stakerReward = (tokenToEmit * rewardToken.balanceOf(address(ve))) / rewardToken.totalSupply();

            rewardStaker[epoch] = stakerReward;
            rewardTrader[epoch] = ((tokenToEmit - stakerReward) * 67) / 100;
            rewardExchange[epoch] = ((tokenToEmit - stakerReward) * 33) / 100;
            rewardToken.mint(address(this), tokenToEmit);
            epochBeginTime[epoch] = block.number;
            if (previousEpochFee > 0) {
                if (epoch == 1){
                    epochTotalFee[0] =  address(this).balance; // staking and trading rewards start at epoch 1, for epoch 0 all contract ETH balance is converted to staker rewards rewards.
                    weth.deposit{value: address(this).balance}();  
                }else{
                    weth.deposit{value: previousEpochFee}();
                }
            }
            emit NewEpoch(epoch, tokenToEmit, stakerReward, previousEpochFee);
        }
    }
    feesTrader[addr[0]][epoch] = feesTrader[addr[0]][epoch] + fee;
    feesExchange[addr[1]][epoch] = feesExchange[addr[1]][epoch] + fee;
    epochTotalFee[epoch] = epochTotalFee[epoch] + fee;
    return;
}
```