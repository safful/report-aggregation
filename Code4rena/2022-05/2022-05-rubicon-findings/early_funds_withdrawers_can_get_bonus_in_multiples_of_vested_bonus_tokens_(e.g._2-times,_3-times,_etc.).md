## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Early funds withdrawers can get bonus in multiples of vested bonus tokens (e.g. 2-times, 3-times, etc.)](https://github.com/code-423n4/2022-05-rubicon-findings/issues/450) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L270
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L629
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L98-L101


# Vulnerability details


The function setBonusToken allows the same BonusToken to be added more than once to the array bonusTokens. 
```
  function setBonusToken(address newBonusERC20) external onlyBathHouse {
    bonusTokens.push(newBonusERC20);
  }
```

## Impact
If that happens, early withdrawers can get Bonus in multiples of what they actually have right to. Late withdrawers, might not get any Bonus due to shortage.

## Proof of Concept
BathToken.sol, function setBonusToken
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L270-L272
1. function setBonusToken allows the same BonusToken to be added more than once to the array.

BathToken.sol, function distributeBonusTokenRewards
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L629
2. a. As and when distributeBonusTokenRewards is triggered during a withdraw call, the same bonusToken will be released more than once.

BathBuddy.sol, function release
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L98-L101
2. b. The release function is called.

## Tools Used
Manual review

## Recommended Mitigation Steps
  Add the required validations to avoid duplicate additions of bonus tokens.

```
  function setBonusToken(address newBonusERC20) external onlyBathHouse {
    require(newBonusERC20 != address(0), "invalid_addr");
    if (bonusTokens.length > 0) {
      for (uint256 index = 0; index < bonusTokens.length; index++) {
        require (token != newBonusERC20, "token already exists")
      }
    }
    bonusTokens.push(newBonusERC20);
  }
```


