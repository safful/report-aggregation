## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- reviewed

# [Customers cannot be `topUp()`ed a second time](https://github.com/code-423n4/2022-04-backd-findings/issues/178) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/CompoundHandler.sol#L71
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/CompoundHandler.sol#L120
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/AaveHandler.sol#L53
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L847


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
Customers cannot be topped up a second time, which will cause them to be liquidated even though they think they're protected

## Proof of Concept
There are multiple places where `safeApprove()` is called a second time without setting the value to zero first. The instances below are all related to topping up.

Compound-specific top-ups will fail the second time around when approving the `ctoken` again:
```solidity
File: backd/contracts/actions/topup/handlers/CompoundHandler.sol   #1

50       function topUp(
51           bytes32 account,
52           address underlying,
53           uint256 amount,
54           bytes memory extra
55       ) external override returns (bool) {
56           bool repayDebt = abi.decode(extra, (bool));
57           CToken ctoken = cTokenRegistry.fetchCToken(underlying);
58           uint256 initialTokens = ctoken.balanceOf(address(this));
59   
60           address addr = account.addr();
61   
62           if (repayDebt) {
63               amount -= _repayAnyDebt(addr, underlying, amount, ctoken);
64               if (amount == 0) return true;
65           }
66   
67           uint256 err;
68           if (underlying == address(0)) {
69               err = ctoken.mint{value: amount}(amount);
70           } else {
71               IERC20(underlying).safeApprove(address(ctoken), amount);
```
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/CompoundHandler.sol#L50-L71

Compound-specific top-ups will also fail when trying to repay debt:
```solidity
File: backd/contracts/actions/topup/handlers/CompoundHandler.sol   #2

62           if (repayDebt) {
63               amount -= _repayAnyDebt(addr, underlying, amount, ctoken);
64               if (amount == 0) return true;
65           }
```
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/CompoundHandler.sol#L62-L65

Aave-specific top-ups will fail for the `lendingPool`:
```solidity
File: backd/contracts/actions/topup/handlers/AaveHandler.sol   #3

36       function topUp(
37           bytes32 account,
38           address underlying,
39           uint256 amount,
40           bytes memory extra
41       ) external override returns (bool) {
42           bool repayDebt = abi.decode(extra, (bool));
43           if (underlying == address(0)) {
44               weth.deposit{value: amount}();
45               underlying = address(weth);
46           }
47   
48           address addr = account.addr();
49   
50           DataTypes.ReserveData memory reserve = lendingPool.getReserveData(underlying);
51           require(reserve.aTokenAddress != address(0), Error.UNDERLYING_NOT_SUPPORTED);
52   
53           IERC20(underlying).safeApprove(address(lendingPool), amount);
```
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/handlers/AaveHandler.sol#L36-L53

The `TopUpAction` itself fails for the `feeHandler`:
```solidity
File: backd/contracts/actions/topup/TopUpAction.sol   #4

840       function _payFees(
841           address payer,
842           address beneficiary,
843           uint256 feeAmount,
844           address depositToken
845       ) internal {
846           address feeHandler = getFeeHandler();
847           IERC20(depositToken).safeApprove(feeHandler, feeAmount);
```
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L840-L847

I've filed the other less-severe instances as a separate medium-severity issue, and flagged the remaining low-severity instances in my QA report

## Tools Used
Code inspection

## Recommended Mitigation Steps
Always do `safeApprove(0)` if the allowance is being changed, or use `safeIncreaseAllowance()`


