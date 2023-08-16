## Tags

- bug
- sponsor confirmed
- 2 (Med Risk)

# [Excessive `require` makes the transaction fail unexpectedly](https://github.com/code-423n4/2021-11-badgerzaps-findings/issues/50) 

# Handle

WatchPug


# Vulnerability details

The check for `RENCRV_VAULT.blockLock` is only needed when `if (_amounts[1] > 0 || _amounts[2] > 0)`. 

However, in the current implementation, the check is done at the very first, making transactions unrelated to `RENCRV_VAULT` fail unexpectedly if there is a prior transaction involved with `RENCRV_VAULT` in the same block.

https://github.com/Badger-Finance/badger-ibbtc-utility-zaps/blob/8d265aacb905d30bd95dcd54505fb26dc1f9b0b6/contracts/IbbtcVaultZap.sol#L149-L199

```solidity=149{154-157,182}
function deposit(uint256[4] calldata _amounts, uint256 _minOut)
    public
    whenNotPaused
{
    // Not block locked by setts
    require(
        RENCRV_VAULT.blockLock(address(this)) < block.number,
        "blockLocked"
    );
    require(
        IBBTC_VAULT.blockLock(address(this)) < block.number,
        "blockLocked"
    );

    uint256[4] memory depositAmounts;

    for (uint256 i = 0; i < 4; i++) {
        if (_amounts[i] > 0) {
            ASSETS[i].safeTransferFrom(
                msg.sender,
                address(this),
                _amounts[i]
            );
            if (i == 0 || i == 3) {
                // ibbtc and sbtc
                depositAmounts[i] += _amounts[i];
            }
        }
    }

    if (_amounts[1] > 0 || _amounts[2] > 0) {
        // Use renbtc and wbtc to mint ibbtc
        // NOTE: Can change to external zap if implemented
        depositAmounts[0] += _renZapToIbbtc([_amounts[1], _amounts[2]]);
    }
    // ...
}
```


