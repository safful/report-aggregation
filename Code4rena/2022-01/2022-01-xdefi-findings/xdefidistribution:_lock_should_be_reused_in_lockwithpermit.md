## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [XDEFIDistribution: lock should be reused in lockWithPermit](https://github.com/code-423n4/2022-01-xdefi-findings/issues/47) 

# Handle

PierrickGT


# Vulnerability details

## Impact
In [lockWithPermit](https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L99), we use the same code to transfer XDEFI and lock the position than in [lock](https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L92-96). We can create an internal function to reuse this code and avoid duplication.

## Proof of Concept
Create an internal function called `_lockPosition` that will transfer XDEFI and lock the position. This function will be called in `lock` and `lockWithPermit`.

## Recommended Mitigation Steps
The following change is recommended.

```
function _lockPosition(uint256 amount_, uint256 duration_, address destination_) internal returns (uint256 tokenId_) {
    // Lock the XDEFI in the contract.
    SafeERC20.safeTransferFrom(IERC20(XDEFI), msg.sender, address(this), amount_);

    // Handle the lock position creation and get the tokenId of the locked position.
    return _lock(amount_, duration_, destination_);
}

function lock(uint256 amount_, uint256 duration_, address destination_) external noReenter returns (uint256 tokenId_) {
    return _lockPosition(amount_, duration_, destination_);
}

function lockWithPermit(uint256 amount_, uint256 duration_, address destination_, uint256 deadline_, uint8 v_, bytes32 r_, bytes32 s_) external noReenter returns (uint256 tokenId_) {
    // Approve this contract for the amount, using the provided signature.
    IEIP2612(XDEFI).permit(msg.sender, address(this), amount_, deadline_, v_, r_, s_);

    return _lockPosition(amount_, duration_, destination_);
}
```

