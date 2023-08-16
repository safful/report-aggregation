## Tags

- bug
- disagree with severity
- 1 (Low Risk)
- sponsor acknowledged
- sponsor confirmed

# [Events not emitted](https://github.com/code-423n4/2021-04-vader-findings/issues/250) 

# Handle

s1m0


# Vulnerability details

## Impact
Events not emitted for important state changes.
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Router.sol#L93
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Router.sol#L98
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Router.sol#L196
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Router.sol#L201
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vault.sol#L61
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L163
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L171
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L179
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L184
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L188
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L193
https://github.com/code-423n4/2021-04-vader/blob/main/vader-protocol/contracts/Vader.sol#L198

## Proof of Concept
-

## Tools Used
Manual analysis.

## Recommended Mitigation Steps
Emit events with meaningful names for the changes made.

