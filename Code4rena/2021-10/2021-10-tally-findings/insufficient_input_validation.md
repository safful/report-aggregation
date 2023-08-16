## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Insufficient input validation](https://github.com/code-423n4/2021-10-tally-findings/issues/25) 

# Handle

WatchPug


# Vulnerability details

`Swap.sol#constructor()` validates `owner_` but does not validate `swapFee_`. While in `setSwapFee()`, it does validates the input to make sure `swapFee_ < SWAP_FEE_DIVISOR`.

https://github.com/code-423n4/2021-10-tally/blob/c585c214edb58486e0564cb53d87e4831959c08b/contracts/swap/Swap.sol#L51-L58

```solidity
constructor(address owner_, address payable feeRecipient_, uint256 swapFee_) {
        require(owner_ != address(0), "Swap::constructor: Owner must not be 0");
        transferOwnership(owner_);
        feeRecipient = feeRecipient_;
        emit NewFeeRecipient(feeRecipient);
        swapFee = swapFee_;
        emit NewSwapFee(swapFee);
    }
```

### Recommendation

Change to:

```solidity
constructor(address owner_, address payable feeRecipient_, uint256 swapFee_) {
    require(owner_ != address(0), "Swap::constructor: Owner must not be 0");
    require(swapFee_ < SWAP_FEE_DIVISOR, "Swap::constructor: Swap fee must not exceed 100%");
    transferOwnership(owner_);
    feeRecipient = feeRecipient_;
    emit NewFeeRecipient(feeRecipient);
    swapFee = swapFee_;
    emit NewSwapFee(swapFee);
}
```

