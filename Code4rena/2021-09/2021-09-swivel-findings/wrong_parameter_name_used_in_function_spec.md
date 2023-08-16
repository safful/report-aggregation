## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Wrong parameter name used in function spec](https://github.com/code-423n4/2021-09-swivel-findings/issues/2) 

# Handle

loop


# Vulnerability details

Line 124 of Swivel.sol describes the parameter `uint256 a`, but has wrong parameter name:
`/// @param o Amount of volume (principal) being filled by the taker's exit`

## Impact
No direct impact apart from code readability

## Proof of Concept
Code snippet for function declaration + spec:
```
/// @notice Allows a user to initiate a zcToken by filling an offline vault initiate order
  /// @dev This method should pass (underlying, maturity, sender, maker, a) to MarketPlace.custodialInitiate
  /// @param o Order being filled
  /// @param o Amount of volume (principal) being filled by the taker's exit
  /// @param c Components of a valid ECDSA signature
  function initiateZcTokenFillingVaultInitiate(Hash.Order calldata o, uint256 a, Sig.Components calldata c) internal
```

## Recommended Mitigation Steps
Change the second `o` to `a`

