## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Checking non-zero value can avoid an external call to save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/181) 

# Handle

Jujic


# Vulnerability details

## Impact
Checking if  `_amount != 0 ` before making the transfer call  can save gas by avoiding the external call in such situations.

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L201-L206

```
function borrowValue(uint256 _amount, address _to) external onlyMarket override {
        debts[msg.sender] += _amount;
        totalDebt += _amount;

        IERC20(token).safeTransfer(_to, _amount);
    }

```

## Tools Used
Remix

## Recommended Mitigation Steps
Add additional check for non zero ` _amount`.

