## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [PaladinRewardReserve.sol may have potential bugs if it uses new tokens as rewards](https://github.com/code-423n4/2022-03-paladin-findings/issues/26) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/PaladinRewardReserve.sol


# Vulnerability details

## Impact
``PaladinRewardReserve.sol`` may have potential bugs if it uses new tokens as rewards.

## Proof of Concept
Currently, ``PaladinRewardReserve.sol`` has following behaviors:

- ``mapping(address => bool) public approvedSpenders`` does not store the info regarding which token it targets
- ``setNewSpender``, ``updateSpenderAllowance``, ``removeSpender`` and ``transferToken`` functions can set ``token`` arbitrarily

Hence, some corner cases may happen as follows:
- Use TokenA at PaladinRewardReserve.sol and do operations.
- Start TokenB as rewards at PaladinRewardReserve.sol. 
- All the information stored in ``approvedSpenders`` was intended for TokenA. So it is possible that following corner cases happen:
  - ``setNewSpender`` function cannot set new token
  - If userA is already added in ``approvedSpenders`` for TokenA, it can call ``updateSpenderAllowance``.


## Tools Used
Statis code analysis

## Recommended Mitigation Steps
Do either of followings depending on the product specification:

(1) If PAL token is only used and other token will never be used at ``PaladinRewardReserve.sol``, stop having ``address token`` argument at ``setNewSpender``, ``updateSpenderAllowance``, ``removeSpender`` and ``transferToken`` functions. Instead, set ``token`` at the constructor or other ways, and limit the ability to flexibly set ``token`` from functions.

(2) If other tokens potentially will be used at ``PaladinRewardReserve.sol``, update data structure of ``approvedSpenders`` mapping and change the logic. 
Firstly, it should also contain the info which ``token`` it targets such as ``mapping(address => address => bool)``. 
Secondly, it should rewrite the ``require`` logic at each function as follows.

```
require(!approvedSpenders[spender][token], "Already Spender on the specified Token");
```

```
require(approvedSpenders[spender][token], "Not approved Spender on the specified Token");
```

