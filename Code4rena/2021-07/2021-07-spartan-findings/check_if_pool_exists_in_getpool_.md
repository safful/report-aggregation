## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [check if pool exists in getPool ](https://github.com/code-423n4/2021-07-spartan-findings/issues/5) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function getPool doesn't check if the pool exits (e.g. it doesn't check if the resulting pool !=0)
Other functions use the results of getPool and do followup actions.

For example createSynth checks isCuratedPool(_pool) == true; if somehow isCuratedPool(0) would set to be true, then further actions could be done.
As far as I can see no actual problem occurs, but this is a dangerous construction and future code changes could introduce vulnerabilities.
Additionally the reverts that will occur if the result of getPool==0 are perhaps difficult to troubleshoot.

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/poolFactory.sol#L119
    function getPool(address token) public view returns(address pool){
        if(token == address(0)){
            pool = mapToken_Pool[WBNB];   // Handle BNB
        } else {
            pool = mapToken_Pool[token];  // Handle normal token
        } 
        return pool;
    }

function createPoolADD(uint256 inputBase, uint256 inputToken, address token) external payable returns(address pool){
        require(getPool(token) == address(0)); // Must be a valid token
     
function createPool(address token) external onlyDAO returns(address pool){
        require(getPool(token) == address(0)); // Must be a valid token
     
// https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/synthFactory.sol#L37
 function createSynth(address token) external returns(address synth){
        require(getSynth(token) == address(0), "exists"); // Synth must not already exist
        address _pool = iPOOLFACTORY(_DAO().POOLFACTORY()).getPool(token); // Get pool address
        require(iPOOLFACTORY(_DAO().POOLFACTORY()).isCuratedPool(_pool) == true, "!curated"); // Pool must be Curated


## Tools Used

## Recommended Mitigation Steps
In function getPool add something like:
require  (pool !=0, "Pool doesn't exist");

Note: the functions createPoolADD and createPool also have to be changed, to use a different way to verify the pool doesn't exist.


