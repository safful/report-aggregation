## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Cache storage variables in the stack can save gas](https://github.com/code-423n4/2021-12-nftx-findings/issues/191) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache them in the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

- `vaultFactory` in `NFTXVaultUpgradeable#_chargeAndDistributeFees()`

    https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXVaultUpgradeable.sol#L470-L484

    ```solidity=470
    function _chargeAndDistributeFees(address user, uint256 amount) internal virtual {
        // Do not charge fees if the zap contract is calling
        // Added in v1.0.3. Changed to mapping in v1.0.5.
        if (vaultFactory.excludedFromFees(msg.sender)) {
            return;
        }
        
        // Mint fees directly to the distributor and distribute.
        if (amount > 0) {
            address feeDistributor = vaultFactory.feeDistributor();
            // Changed to a _transfer() in v1.0.3.
            _transfer(user, feeDistributor, amount);
            INFTXFeeDistributor(feeDistributor).distribute(vaultId);
        }
    }
    ```

