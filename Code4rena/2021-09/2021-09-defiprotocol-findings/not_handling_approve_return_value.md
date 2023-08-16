## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Not handling approve return value](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/73) 

# Handle

WatchPug


# Vulnerability details

As defined in the ERC20 Specification, the approve function returns a bool that signals the success of the call. However, in `Basket.sol#approveUnderlying()` the value returned from calls to approve is ignored.

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L224-L228

## Recommended Mitigation Steps

To handle calls to approve safely, consider using the safeApprove function in OpenZeppelin’s SafeERC20 contract for all approvals.

