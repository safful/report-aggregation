## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Oracles

# [updateTime of get is 0](https://github.com/code-423n4/2021-08-yield-findings/issues/7) 

# Handle

gpersoon


# Vulnerability details

## Impact
In function get of CompositeMultiOracle the updateTime is not initialized, so it will be 0

Function _get has the following statement:
   updateTimeOut = (updateTimeOut < updateTimeIn) ? updateTimeOut : updateTimeIn;    

updateTimeIn ==0 ==>  (updateTimeOut < updateTimeIn)== false ==> result of the expression is updateTimeIn == 0 ==> updateTimeOut =0

So this means the function get will always return updateTime==0

The updateTime result of the function get doesn't seem to be used in the code so the risk is low.
If would only be relevant for future code updates.


## Proof of Concept
//https://github.com/code-423n4/2021-08-yield/blob/main/contracts/oracles/composite/CompositeMultiOracle.sol#L94

function get(bytes32 base, bytes32 quote, uint256 amount)  external virtual override  returns (uint256 value, uint256 updateTime)  {
...
        for (uint256 p = 0; p < path.length; p++) {
            (price, updateTime) = _get(base_, path[p], price, updateTime);

function _get(bytes6 base, bytes6 quote, uint256 priceIn, uint256 updateTimeIn)  private returns (uint priceOut, uint updateTimeOut) {
    ...
        (priceOut, updateTimeOut) = IOracle(source.source).get(base, quote, 10 ** source.decimals);    // Get price for one unit
      ...
        updateTimeOut = (updateTimeOut < updateTimeIn) ? updateTimeOut : updateTimeIn;                 // Take the oldest update time
    }

## Tools Used

## Recommended Mitigation Steps
In function get, add the following in the beginning of the function:
updateTime = block.timestamp;

