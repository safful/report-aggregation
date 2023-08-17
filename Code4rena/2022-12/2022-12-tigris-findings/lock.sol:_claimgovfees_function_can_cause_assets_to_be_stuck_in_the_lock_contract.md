## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- M-01

# [Lock.sol: claimGovFees function can cause assets to be stuck in the Lock contract](https://github.com/code-423n4/2022-12-tigris-findings/issues/73) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L110
https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/BondNFT.sol#L215


# Vulnerability details

## Impact
When calling `Lock.claimGovFees` ([https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L110](https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L110)), assets that are set to be not allowed or assets that don't have any shares yet in the `BondNFT` contract will cause a silent failure in `BondNFT.distribute` ([https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/BondNFT.sol#L215](https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/BondNFT.sol#L215)).  
The funds from the `GovNFT` contract will get transferred into the `Lock` contract and then will be stuck there. They cannot be recovered.  

## Proof of Concept
1. An asset is added to the `BondNFT` contract by calling `BondNFT.addAsset` ([https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/BondNFT.sol#L349](https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/BondNFT.sol#L349))
2. There are no bonds yet for this asset so the amount of shares for the asset is zero
3. `Lock.claimGovFees` ([https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L110](https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L110)) is called
4. Funds are transferred from the `GovNFT` contract to the `Lock` contract ([https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L115](https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Lock.sol#L115))
5. The call to `BondNFT.distribute` now fails quietly without reverting the transaction:  
   ```solidity
    if (totalShares[_tigAsset] == 0 || !allowedAsset[_tigAsset]) return;
   ```
6. The funds are now stuck in the `Lock` contract. They cannot be recovered.

## Tools Used
VSCode

## Recommended Mitigation Steps
A naive solution would be to use `revert` instead of `return` in `BondNFT.distribute` such that funds are either transferred from `GovNFT` to `Lock` and then to `BondNFT` or not at all.  

```solidity
     ) external {
-        if (totalShares[_tigAsset] == 0 || !allowedAsset[_tigAsset]) return;
+        if (totalShares[_tigAsset] == 0 || !allowedAsset[_tigAsset]) revert;
         IERC20(_tigAsset).transferFrom(_msgSender(), address(this), _amount);
         unchecked {
             uint aEpoch = block.timestamp / DAY;
```

This however is an incomplete fix because if there is a single "bad" asset, rewards for the other assets cannot be distributed either.  

Moreover functions like `Lock.lock` and `Lock.release` rely on `Lock.claimGovFees` to not revert.  

So you might allow the owner to rescue stuck tokens from the `Lock` contract. Of course only allow rescuing the balance of the `Lock` contract minus the `totalLocked` of the asset in the `Lock` contract such that the locked amount cannot be rescued.  