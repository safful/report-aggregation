## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [StakerVault.unstake(), StakerVault.unstakeFor() would revert with a uint underflow error of StakerVault.strategiesTotalStaked, StakerVault._poolTotalStaked.](https://github.com/code-423n4/2022-05-backd-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/StakerVault.sol#L98-L102
https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/StakerVault.sol#L342-L346
https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/StakerVault.sol#L391-L395


# Vulnerability details

## Impact
StakerVault.unstake(), StakerVault.unstakeFor() would revert with a uint underflow error of StakerVault.strategiesTotalStaked, StakerVault._poolTotalStaked.

## Proof of Concept
Currently it saves totalStaked for strategies and non-strategies separately.
uint underflow error could be occured in these cases.

Scenario 1.
1. Address A(non-strategy) stakes some amount x and it will be added to StakerVault_poolTotalStaked.
2. This address A is approved as a strategy by StakerVault.inflationManager.
3. Address A tries to unstake amount x, it will be deducted from StakerVault.strategiesTotalStaked because this address is a strategy already.
Even if it would succeed for this strategy but it will revert for other strategies because StakerVault.strategiesTotalStaked is less than correct staked amount for strategies.

Scenario 2.
There is a transfer between strategy and non-strategy using StakerVault.transfer(), StakerVault.transferFrom() functions.
In this case, StakerVault.strategiesTotalStaked and StakerVault._poolTotalStaked must be changed accordingly but there is no such logic.

## Tools Used
Solidity Visual Developer of VSCode

## Recommended Mitigation Steps
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

