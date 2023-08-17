## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Deny of service in `CNote.doTransferOut`](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/43) 

# Lines of code

https://github.dev/Plex-Engineer/lending-market-v2/blob/2646a7676b721db8a7754bf5503dcd712eab2f8a/contracts/CNote.sol#L148


# Vulnerability details

## Impact
The `CNote.doTransferOut` method is susceptible to denial of service.

## Proof of Concept
The logic of the `doTransferOut` method in `CNote` is as follows:
```javascript
    function doTransferOut(address payable to, uint amount) virtual override internal {
        require(address(_accountant) != address(0));
        EIP20Interface token = EIP20Interface(underlying);
        if (to != address(_accountant)) {
            uint err = _accountant.supplyMarket(amount);
            if (err != 0) { revert AccountantRedeemError(amount); }
        }   
        token.transfer(to, amount);
        bool success;
        assembly {
            switch returndatasize()
                case 0 { success := not(0) }
                case 32 { 
                    returndatacopy(0, 0, 32)
                    success := mload(0)
                }
                default { revert(0, 0) }
        } 
        require(success, "TOKEN_TRANSFER_OUT_FAILED");
        require(token.balanceOf(address(this)) == 0, "cNote::doTransferOut: TransferOut Failed"); // <-- ERROR
    }
```

The `doTransferOut` method receives an `amount` which is transferred to `to`, after it the balance of the contract token is checked to be equal to zero or the transaction will be reverted.

In the following cases a denial of service will occur:
- In the case that is used an `amount` different than the balance, the transaction will be reverted.
- **In the case that an attacker front-runs the transaction and sends one token more than the established by the `_accountant`.**
- In case of increasing balance tokens like `mDai` that constantly change their balance, the established by the `_accountant` will be different when the transaction is persisted.

## Recommended Mitigation Steps
- Use balance differences instead of the 0 check.

