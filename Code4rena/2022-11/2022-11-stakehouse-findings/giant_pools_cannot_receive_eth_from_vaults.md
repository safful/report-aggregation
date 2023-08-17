## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-03

# [Giant pools cannot receive ETH from vaults](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/74) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L137
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L126


# Vulnerability details

## Impact

Both giant pools are affected:
1. GiantSavETHVaultPool
2. bringUnusedETHBackIntoGiantPool


The giant pools have a `bringUnusedETHBackIntoGiantPool` function that calls the vaults to send back any unused ETH.
Currently, any call to this function will revert.
Unused ETH will not be sent to the giant pools and will stay in the vaults.

This causes an insolvency issue when many users want to withdraw ETH and there is not enough liquidity inside the giant pools.

## Proof of Concept

`bringUnusedETHBackIntoGiantPool` calls the vaults to receive ETH:
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantSavETHVaultPool.sol#L137
```
    function bringUnusedETHBackIntoGiantPool(
        address[] calldata _savETHVaults,
        LPToken[][] calldata _lpTokens,
        uint256[][] calldata _amounts
    ) external {
        uint256 numOfVaults = _savETHVaults.length;
        require(numOfVaults > 0, "Empty arrays");
        require(numOfVaults == _lpTokens.length, "Inconsistent arrays");
        require(numOfVaults == _amounts.length, "Inconsistent arrays");
        for (uint256 i; i < numOfVaults; ++i) {
            SavETHVault vault = SavETHVault(_savETHVaults[i]);
            for (uint256 j; j < _lpTokens[i].length; ++j) {
                require(
                    vault.isDETHReadyForWithdrawal(address(_lpTokens[i][j])) == false,
                    "ETH is either staked or derivatives minted"
                );
            }
            vault.burnLPTokens(_lpTokens[i], _amounts[i]);
        }
    }
```

the vaults go through a process of burning the `_lpTokens` and sending the caller giant pool ETH.

`burnLPToken`
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SavETHVault.sol#L126
```
    function burnLPToken(LPToken _lpToken, uint256 _amount) public nonReentrant returns (uint256) {
        /// .....
        (bool result,) = msg.sender.call{value: _amount}("");
        // .....
    }
```

Giant pools do not have a `fallback` or `receive` function. ETH cannot be sent to them

additionally, there is no accounting of `idleETH`, which should be increased with the received ETH in order to facilitate withdraws

## Tools Used

VS Code

## Recommended Mitigation Steps

1. Add a `fallback` or `receive` function to the pools.
2. `idleETH` should be increased with the received ETH 