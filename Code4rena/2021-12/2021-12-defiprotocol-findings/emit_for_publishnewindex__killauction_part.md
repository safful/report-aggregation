## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Emit for publishNewIndex / killAuction part](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/32) 

# Handle

gpersoon


# Vulnerability details

## Impact
Most of the public functions have an emit, however in function publishNewIndex(), there is no emit for the "killauction" part.
It might be useful to have an emit there too.

## Proof of Concept
https://github.com/code-423n4/2021-12-defiprotocol/blob/205d3766044171e325df6a8bf2e79b37856eece1/contracts/contracts/Basket.sol#L216-L244

```JS
 function publishNewIndex(address[] memory _tokens, uint256[] memory _weights, uint256 _minIbRatio) onlyPublisher public override {
   ...
            } else {
                auction.killAuction();
                pendingWeights.tokens = _tokens;
                pendingWeights.weights = _weights;
                pendingWeights.timestamp = block.timestamp;
                pendingWeights.minIbRatio = _minIbRatio;
               // no emit
            }
```

## Tools Used

## Recommended Mitigation Steps
Possibly add an emit in the "killauction" part


