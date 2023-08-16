## Tags

- bug
- G (Gas Optimization)
- disagree with severity
- sponsor confirmed

# [External call does not have amount check in TransferHelper](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/56) 

# Handle

defsec


# Vulnerability details

## Impact

There is occurrence in the code of the TransferHelper contract where amount is checked after the external call.

## Proof of Concept

- Navigate to [TransferHelper.sol](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/TransferHelper.sol)
-  Amount is checked after an external call. [TransferHelper.sol amount check](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/TransferHelper.sol#L22) 
-  To favor readability and avoid confusions, consider applying check at the beginning of function.

## Tools Used

None

## Recommended Mitigation Steps

To favor readability and avoid confusions, consider applying check at the beginning of function.

```sh
  function _safeTransferFrom(address _token, address _sender, uint _amount) internal virtual {
    require(_amount > 0, "TransferHelper: amount must be > 0");
    ...
  }
```

