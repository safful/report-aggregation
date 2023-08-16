## Tags

- bug
- G (Gas Optimization)
- sponsor acknowledged
- sponsor confirmed

# [Harvester: Simpler implementation for canHarvest()](https://github.com/code-423n4/2021-09-yaxis-findings/issues/66) 

# Handle

hickuphh3


# Vulnerability details

### Recommended Mitigation Steps

The negation of the disjunction form of the `canHarvest()` function will help save gas. In other words, instead of `!(A || B)`, return `(!A && !B)`.

```jsx
function canHarvest(
	address _vault
)
  public
  view
  returns (bool)
{
  Strategy storage strategy = strategies[_vault];
  // only can harvest if there are strategies, and when sufficient time has elapsed
  // solhint-disable-next-line not-rely-on-time
	return (strategy.addresses.length > 0 && strategy.lastCalled <= block.timestamp.sub(strategy.timeout));
}
```

