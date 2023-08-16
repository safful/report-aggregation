## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Wand

# [improve safety of role constants ](https://github.com/code-423n4/2021-08-yield-findings/issues/9) 

# Handle

gpersoon


# Vulnerability details

## Impact
The contract Wand defines a few role constants with bytes4(keccak256("...function..."))
However if the function template would change slightly, for example when uint128 is replaced by uint256, then this construction isn't valid anymore.

It is safer the use the function selector, as is done in EmergencyBrake.sol

## Proof of Concept
https://github.com/code-423n4/2021-08-yield/blob/main/contracts/Wand.sol#L27

    bytes4 public constant JOIN = bytes4(keccak256("join(address,uint128)"));
    bytes4 public constant EXIT = bytes4(keccak256("exit(address,uint128)"));
    bytes4 public constant MINT = bytes4(keccak256("mint(address,uint256)"));
    bytes4 public constant BURN = bytes4(keccak256("burn(address,uint256)"));

https://github.com/code-423n4/2021-08-yield/blob/main/contracts/utils/EmergencyBrake.sol#L35
  _grantRole(IEmergencyBrake.plan.selector, planner);

## Tools Used

## Recommended Mitigation Steps
Use function selectors in Wand.sol


