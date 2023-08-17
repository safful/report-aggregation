## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [Lack of `safeApprove(0)` prevents some registrations, and the changing of stakers and LP tokens](https://github.com/code-423n4/2022-04-backd-findings/issues/180) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L50
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L721


# Vulnerability details

OpenZeppelin's `safeApprove()` will revert if the account already is approved and the new `safeApprove()` is done with a non-zero value
```solidity
    function safeApprove(
        IERC20 token,
        address spender,
        uint256 value
    ) internal {
        // safeApprove should only be called when setting an initial allowance,
        // or when resetting it to zero. To increase and decrease it, use
        // 'safeIncreaseAllowance' and 'safeDecreaseAllowance'
        require(
            (value == 0) || (token.allowance(address(this), spender) == 0),
            "SafeERC20: approve from non-zero to non-zero allowance"
        );
        _callOptionalReturn(token, abi.encodeWithSelector(token.approve.selector, spender, value));
    }
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/fcf35e5722847f5eadaaee052968a8a54d03622a/contracts/token/ERC20/utils/SafeERC20.sol#L45-L58

## Impact
Customers can be prevented from `register()`ing the same `token`/`stakerVaultAddress` as another customer; and once changed away from, stakers and lptokens can't be used in the future.

## Proof of Concept
There are multiple places where `safeApprove()` is called a second time without setting the value to zero first.

`register()` calls `lockFunds()` for each user registration, and since users will use the same tokens and staker vaults, the second user's `register()` call will fail:
```solidity
File: backd/contracts/actions/topup/TopUpAction.sol   #1

36       function lockFunds(
37           address stakerVaultAddress,
38           address payer,
39           address token,
40           uint256 lockAmount,
41           uint256 depositAmount
42       ) external {
43           uint256 amountLeft = lockAmount;
44           IStakerVault stakerVault = IStakerVault(stakerVaultAddress);
45   
46           // stake deposit amount
47           if (depositAmount > 0) {
48               depositAmount = depositAmount > amountLeft ? amountLeft : depositAmount;
49               IERC20(token).safeTransferFrom(payer, address(this), depositAmount);
50               IERC20(token).safeApprove(stakerVaultAddress, depositAmount);
```
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L36-L50

The changing of either the staker or an lp token is behind a time-lock, and once the time has passed, the changed variables rely on this function:
```solidity
File: backd/contracts/pool/LiquidityPool.sol   #2

717       function _approveStakerVaultSpendingLpTokens() internal {
718           address staker_ = address(staker);
719           address lpToken_ = address(lpToken);
720           if (staker_ == address(0) || lpToken_ == address(0)) return;
721           IERC20(lpToken_).safeApprove(staker_, type(uint256).max);
722       }
```
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/pool/LiquidityPool.sol#L717-L722

If a bug is found in a new `staker` or `lpToken` and the governor wishes to change back to the old one(s), the governor will have to wait for the time-lock delay only to find out that the old value(s) cause the code to revert.

I've filed the other more-severe instances as a separate high-severity issue, and flagged the remaining low-severity instances in my QA report

## Tools Used
Code inspection

## Recommended Mitigation Steps
Always do `safeApprove(0)` if the allowance is being changed, or use `safeIncreaseAllowance()`


