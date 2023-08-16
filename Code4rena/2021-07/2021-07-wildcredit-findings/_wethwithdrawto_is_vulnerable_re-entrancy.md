## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [_wethWithdrawTo is vulnerable re-entrancy](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/71) 

# Handle

pauliax


# Vulnerability details

## Impact
function withdrawBorrowETH invokes _wethWithdrawTo and later _checkMinReserve, however, the check of reserve is not necessary here, as function _wethWithdrawTo also does that after transferring the ether. However, this reserve check might be bypassed as TransferHelper._wethWithdrawTo uses a low level call that is vulnerable to re-entrancy attacks. As this MIN_RESERVE sounds like an important value, you should consider preventing re-entrancy attacks here.
  // Prevents division by zero and other undesirable behavior
  uint public constant MIN_RESERVE = 1000;

## Recommended Mitigation Steps
Consider using re-entrancy guard on all main action functions (e.g. deposit, withdraw, borrow, repay, etc): https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol

