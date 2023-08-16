## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Improper Upper Bound Definition on the Fee](https://github.com/code-423n4/2022-03-biconomy-findings/issues/8) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/token/TokenManager.sol#L51
https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/token/TokenManager.sol#L52


# Vulnerability details

## Impact

The **equilibriumFee** and **maxFee** does not have any upper or lower bounds. Values that are too large will lead to reversions in several critical functions or the LP user will lost all funds when paying the fee.

## Proof of Concept

1. Navigate to the following contract.

https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/token/TokenManager.sol#L52

2. Owner can identify fee amount. That directly affect to LP management. (https://github.com/code-423n4/2022-03-biconomy/blob/db8a1fdddd02e8cc209a4c73ffbb3de210e4a81a/contracts/hyphen/LiquidityPool.sol#L352)

3. Here you can see there is no upper bound has been defined. 

```
    function changeFee(
        address tokenAddress,
        uint256 _equilibriumFee,
        uint256 _maxFee
    ) external override onlyOwner whenNotPaused {
        require(_equilibriumFee != 0, "Equilibrium Fee cannot be 0");
        require(_maxFee != 0, "Max Fee cannot be 0");
        tokensInfo[tokenAddress].equilibriumFee = _equilibriumFee;
        tokensInfo[tokenAddress].maxFee = _maxFee;
        emit FeeChanged(tokenAddress, tokensInfo[tokenAddress].equilibriumFee, tokensInfo[tokenAddress].maxFee);
    }

```

## Tools Used

Code Review

## Recommended Mitigation Steps

Consider defining upper and lower bounds on the **equilibriumFee** and **maxFee**.

