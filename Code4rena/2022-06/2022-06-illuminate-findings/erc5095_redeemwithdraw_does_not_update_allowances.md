## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ERC5095 redeem/withdraw does not update allowances](https://github.com/code-423n4/2022-06-illuminate-findings/issues/245) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/main/marketplace/ERC5095.sol#L100


# Vulnerability details

ERC5095's `redeem`/`withdraw` allows an ERC20-approved account to redeem user's tokens, but does not update the allowance after burning.

## Impact
User Mal can burn more tokens than Alice allowed him to.
He can set himself to be the receiver of the underlying, therefore Alice will lose funds.

## Proof of Concept
[`withdraw`](https://github.com/code-423n4/2022-06-illuminate/blob/main/marketplace/ERC5095.sol#L100) and [`redeem`](https://github.com/code-423n4/2022-06-illuminate/blob/main/marketplace/ERC5095.sol#L116) functions check that the msg.sender has enough approvals to redeem the tokens:
```
            require(_allowance[holder][msg.sender] >= underlyingAmount, 'not enough approvals');
```
But they do not update the allowances.
They then call `authRedeem`, which also does not update the allowances.
Therefore, an approved user could "re-use his approval" again and again and redeem whole of approver's funds to himself.

## Recommended Mitigation Steps
Update the allowances upon spending.

