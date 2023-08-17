## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Dysfunctional `CToken._acceptAdmin` due to lack of function to assign `pendingAdmin`](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/29) 

# Lines of code

https://github.com/code-423n4/2022-04-dualityfocus/blob/main/contracts/compound_rari_fork/CToken.sol#L1379


# Vulnerability details

## Impact

The implementation of `CToken` in Duality introduced an `_acceptAdmin` function, which presumably should allow changing the `admin`. However, there does not exist a pairing `proposePendingAdmin` function that can propose a new `pendingAdmin`, thus `pendingAdmin` will never be set. This renders the `_acceptAdmin` function useless.

## Proof of Concept

`_acceptAdmin` requires `msg.sender` to equal `pendingAdmin`, however, since `pendingAdmin` can never be set, it will always be `address(0)`, making this function unusable.

```
    function _acceptAdmin() external returns (uint256) {
        // Check caller is pendingAdmin and pendingAdmin ≠ address(0)
        if (msg.sender != pendingAdmin || msg.sender == address(0)) {
            return fail(Error.UNAUTHORIZED, FailureInfo.ACCEPT_ADMIN_PENDING_ADMIN_CHECK);
        }
        // Save current values for inclusion in log
        address oldAdmin = admin;
        address oldPendingAdmin = pendingAdmin;
        // Store admin with value pendingAdmin
        admin = pendingAdmin;
        // Clear the pending value
        pendingAdmin = address(0);
        emit NewAdmin(oldAdmin, admin);
        emit NewPendingAdmin(oldPendingAdmin, pendingAdmin);
        return uint256(Error.NO_ERROR);
    }
```

## Tools Used

vim, ganache-cli

## Recommended Mitigation Steps

Add a `proposePendingAdmin` function where the current admin can propose successors.

```
    function _proposePendingAdmin(address newPendingAdmin) external {
        if (msg.sender != admin) {
            return fail(Error.UNAUTHORIZED, FailureInfo.PROPOSE_PENDING_ADMIN_CHECK);
        }
        address oldPendingAdmin = pendingAdmin;
        pendingAdmin = newPendingAdmin;
        emit NewPendingAdmin(oldPendingAdmin, newPendingAdmin);
        return uint256(Error.NO_ERROR)
    }
```


