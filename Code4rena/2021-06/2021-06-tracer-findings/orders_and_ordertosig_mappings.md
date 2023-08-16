## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [orders and orderToSig mappings](https://github.com/code-423n4/2021-06-tracer-findings/issues/47) 

# Handle

pauliax


# Vulnerability details

## Impact
Contract Trader has 2 mappings: orders and orderToSig. I see that orderToSig also stores Perpetuals.Order inside it, so I wonder if it really was necessary to separate these mappings as some state (order) is duplicated among them. It may be a bit more efficient to access orders without signatures but it also makes it more error-prone as you need to keep the invariant that orders match in these mappings. Currently I don't see an exact problem as orderToSig are only set in function grabOrder and never used in code anywhere but I am not sure if it is really necessary. 

Tracer representetive's answer on Discord: 'Yeah thats a good point on the Trader mapping, one does look redundant now as they both store the order itself. I think originally one was mutated and one wasn't, but then that functionality got moved into the filled mapping anyway. Seems safe to remove orders and simply reference the orderToSig mapping'.

## Recommended Mitigation Steps
Remove orders and simply reference the orderToSig mapping.

