## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [_transfer what happens if sender==recipient](https://github.com/code-423n4/2021-08-notional-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _transfer of nTokenAction.sol uses temporary variables and updates the sender and recipient separately.
This is a dangerous constructions because the update of the recipient could overwrite the update of the sender.
This has led to several hacks at other comparable contracts

## Proof of Concept
https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/actions/nTokenAction.sol
```JS
 function _transfer(uint256 currencyId,address sender,address recipient, uint256 amount) internal returns (bool) {
       ...  
        senderBalance.netNTokenTransfer = amountInt.neg();
        recipientBalance.netNTokenTransfer = amountInt;

        senderBalance.finalize(sender, senderContext, false);
        recipientBalance.finalize(recipient, recipientContext, false);
        senderContext.setAccountContext(sender);
        recipientContext.setAccountContext(recipient);
...
```

## Tools Used

## Recommended Mitigation Steps
Double check what happens when sender==recipient

Add checks to make sure (sender!=recipient) because that usually isn't useful anyway.


