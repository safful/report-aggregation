## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas saving removing variable](https://github.com/code-423n4/2022-01-behodler-findings/issues/50) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Detailed description of the impact of this finding.

## Proof of Concept
Removing the variable `_redeemRate` and using only the call `redeemRate()` in the method `mint` inside the contract `RebaseProxy` it`s possible to save gas. It will save gas if the case of `transferFrom` failure.

```
function mint(address to, uint256 amount)
        public
        override
        returns (uint256)
    {
        uint256 _redeemRate = redeemRate();
        require(
            IERC20(baseToken).transferFrom(msg.sender, address(this), amount)
        );
        uint256 baseBalance = IERC20(baseToken).balanceOf(address(this));
        uint256 proxy = (baseBalance * ONE) / _redeemRate;
        _mint(to, proxy);
    }
```

## Tools Used
Manual review.

## Recommended Mitigation Steps
Remove the mentioned argument.

