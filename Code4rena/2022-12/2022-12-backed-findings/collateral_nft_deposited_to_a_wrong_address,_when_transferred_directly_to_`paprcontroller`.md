## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-03

# [Collateral NFT deposited to a wrong address, when transferred directly to `PaprController`](https://github.com/code-423n4/2022-12-backed-findings/issues/183) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L159


# Vulnerability details

## Impact
Users will lose collateral NFTs when they are transferred to `PaprController` by an approved address or an operator.
## Proof of Concept
The `PaprController` allows users to deposit NFTs as collateral to borrow Papr tokens. One of the way of depositing is by transferring an NFT to the contract directly via a call to `safeTransferFrom`: the contract implements the `onERC721Received` hook that will handle accounting of the transferred NFT ([PaprController.sol#L159](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L159)). However, the hook implementation uses a wrong argument to identify token owner: the first argument, which is used by the contract to identify token owner, is the address of the `safeTransferFrom` function caller, which may be an approved address or an operator. The actual owner address is the second argument ([ERC721.sol#L436](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L436)):
```solidity
try IERC721Receiver(to).onERC721Received(_msgSender(), from, tokenId, data) returns (bytes4 retval) {
```

Thus, when an NFT is sent by an approved address or an operator, it'll be deposited to the vault of the approved address or operator:
```solidity
// test/paprController/OnERC721ReceivedTest.sol

function testSafeTransferByOperator_AUDIT() public {
    address operator = address(0x12345);

    vm.prank(borrower);
    nft.setApprovalForAll(operator, true);

    vm.prank(operator);
    nft.safeTransferFrom(borrower, address(controller), collateralId, abi.encode(safeTransferReceivedArgs));

    // NFT was deposited to the operator's vault.
    IPaprController.VaultInfo memory vaultInfo = controller.vaultInfo(operator, collateral.addr);
    assertEq(vaultInfo.count, 1);

    // Borrower has 0 tokens in collateral.
    vaultInfo = controller.vaultInfo(borrower, collateral.addr);
    assertEq(vaultInfo.count, 0);
}

function testSafeTransferByApproved_AUDIT() public {
    address approved = address(0x12345);

    vm.prank(borrower);
    nft.approve(approved, collateralId);

    vm.prank(approved);
    nft.safeTransferFrom(borrower, address(controller), collateralId, abi.encode(safeTransferReceivedArgs));

    // NFT was deposited to the approved address's vault.
    IPaprController.VaultInfo memory vaultInfo = controller.vaultInfo(approved, collateral.addr);
    assertEq(vaultInfo.count, 1);

    // Borrower has 0 tokens in collateral.
    vaultInfo = controller.vaultInfo(borrower, collateral.addr);
    assertEq(vaultInfo.count, 0);
}
```
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider this change:
```diff
--- a/src/PaprController.sol
+++ b/src/PaprController.sol
@@ -156,7 +156,7 @@ contract PaprController is
     /// @param _id the id of the NFT
     /// @param data encoded IPaprController.OnERC721ReceivedArgs
     /// @return selector indicating succesful receiving of the NFT
-    function onERC721Received(address from, address, uint256 _id, bytes calldata data)
+    function onERC721Received(address, address from, uint256 _id, bytes calldata data)
         external
         override
         returns (bytes4)
```