## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unnecessary use of _msgSender()](https://github.com/code-423n4/2022-01-insure-findings/issues/166) 

# Handle

Jujic


# Vulnerability details

## Impact
The use of _msgSender() when there is no implementation of a meta transaction mechanism that uses it, such as EIP-2771, very slightly increases gas consumption.

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/InsureDAOERC20.sol#L105

```
function transfer(address recipient, uint256 amount)
        public
        virtual
        override
        returns (bool)
    {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }
```

## Tools Used
Remix
## Recommended Mitigation Steps
Replace _msgSender() with msg.sender if there is no mechanism to support meta-transactions like EIP-2771 implemented.

