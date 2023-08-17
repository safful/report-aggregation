## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- sponsor disputed

# [Unsecure `transferFrom`](https://github.com/code-423n4/2022-06-yieldy-findings/issues/36) 

# Lines of code

https://github.com/code-423n4/2022-06-yieldy/blob/8400e637d9259b7917bde259a5a2fbbeb5946d45/src/contracts/Yieldy.sol#L212


# Vulnerability details

## Impact
The security of the `Yieldy` contract is delegated to the compiler used.

## Proof of Concept
The `allowance` of an account does not have to reflect the real balance of an account, however in the `transferFrom` method, it is the value that is checked in order to verify that the user has enough balance to make the transfer.

```javascript
    function transferFrom(
        address _from,
        address _to,
        uint256 _value
    ) public override returns (bool) {
        require(_allowances[_from][msg.sender] >= _value, "Allowance too low");
```

However, the real balance of the `Yieldy` contract is based on the calculation made by the `creditsForTokenBalance` method, so an underflow could be made in the subtraction of the balance of the `from` account.

```javascript
        uint256 creditAmount = creditsForTokenBalance(_value);
        creditBalances[_from] = creditBalances[_from] - creditAmount;
        creditBalances[_to] = creditBalances[_to] + creditAmount;
        emit Transfer(_from, _to, _value);
```

This means that the security of the contract is delegated to the checks added by the compiler depending on the pragma used, it must be taken into account that these checks may appear and disappear in future versions of the compiler, so they must be checked at the level of smart contracts.

Affected source code:

- [Yieldy.sol#L212](https://github.com/code-423n4/2022-06-yieldy/blob/8400e637d9259b7917bde259a5a2fbbeb5946d45/src/contracts/Yieldy.sol#L212)

## Recommended Mitigation Steps
- Check that the from account has a `creditAmount` balance.

