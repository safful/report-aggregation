## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Implement check effect interaction to align with best practices](https://github.com/code-423n4/2022-01-insure-findings/issues/227) 

# Handle

defsec


# Vulnerability details

## Impact

On the InsureDAOERC20, transferFrom function is vulnerable on the re-entrancy.

## Proof of Concept

1. Navigate to the following contract. Approve function is written after transfer call. It is not possible to exploit on the current environment but that can be possible on the EVM.

```
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        _transfer(sender, recipient, amount);

        uint256 currentAllowance = _allowances[sender][_msgSender()];
        require(
            currentAllowance >= amount,
            "ERC20: transfer amount exceeds allowance"
        );

        _approve(sender, _msgSender(), currentAllowance - amount);

        return true;
    }
```


## Tools Used

Code Review

## Recommended Mitigation Steps


Follow check effect interaction pattern. Consider to use openzeppelin erc20 contract. The sample transferFrom function can be seen from below.


```
  function transferFrom(
      address sender,
      address recipient,
      uint256 amount
  ) public virtual override returns (bool) {
      uint256 currentAllowance = _allowances[sender][_msgSender()];
      if (currentAllowance != type(uint256).max) {
          require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
          unchecked {
              _approve(sender, _msgSender(), currentAllowance - amount);
          }
      }

      _transfer(sender, recipient, amount);

      return true;
  }
```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L161

