## Tags

- bug
- disagree with severity
- G (Gas Optimization)
- sponsor confirmed

# [Missing Sanity Checks Will Cause To Revert On the Function](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/71) 

# Handle

defsec


# Vulnerability details

## Impact

In the JoeStaking contract, the amount check should be placed on the contract. IF the amount is more than transfer operations should be completed.

## Proof of Concept

1. Navigate to the following contract.

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeStaking.sol#L129

2. _amount is not checked if Its more than zero.

```
    function withdraw(uint256 _amount) external {
        UserInfo storage user = userInfo[msg.sender];
        require(
            user.amount >= _amount,
            "RocketJoeStaking: withdraw amount exceeds balance"
        );

        updatePool();

        uint256 pending = (user.amount * accRJoePerShare) /
            PRECISION -
            user.rewardDebt;

        user.amount = user.amount - _amount;
        user.rewardDebt = (user.amount * accRJoePerShare) / PRECISION;

        _safeRJoeTransfer(msg.sender, pending);
        joe.safeTransfer(address(msg.sender), _amount);
        emit Withdraw(msg.sender, _amount);
    }

```

## Tools Used

Code Review

## Recommended Mitigation Steps

Consider to add the following check.


```
    function withdraw(uint256 _amount) external {
        UserInfo storage user = userInfo[msg.sender];
        require(
            user.amount >= _amount,
            "RocketJoeStaking: withdraw amount exceeds balance"
        );

        updatePool();

        uint256 pending = (user.amount * accRJoePerShare) /
            PRECISION -
            user.rewardDebt;

        user.amount = user.amount - _amount;
        user.rewardDebt = (user.amount * accRJoePerShare) / PRECISION;

 if(pending != 0) {
        _safeRJoeTransfer(msg.sender, pending);
}

if(_amount != 0){
        joe.safeTransfer(address(msg.sender), _amount);
}

        emit Withdraw(msg.sender, _amount);

    }


```


