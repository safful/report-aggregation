## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [User can alter amount returned by redeem function due to control transfer](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/235) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/6f8c325f604e2576e2fe257b6b57892ca181509a/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L212-L222


# Vulnerability details

## Impact
Control is transferred to the receiver when receiving the ERC777. They are able to transfer the ERC777 to another account, at which time the before and after balance calculation will be incorrect.

```
        uint256 balanceBefore = IERC20(asset()).balanceOf(receiver);


        if (msg.sender != owner) {
            _spendAllowance(owner, msg.sender, shares);
        }
        _redeemInternal(shares, receiver, owner);
/////////////
Control is transferred to user. They can alter their balance here.
///////////

        uint256 balanceAfter = IERC20(asset()).balanceOf(receiver);
        uint256 assets = balanceAfter - balanceBefore;

//////////
Assets can be as low as 0 if they have transferred the same amount out as received.
//////////

        emit Withdraw(msg.sender, receiver, owner, assets, shares);
        return assets;
```

## Tools Used
Manual review

