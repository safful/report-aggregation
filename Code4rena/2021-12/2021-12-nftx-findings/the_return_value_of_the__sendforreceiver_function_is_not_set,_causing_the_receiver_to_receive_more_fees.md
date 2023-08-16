## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [The return value of the _sendForReceiver function is not set, causing the receiver to receive more fees](https://github.com/code-423n4/2021-12-nftx-findings/issues/67) 

# Handle

cccz


# Vulnerability details

## Impact

In the NFTXSimpleFeeDistributor.sol contract, the distribute function is used to distribute the fee, and the distribute function judges whether the fee is sent successfully according to the return value of the _sendForReceiver function.

```
  function distribute(uint256 vaultId) external override virtual nonReentrant {
    require(nftxVaultFactory != address(0));
    address _vault = INFTXVaultFactory(nftxVaultFactory).vault(vaultId);

    uint256 tokenBalance = IERC20Upgradeable(_vault).balanceOf(address(this));

    if (distributionPaused || allocTotal == 0) {
      IERC20Upgradeable(_vault).safeTransfer(treasury, tokenBalance);
      return;
    }

    uint256 length = feeReceivers.length;
    uint256 leftover;
    for (uint256 i = 0; i <length; i++) {
      FeeReceiver memory _feeReceiver = feeReceivers[i];
      uint256 amountToSend = leftover + ((tokenBalance * _feeReceiver.allocPoint) / allocTotal);
      uint256 currentTokenBalance = IERC20Upgradeable(_vault).balanceOf(address(this));
      amountToSend = amountToSend> currentTokenBalance? currentTokenBalance: amountToSend;
      bool complete = _sendForReceiver(_feeReceiver, vaultId, _vault, amountToSend);
      if (!complete) {
        leftover = amountToSend;
      } else {
        leftover = 0;
      }
    }
```

In the _sendForReceiver function, when _receiver is not a contract, no value is returned. By default, this will return false. This will make the distribute function think that the fee sending has failed, and will send more fees next time.

```
  function _sendForReceiver(FeeReceiver memory _receiver, uint256 _vaultId, address _vault, uint256 amountToSend) internal virtual returns (bool) {
    if (_receiver.isContract) {
      IERC20Upgradeable(_vault).approve(_receiver.receiver, amountToSend);
      // If the receive is not properly processed, send it to the treasury instead.
       
      bytes memory payload = abi.encodeWithSelector(INFTXLPStaking.receiveRewards.selector, _vaultId, amountToSend);
      (bool success,) = address(_receiver.receiver).call(payload);

      // If the allowance has not been spent, it means we can pass it forward to next.
      return success && IERC20Upgradeable(_vault).allowance(address(this), _receiver.receiver) == 0;
    } else {
      IERC20Upgradeable(_vault).safeTransfer(_receiver.receiver, amountToSend);
    }
  }
```
## Proof of Concept

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L157-L168

https://github.com/code-423n4/2021-12-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXSimpleFeeDistributor.sol#L49-L67

## Tools Used

Manual analysis

## Recommended Mitigation Steps

```
  function _sendForReceiver(FeeReceiver memory _receiver, uint256 _vaultId, address _vault, uint256 amountToSend) internal virtual returns (bool) {
    if (_receiver.isContract) {
      IERC20Upgradeable(_vault).approve(_receiver.receiver, amountToSend);
      // If the receive is not properly processed, send it to the treasury instead.
       
      bytes memory payload = abi.encodeWithSelector(INFTXLPStaking.receiveRewards.selector, _vaultId, amountToSend);
      (bool success, ) = address(_receiver.receiver).call(payload);

      // If the allowance has not been spent, it means we can pass it forward to next.
      return success && IERC20Upgradeable(_vault).allowance(address(this), _receiver.receiver) == 0;
    } else {
      - IERC20Upgradeable(_vault).safeTransfer(_receiver.receiver, amountToSend);
      + return IERC20Upgradeable(_vault).safeTransfer(_receiver.receiver, amountToSend);
    }
  }
```

