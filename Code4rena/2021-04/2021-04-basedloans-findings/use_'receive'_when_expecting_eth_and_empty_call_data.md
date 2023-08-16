## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Use 'receive' when expecting eth and empty call data](https://github.com/code-423n4/2021-04-basedloans-findings/issues/25) 

# Handle

paulius.eth


# Vulnerability details

## Impact
contract CEther fallback function was refactored to be compatible with the Solidity 0.6 version:

  /**
   * @notice Send Ether to CEther to mint
   */
  fallback () external payable {
      (uint err,) = mintInternal(msg.value);
      requireNoError(err, "mint failed");
  }

From Solidity 0.6 documentation:

"The unnamed function commonly referred to as “fallback function” was split up into a new fallback function that is defined using the fallback keyword and a receive ether function defined using the receive keyword. If present, the receive ether function is called whenever the call data is empty (whether or not ether is received). This function is implicitly payable. The new fallback function is called when no other function matches (if the receive ether function does not exist then this includes calls with empty call data). You can make this function payable or not. If it is not payable then transactions not matching any other function which send value will revert. You should only need to implement the new fallback function if you are following an upgrade or proxy pattern."

I think in this case "receive" is more suitable as the function is expecting to receive ether and empty call data.

## Recommended Mitigation Steps
Replace "fallback" with "receive".

