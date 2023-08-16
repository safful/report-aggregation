## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Allowance checks not correctly implemented](https://github.com/code-423n4/2022-01-insure-findings/issues/226) 

# Handle

defsec


# Vulnerability details

## Impact

On the ERC20, There is a known problem named as Approve/TransferFrom race condition. On the transferFrom, allowance max check has not been added.

## Proof of Concept

1. Navigate to the following contract.

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/InsureDAOERC20.sol#L152

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

2. Max Allowance check has not been added into the function. ERC20 standart (Openzeppelin) is not followed.

## Tools Used

None

## Recommended Mitigation Steps

Consider to use openzeppelin erc20 contract. The sample transferFrom function can be seen from below.


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

