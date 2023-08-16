## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [redundant call to _checkMinReserve in withdrawBorrowETH ](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/26) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function withdrawBorrowETH of the contract LendingPair calls _wethWithdrawTo and then calls _checkMinReserve.
However _wethWithdrawTo also calls _checkMinReserve  (except when _amount but then not much happens anyway.

So the call to _checkMinReserve in withdrawBorrowETH is redundant and uses some extra gas.

## Proof of Concept
//https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L106

function withdrawBorrowETH(uint _amount) external {
   ..
    _wethWithdrawTo(msg.sender, _amount);
    _checkMinReserve(address(WETH)); // is also called in _wethWithdrawTo
  }

 function _wethWithdrawTo(address _to, uint _amount) internal override {
    if (_amount > 0) {
      TransferHelper._wethWithdrawTo(_to, _amount);
      _checkMinReserve(address(WETH));
    }
  }

## Tools Used

## Recommended Mitigation Steps
Consider removing the _checkMinReserve in withdrawBorrowETH
Or consider moving the _checkMinReserve to all functions where _wethWithdrawTo is called

