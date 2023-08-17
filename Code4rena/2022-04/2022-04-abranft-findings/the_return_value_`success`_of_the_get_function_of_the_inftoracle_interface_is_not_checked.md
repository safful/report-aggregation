## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [The return value `success` of the get function of the INFTOracle interface is not checked](https://github.com/code-423n4/2022-04-abranft-findings/issues/21) 

# Lines of code

https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/interfaces/INFTOracle.sol#L10-L10


# Vulnerability details

## Impact
```
    function get(address pair, uint256 tokenId) external returns (bool success, uint256 rate);
```
The get function of the INFTOracle interface returns two values, but the success value is not checked when used in the NFTPairWithOracle contract. When success is false, NFTOracle may return stale data.

## Proof of Concept
https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/interfaces/INFTOracle.sol#L10-L10
https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L287-L287
https://github.com/code-423n4/2022-04-abranft/blob/5cd4edc3298c05748e952f8a8c93e42f930a78c2/contracts/NFTPairWithOracle.sol#L321-L321
## Tools Used
None
## Recommended Mitigation Steps
```
(bool success, uint256 rate) = loanParams.oracle.get(address(this), tokenId);
require(success);
```

