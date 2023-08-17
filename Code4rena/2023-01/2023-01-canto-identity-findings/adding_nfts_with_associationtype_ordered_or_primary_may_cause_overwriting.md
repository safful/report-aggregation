## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-01

# [Adding NFTS with AssociationType ORDERED or PRIMARY may cause overwriting](https://github.com/code-423n4/2023-01-canto-identity-findings/issues/187) 

## Risk rating
Medium Risk

## Links to affected code
https://github.com/OpenCoreCH/cid-c4-squad/blob/4558d25aa8ea92644f3e778457fd6708104e0f24/src/CidNFT.sol#L192-L199

## Impact
Subprotocol NFTs may be trapped in contract CidNFT forever.

## Proof of Concept
When [adding NFT to CidNFT with AssociationType ORDERED or PRIMARY](https://github.com/OpenCoreCH/cid-c4-squad/blob/4558d25aa8ea92644f3e778457fd6708104e0f24/src/CidNFT.sol#L192-L199), the cidData is written directly, without checking and handling the case that a previously added nft may not have been removed:

  ```
  if (_type == AssociationType.ORDERED) {
      if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
      cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;
      emit OrderedDataAdded(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
  } else if (_type == AssociationType.PRIMARY) {
      if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
      cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
      emit PrimaryDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd);
  ...
  ```

For `AssociationType.ORDERED`:
If `(key1, subNft1)` and `(key1, subNft2)` were added consecutively, `subNft1` would be trapped in the contract forever, because `subNft1` stored in `cidData` was overwritten by `subNft2`, and only `subNft2` can be retrieved through `CidNFT.remove()`.

For `AssociationType.PRIMARY`:
If `subNft1` and `subNft2` were added consecutively, `subNft1` would be trapped in the contract forever, because `subNft1` stored in `cidData` was overwritten by `subNft2`, and only `subNft2` can be retrieved through `CidNFT.remove()`.

Test code for PoC:
```
diff --git a/src/test/CidNFT.t.sol b/src/test/CidNFT.t.sol
index 8a6a87a..45d91bd 100644
--- a/src/test/CidNFT.t.sol
+++ b/src/test/CidNFT.t.sol
@@ -67,6 +67,81 @@ contract CidNFTTest is DSTest, ERC721TokenReceiver {
         vm.stopPrank();
     }
 
+    function testTrappedByAddingOrdered() public {
+        address user = user2;
+        vm.startPrank(user);
+
+        // mint two nft for user
+        (uint256 nft1, uint256 nft2) = (101, 102);
+        sub1.mint(user, nft1);
+        sub1.mint(user, nft2);
+        sub1.setApprovalForAll(address(cidNFT), true);
+        // mint CidNFT
+        uint256 cid = cidNFT.numMinted() + 1;
+        cidNFT.mint(new bytes[](0));
+        uint256 key = 111;
+
+        // add nft1 to CidNFT a key
+        cidNFT.add(cid, "sub1", key, nft1, CidNFT.AssociationType.ORDERED);
+        // add nft2 to CidNFT with the same key
+        cidNFT.add(cid, "sub1", key, nft2, CidNFT.AssociationType.ORDERED);
+
+        // confirm: both nft1 and nft2 have been transferred to CidNFT
+        assertEq(sub1.ownerOf(nft1), address(cidNFT));
+        assertEq(sub1.ownerOf(nft2), address(cidNFT));
+
+        // the first remove will success
+        cidNFT.remove(cid, "sub1", key, nft1, CidNFT.AssociationType.ORDERED);
+        // nft2 has been transferred back to the user
+        assertEq(sub1.ownerOf(nft2), user);
+
+        // the second remove will fail for OrderedValueNotSet
+        vm.expectRevert(abi.encodeWithSelector(CidNFT.OrderedValueNotSet.selector, cid, "sub1", key));
+        cidNFT.remove(cid, "sub1", key, nft1, CidNFT.AssociationType.ORDERED);
+        // nft1 is trapped in CidNFT forever
+        assertEq(sub1.ownerOf(nft1), address(cidNFT));
+
+        vm.stopPrank();
+    }
+
+    function testTrappedByAddingPrimary() public {
+        address user = user2;
+        vm.startPrank(user);
+
+        // mint two nft for user
+        (uint256 nft1, uint256 nft2) = (101, 102);
+        sub1.mint(user, nft1);
+        sub1.mint(user, nft2);
+        sub1.setApprovalForAll(address(cidNFT), true);
+        // mint CidNFT
+        uint256 cid = cidNFT.numMinted() + 1;
+        cidNFT.mint(new bytes[](0));
+        // key is useless when adding PRIMARY type
+        uint256 key = 111;
+
+        // add nft1 to CidNFT
+        cidNFT.add(cid, "sub1", key, nft1, CidNFT.AssociationType.PRIMARY);
+        // add nft2 to CidNFT
+        cidNFT.add(cid, "sub1", key, nft2, CidNFT.AssociationType.PRIMARY);
+
+        // confirm: both nft1 and nft2 have been transferred to CidNFT
+        assertEq(sub1.ownerOf(nft1), address(cidNFT));
+        assertEq(sub1.ownerOf(nft2), address(cidNFT));
+
+        // the first remove will success
+        cidNFT.remove(cid, "sub1", key, nft1, CidNFT.AssociationType.PRIMARY);
+        // nft2 has been transferred back to the user
+        assertEq(sub1.ownerOf(nft2), user);
+
+        // the second remove will fail for PrimaryValueNotSet
+        vm.expectRevert(abi.encodeWithSelector(CidNFT.PrimaryValueNotSet.selector, cid, "sub1"));
+        cidNFT.remove(cid, "sub1", key, nft1, CidNFT.AssociationType.PRIMARY);
+        // nft1 is trapped in CidNFT forever
+        assertEq(sub1.ownerOf(nft1), address(cidNFT));
+
+        vm.stopPrank();
+    }
+
     function testAddID0() public {
         // Should revert if trying to add NFT ID 0
         vm.expectRevert(abi.encodeWithSelector(CidNFT.NotAuthorizedForCIDNFT.selector, address(this), 0, address(0)));
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Should revert the tx if an overwriting is found in [CidNFT.add()](https://github.com/OpenCoreCH/cid-c4-squad/blob/4558d25aa8ea92644f3e778457fd6708104e0f24/src/CidNFT.sol#L192-L199):
```
diff --git a/src/CidNFT.sol b/src/CidNFT.sol
index b6c88de..c389971 100644
--- a/src/CidNFT.sol
+++ b/src/CidNFT.sol
@@ -101,6 +101,8 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
     error AssociationTypeNotSupportedForSubprotocol(AssociationType associationType, string subprotocolName);
     error NotAuthorizedForCIDNFT(address caller, uint256 cidNFTID, address cidNFTOwner);
     error NotAuthorizedForSubprotocolNFT(address caller, uint256 subprotocolNFTID);
+    error OrderedKeyIsSetAlready(uint256 cidNFTID, string subprotocolName, uint256 key, uint256 nftIDToAdd);
+    error PrimaryIsSetAlready(uint256 cidNFTID, string subprotocolName, uint256 nftIDToAdd);
     error ActiveArrayAlreadyContainsID(uint256 cidNFTID, string subprotocolName, uint256 nftIDToAdd);
     error OrderedValueNotSet(uint256 cidNFTID, string subprotocolName, uint256 key);
     error PrimaryValueNotSet(uint256 cidNFTID, string subprotocolName);
@@ -191,10 +193,16 @@ contract CidNFT is ERC721, ERC721TokenReceiver {
         }
         if (_type == AssociationType.ORDERED) {
             if (!subprotocolData.ordered) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
+            if (cidData[_cidNFTID][_subprotocolName].ordered[_key] != 0) {
+                revert OrderedKeyIsSetAlready(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
+            }
             cidData[_cidNFTID][_subprotocolName].ordered[_key] = _nftIDToAdd;
             emit OrderedDataAdded(_cidNFTID, _subprotocolName, _key, _nftIDToAdd);
         } else if (_type == AssociationType.PRIMARY) {
             if (!subprotocolData.primary) revert AssociationTypeNotSupportedForSubprotocol(_type, _subprotocolName);
+            if (cidData[_cidNFTID][_subprotocolName].primary != 0) {
+                revert PrimaryIsSetAlready(_cidNFTID, _subprotocolName, _nftIDToAdd);
+            }
             cidData[_cidNFTID][_subprotocolName].primary = _nftIDToAdd;
             emit PrimaryDataAdded(_cidNFTID, _subprotocolName, _nftIDToAdd);
         } else if (_type == AssociationType.ACTIVE) {
```