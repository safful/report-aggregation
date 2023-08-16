## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`initialize` functions can be frontrun](https://github.com/code-423n4/2021-08-notional-findings/issues/59) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `initialize` function that initializes important contract state can be called by anyone.
Occurences:
- `NoteERC20.initialize`
- `Router.initialize`

## Impact
The attacker can initialize the contract before the legitimate deployer, hoping that the victim continues to use the same contract.
In the best case for the victim, they notice it and have to redeploy their contract costing gas.

## Recommended Mitigation Steps
Use the constructor to initialize non-proxied contracts.
For initializing proxy contracts deploy contracts using a factory contract that immediately calls `initialize` after deployment or make sure to call it immediately after deployment and verify the transaction succeeded.


