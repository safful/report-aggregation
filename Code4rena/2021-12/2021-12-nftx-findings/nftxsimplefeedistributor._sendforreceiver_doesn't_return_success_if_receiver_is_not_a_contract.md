## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [NFTXSimpleFeeDistributor._sendForReceiver doesn't return success if receiver is not a contract](https://github.com/code-423n4/2021-12-nftx-findings/issues/105) 

# Handle

hyh


# Vulnerability details

# Impact

Double spending of fees being distributed will happen in favor of the first fee receivers in the `feeReceivers` list at the expense of the last ones.
As `_sendForReceiver` doesn't return success for completed transfer when receiver isn't a contract, the corresponding fee amount is sent out twice, to the current and to the next fee receiver in the list. This will lead to double payments for those receivers who happen to be next in the line right after EOAs, and missed payments for the receivers positioned closer to the end of the list as the funds available are going to be already depleted when their turn comes.

## Proof of Concept

`distribute` use `_sendForReceiver` to transfer current vault balance across `feeReceivers`:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L67

`_sendForReceiver` returns a boolean that is used to move current distribution amount to the next receiver when last transfer failed.
When `_receiver.isContract` is `false` nothing is returned, while `safeTransfer` is done:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L168

This way `_sendForReceiver` will indicate that transfer is failed and leftover amount to be added to the next transfer, i.e. the `amountToSend` will be spent twice:
https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L64

## Recommended Mitigation Steps

Now:
```
function _sendForReceiver(FeeReceiver memory _receiver, uint256 _vaultId, address _vault, uint256 amountToSend) internal virtual returns (bool) {
	if (_receiver.isContract) {
	...
	} else {
		IERC20Upgradeable(_vault).safeTransfer(_receiver.receiver, amountToSend);
	}
}
```

To be:
```
function _sendForReceiver(FeeReceiver memory _receiver, uint256 _vaultId, address _vault, uint256 amountToSend) internal virtual returns (bool) {
	if (_receiver.isContract) {
	...
	} else {
		IERC20Upgradeable(_vault).safeTransfer(_receiver.receiver, amountToSend);
		return true;
	}
}
```

