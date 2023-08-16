## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [`FeePoolV0.sol#distributeMochi()` will unexpectedly flush `treasuryShare`, causing the protocol fee cannot be properly accounted for and collected](https://github.com/code-423n4/2021-10-mochi-findings/issues/114) 

# Handle

WatchPug


# Vulnerability details

`distributeMochi()` will call `_buyMochi()` to convert `mochiShare` to Mochi token and call `_shareMochi()` to send Mochi to vMochi Vault and veCRV Holders. It wont touch the `treasuryShare`.

However, in the current implementation, `treasuryShare` will be reset to `0`. This is unexpected and will cause the protocol fee can not be properly accounted for and collected.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/feePool/FeePoolV0.sol#L79-L95

```solidity=79
function _shareMochi() internal {
    IMochi mochi = engine.mochi();
    uint256 mochiBalance = mochi.balanceOf(address(this));
    // send Mochi to vMochi Vault
    mochi.transfer(
        address(engine.vMochi()),
        (mochiBalance * vMochiRatio) / 1e18
    );
    // send Mochi to veCRV Holders
    mochi.transfer(
        crvVoterRewardPool,
        (mochiBalance * (1e18 - vMochiRatio)) / 1e18
    );
    // flush mochiShare
    mochiShare = 0;
    treasuryShare = 0;
}
```

### Impact

Anyone can call `distributeMochi()` and reset `treasuryShare` to `0`, and then call `updateReserve()` to allocate part of the wrongfuly resetted `treasuryShare` to `mochiShare` and call `distributeMochi()`.

Repeat the steps above and the `treasuryShare` will be consumed to near zero, profits the vMochi Vault holders and veCRV Holders. The protocol suffers the loss of funds.

### Recommendation

Change to:

```solidity=64
function _buyMochi() internal {
    IUSDM usdm = engine.usdm();
    address[] memory path = new address[](2);
    path[0] = address(usdm);
    path[1] = address(engine.mochi());
    usdm.approve(address(uniswapRouter), mochiShare);
    uniswapRouter.swapExactTokensForTokens(
        mochiShare,
        1,
        path,
        address(this),
        type(uint256).max
    );
    // flush mochiShare
    mochiShare = 0;
}

function _shareMochi() internal {
    IMochi mochi = engine.mochi();
    uint256 mochiBalance = mochi.balanceOf(address(this));
    // send Mochi to vMochi Vault
    mochi.transfer(
        address(engine.vMochi()),
        (mochiBalance * vMochiRatio) / 1e18
    );
    // send Mochi to veCRV Holders
    mochi.transfer(
        crvVoterRewardPool,
        (mochiBalance * (1e18 - vMochiRatio)) / 1e18
    );
}
```

