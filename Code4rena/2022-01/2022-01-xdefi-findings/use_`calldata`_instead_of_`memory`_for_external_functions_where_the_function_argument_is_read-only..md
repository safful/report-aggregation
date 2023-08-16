## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use `calldata` instead of `memory` for external functions where the function argument is read-only.](https://github.com/code-423n4/2022-01-xdefi-findings/issues/29) 

# Handle

Dravee


# Vulnerability details

## Impact  
On external functions, when using the `memory` keyword with a function argument, what's happening is that a `memory` acts as an intermediate.  
  
Reading directly from `calldata` using `calldataload` instead of going via `memory` saves the gas from the intermediate memory operations that carry the values.  
  
As an extract from [https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory](https://ethereum.stackexchange.com/questions/74442/when-should-i-use-calldata-and-when-should-i-use-memory) :  
> `memory` and `calldata` (as well as `storage`) are keywords that define the data area where a variable is stored. To answer your question directly, `memory` should be used when declaring variables (both function parameters as well as inside the logic of a function) that you want stored in memory (temporary), and `calldata` _must_ be used when declaring an **external** function's **dynamic** parameters. The easiest way to think about the difference is that `calldata` is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.  
  
## Proof of Concept  
```  
interfaces\IXDEFIDistribution.sol:55:    function baseURI() external view returns (string memory baseURI_);
interfaces\IXDEFIDistribution.sol:74:    function setBaseURI(string memory baseURI_) external;
interfaces\IXDEFIDistribution.sol:77:    function setLockPeriods(uint256[] memory durations_, uint8[] memory multipliers) external;
interfaces\IXDEFIDistribution.sol:106:    function relockBatch(uint256[] memory tokenIds_, uint256 lockAmount_, uint256 duration_, address destination_) external returns (uint256 amountUnlocked_, uint256 
newTokenId_);
interfaces\IXDEFIDistribution.sol:109:    function unlockBatch(uint256[] memory tokenIds_, address destination_) external returns (uint256 amountUnlocked_);
interfaces\IXDEFIDistribution.sol:119:    function merge(uint256[] memory tokenIds_, address destination_) external returns (uint256 tokenId_);
interfaces\IXDEFIDistribution.sol:125:    function tokenURI(uint256 tokenId_) external view returns (string memory tokenURI_);
XDEFIDistribution.sol:73:    function setBaseURI(string memory baseURI_) external onlyOwner {
XDEFIDistribution.sol:77:    function setLockPeriods(uint256[] memory durations_, uint8[] memory multipliers) external onlyOwner {
XDEFIDistribution.sol:165:    function relockBatch(uint256[] memory tokenIds_, uint256 lockAmount_, uint256 duration_, address destination_) external noReenter returns (uint256 amountUnlocked_, uint256 
newTokenId_) {
XDEFIDistribution.sol:186:    function unlockBatch(uint256[] memory tokenIds_, address destination_) external noReenter returns (uint256 amountUnlocked_) {
XDEFIDistribution.sol:205:    function merge(uint256[] memory tokenIds_, address destination_) external returns (uint256 tokenId_) {
```  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Use `calldata` instead of `memory` for external functions where the function argument is read-only.

