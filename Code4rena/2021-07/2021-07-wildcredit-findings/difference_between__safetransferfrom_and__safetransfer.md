## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [difference between _safeTransferFrom and _safeTransfer](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/32) 

# Handle

gpersoon


# Vulnerability details

## Impact
 
The functions _safeTransferFrom and _safeTransfer are similar but there is one difference:
_safeTransferFrom reverts when _amount == 0
_safeTransfer  doesn't do any action when _amount == 0

I don't see any reason for the different behavior.

## Proof of Concept
https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/TransferHelper.sol
function _safeTransferFrom(address _token, address _sender, uint _amount) internal virtual {
    bool success = IERC20(_token).transferFrom(_sender, address(this), _amount);
    require(success, "TransferHelper: transfer failed");
    require(_amount > 0, "TransferHelper: amount must be > 0");
  }

//https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L468
  function _safeTransfer(IERC20 _token, address _recipient, uint _amount) internal {
    if (_amount > 0) {
      bool success = _token.transfer(_recipient, _amount);
      require(success, "LendingPair: transfer failed");
      _checkMinReserve(address(_token));
    }
  }

## Tools Used

## Recommended Mitigation Steps
Double check the difference and perhaps apply the same logic for amount==0 to both functions.

