## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`AlchemistVault.sol` can be optimised](https://github.com/code-423n4/2021-11-yaxis-findings/issues/100) 

# Handle

0x0x0x


# Vulnerability details

## Proof of Concept
There is no need to cache calculation steps between the return values.
`L98-L122` is as follows:
```
        uint256 _endingBalance = _token.balanceOf(_recipient);
        uint256 _withdrawnAmount = _token.balanceOf(_recipient).sub(_startingBalance);

        uint256 _endingTotalValue = _self.totalValue();
        uint256 _decreasedValue = _startingTotalValue.sub(_endingTotalValue);
```

Which can be replaced by following code to save gas:

```
        uint256 _withdrawnAmount = _token.balanceOf(_recipient).sub(_startingBalance);

        uint256 _decreasedValue = _startingTotalValue.sub(_self.totalValue());
```

## Tools Used

Manual analysis

