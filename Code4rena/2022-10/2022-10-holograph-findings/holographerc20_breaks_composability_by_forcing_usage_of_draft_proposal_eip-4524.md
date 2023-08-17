## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- selected for report
- responded

# [HolographERC20 breaks composability by forcing usage of draft proposal EIP-4524](https://github.com/code-423n4/2022-10-holograph-findings/issues/440) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC20.sol#L539


# Vulnerability details

## Description

HolographERC20 is the ERC20 enforcer for Holograph. In  the safeTransferFrom operation, it calls \_checkOnERC20Received:

```
if (_isEventRegistered(HolographERC20Event.beforeSafeTransfer)) {
  require(SourceERC20().beforeSafeTransfer(account, recipient, amount, data));
}
_transfer(account, recipient, amount);
require(_checkOnERC20Received(account, recipient, amount, data), "ERC20: non ERC20Receiver");
if (_isEventRegistered(HolographERC20Event.afterSafeTransfer)) {
  require(SourceERC20().afterSafeTransfer(account, recipient, amount, data));
}
```

The checkOnERC20Received function:
```
if (_isContract(recipient)) {
  try ERC165(recipient).supportsInterface(ERC165.supportsInterface.selector) returns (bool erc165support) {
    require(erc165support, "ERC20: no ERC165 support");
    // we have erc165 support
    if (ERC165(recipient).supportsInterface(0x534f5876)) {
      // we have eip-4524 support
      try ERC20Receiver(recipient).onERC20Received(address(this), account, amount, data) returns (bytes4 retv
        return retval == ERC20Receiver.onERC20Received.selector;
      } catch (bytes memory reason) {
        if (reason.length == 0) {
          revert("ERC20: non ERC20Receiver");
        } else {
          assembly {
            revert(add(32, reason), mload(reason))
          }
        }
      }
    } else {
      revert("ERC20: eip-4524 not supported");
    }
  } catch (bytes memory reason) {
    if (reason.length == 0) {
      revert("ERC20: no ERC165 support");
    } else {
      assembly {
        revert(add(32, reason), mload(reason))
      }
    }
  }
} else {
  return true;
}
```

In essence, if the target is a contract, the enforcer requires it to fully implement EIP-4524. The problem is that [this](https://eips.ethereum.org/EIPS/eip-4524) EIP is just a draft proposal, which the project cannot assume to be supported by any receiver contract, and definitely not every receiver contract.

The specs warn us:
```
⚠️ This EIP is not recommended for general use or implementation as it is likely to change.

```

Therefore, it is a very dangerous requirement to add in an ERC20 enforcer, and must be left to the implementation to do if it so desires.

## Impact

ERC20s enforced by HolographERC20 are completely uncomposable. They cannot be used for almost any DeFi application, making it basically useless.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Remove the EIP-4524 requirements altogether.