## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Caching variables](https://github.com/code-423n4/2022-01-insure-findings/issues/178) 

# Handle

Jujic


# Vulnerability details

## Impact
Some of the variables can be cached to slightly reduce gas usage

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L343

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L406-L407

https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L461-L479

```
function withdrawRedundant(address _token, address _to)
        external
        override
        onlyOwner
    {
        if (
            _token == address(token) &&
            balance < IERC20(token).balanceOf(address(this))
        ) {
            uint256 _redundant = IERC20(token).balanceOf(address(this)) -
                balance;
            IERC20(token).safeTransfer(_to, _redundant);
        } else if (IERC20(_token).balanceOf(address(this)) > 0) {
            IERC20(_token).safeTransfer(
                _to,
                IERC20(_token).balanceOf(address(this))
            );
        }
    }
```

## Tools Used
Remix
## Recommended Mitigation Steps
Consider caching those variable for read and make sure write back to storage
Example:
```
bal =  IERC20(_token).balanceOf(address(this);
```

