## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Underflown variable in ``borrowGivenDebtETHCollateral`` function](https://github.com/code-423n4/2022-03-timeswap-findings/issues/32) 

# Lines of code

https://github.com/code-423n4/2022-03-timeswap/blob/main/Timeswap/Convenience/contracts/libraries/Borrow.sol#L121-L127


# Vulnerability details

## Impact
``borrowGivenDebtETHCollateral`` function does never properly call ``ETH.transfer`` due to underflow. If ``borrowGivenDebtETHCollateral`` function is not deprecated, it would cause unexpected behaviors for users.


## Proof of Concept
Here are codes which contain a potential issue.

https://github.com/code-423n4/2022-03-timeswap/blob/main/Timeswap/Convenience/contracts/libraries/Borrow.sol#L121-L127
```
if (maxCollateral > dueOut.collateral) {
    uint256 excess;
    unchecked {
        excess -= dueOut.collateral;
    }
    ETH.transfer(payable(msg.sender), excess);
}
```

``excess`` variable is ``uint256``, and ``dueOut.collateral`` variable is ``uint112`` as shown below. Hence, both variables will never be less than 0.

https://github.com/code-423n4/2022-03-timeswap/blob/main/Timeswap/Core/contracts/interfaces/IPair.sol#L22-L26
```
struct Due {
    uint112 debt;
    uint112 collateral;
    uint32 startBlock;
}
```

``uint256 excess`` is initialized to 0. However, subtracting ``dueOut.collateral`` variable which is more than or equal to 0 from ``excess`` variable which is 0 will be less than 0. Hence, ``excess -= dueOut.collateral`` will be less than 0, and ``excess`` will be underflown.


## Tools Used
static code analysis


## Recommended Mitigation Steps
The code should properly initialize ``excess`` variable.

``borrowGivenPercentETHCollateral`` function uses ``uint256 excess = maxCollateral`` at similar functionality.
https://github.com/code-423n4/2022-03-timeswap/blob/main/Timeswap/Convenience/contracts/libraries/Borrow.sol#L347

Hence, just initializing ``excess`` variable with ``maxCollateral`` can be a potential workaround to prevent the underflown.
```
if (maxCollateral > dueOut.collateral) {
    uint256 excess = maxCollateral;
    unchecked {
        excess -= dueOut.collateral;
    }
    ETH.transfer(payable(msg.sender), excess);
}
```

