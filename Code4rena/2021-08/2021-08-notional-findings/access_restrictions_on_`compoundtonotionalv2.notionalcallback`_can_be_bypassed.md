## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Access restrictions on `CompoundToNotionalV2.notionalCallback` can be bypassed](https://github.com/code-423n4/2021-08-notional-findings/issues/69) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `CompoundToNotionalV2.notionalCallback` is supposed to only be called from the verified contract that calls this callback but the access restrictions can be circumvented by simply providing `sender = this` as `sender` is a parameter of the function that can be chosen by the attacker.

```solidity
function notionalCallback(
    address sender,
    address account,
    bytes calldata callbackData
) external returns (uint256) {
// @audit sender can be passed in by the attacker
require(sender == address(this), "Unauthorized callback");
```

## Impact
An attacker can call the function passing in an arbitrary `account` whose tokens are then transferred to the contract.
The `account` first has to approve this contract but this can happen with accounts that legitimately want to call the outer function and have to send a first transaction to approve the contract, but then an attacker frontruns the actual transaction.

It's at least a griefing attack:
I can pass in a malicious `cTokenBorrow` that returns any token of my choice (through the `.underlying()` call) but whose `repayBorrowBehalf` is a no-op.
This will lead to any of the victim's approved tokens becoming stuck in the contract, essentially burning them:

```solidity
// @audit using a malicious contract, this can be any token
address underlyingToken = CTokenInterface(cTokenBorrow).underlying();
bool success = IERC20(underlyingToken).transferFrom(account, address(this), cTokenRepayAmount);
require(success, "Transfer of repayment failed");

// Use the amount transferred to repay the borrow
// @audit using a malicious contract, this can be a no-op
uint code = CErc20Interface(cTokenBorrow).repayBorrowBehalf(account, cTokenRepayAmount);
```

Note that the assumption at the end of the function "// When this exits a free collateral check will be triggered" is not correct anymore but I couldn't find a way to make use of it to lead to an invalid account state.

## Recommended Mitigation Steps
Fix the authorization check.


