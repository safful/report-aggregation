## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Strategy in StakerVault.sol can steal more rewards even though it's designed strategies shouldn't get rewards.](https://github.com/code-423n4/2022-05-backd-findings/issues/85) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/StakerVault.sol#L95
https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/tokenomics/LpGauge.sol#L52-L63


# Vulnerability details

## Impact
Strategy in StakerVault.sol can steal more rewards even though it's designed strategies shouldn't get rewards.
Also there will be a problem with a rewarding system in LpGauge.sol so that some normal users wouldn't get rewards properly.


## Proof of Concept
1. Strategy A staked amount x and x will be added to StakerVault.strategiesTotalStaked.
contracts\StakerVault.sol#L312
2. Strategy A transferred the amount x to non-strategy B and StakerVault.strategiesTotalStaked, StakerVault._poolTotalStaked won't be updated.
contracts\StakerVault.sol#L111
3. After some time for the larger LpGauge.poolStakedIntegral, B claims rewards using the LpGauge.claimRewards() function.
contracts\tokenomics\LpGauge.sol#L52

Inside LpGauge.userCheckPoint(), it's designed not to calculate LpGauge.perUserShare for strategy, but it will pass this condition because B is not a strategy.
contracts\tokenomics\LpGauge.sol#L90

Furthermore, when calculate rewards, LpGauge.poolStakedIntegral will be calculated larger than a normal user stakes same amount.
It's because StakerVault._poolTotalStaked wasn't updated when A transfers x amount to B so LpGauge.poolTotalStaked is less than correct value.
contracts\tokenomics\LpGauge.sol#L113-L117

Finally B can get more rewards than he should and the reward system will pay more rewards than it's designed.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
I think there will be two methods to fix.
Method 1 is to forbid a transfer between strategy and non-strategy so that strategy can't move funds to non-strategy.
Method 2 is to update StakerVault.strategiesTotalStaked and StakerVault._poolTotalStaked correctly so that strategy won't claim more rewards than he should even though he claims rewards using non-strategy.

Method 1.
You need to modify two functions. StakerVault.transfer(), StakerVault.transferFrom().

1. You need to add this require() at L112 for transfer().
require(strategies[msg.sender] == strategies[account], Error.FAILED_TRANSFER);

2. You need to add this require() at L144 for transferFrom().
require(strategies[src] == strategies[dst], Error.FAILED_TRANSFER);

Method 2.
I've explained about this method in my medium risk report "StakerVault.unstake(), StakerVault.unstakeFor() would revert with a uint underflow error of StakerVault.strategiesTotalStaked, StakerVault._poolTotalStaked"
I will copy the same code for your convenience.

You need to modify 3 functions. StakerVault.addStrategy(), StakerVault.transfer(), StakerVault.transferFrom().

1. You need to move staked amount from StakerVault._poolTotalStaked to StakerVault.strategiesTotalStaked every time when StakerVault.inflationManager approves a new strategy.
You can modify addStrategy() at L98-L102 like this.

function addStrategy(address strategy) external override returns (bool) {
    require(msg.sender == address(inflationManager), Error.UNAUTHORIZED_ACCESS);
    require(!strategies[strategy], Error.ADDRESS_ALREADY_SET);

    strategies[strategy] = true;
    _poolTotalStaked -= balances[strategy];
    strategiesTotalStaked += balances[strategy];

    return true;
}

2. You need to add below code at L126 of transfer() function.

if(strategies[msg.sender] != strategies[account]) {
    if(strategies[msg.sender]) { // from strategy to non-strategy
        _poolTotalStaked += amount;
        strategiesTotalStaked -= amount;
    }
    else { // from non-strategy to strategy
        _poolTotalStaked -= amount;
        strategiesTotalStaked += amount;
    }
}

3. You need to add below code at L170 of transferFrom() function.

if(strategies[src] != strategies[dst]) {
    if(strategies[src]) { // from strategy to non-strategy
        _poolTotalStaked += amount;
        strategiesTotalStaked -= amount;
    }
    else { // from non-strategy to strategy
        _poolTotalStaked -= amount;
        strategiesTotalStaked += amount;
    }
}

