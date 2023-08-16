## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [more efficient calls to DAO functions](https://github.com/code-423n4/2021-07-spartan-findings/issues/4) 

# Handle

gpersoon


# Vulnerability details

## Impact
Sometimes the reference to function calls, that are done via the DAO, are looked up multiple times in one function call.
For example mintSynth calls:  ​
-   _DAO() 4x
-   _DAO().UTILS() 3x

This can be done more efficient by caching the result of _DAO() and _DAO().UTILS()

f## Proof of Concept
// https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Pool.sol#L229
​function mintSynth(address synthOut, address member) external returns(uint outputAmount, uint fee) {
       ​require(iSYNTHFACTORY(_DAO().SYNTHFACTORY()).isSynth(synthOut) == true, "!synth"); // Must be a valid Synth
     ​..
       ​uint output = iUTILS(_DAO().UTILS()).calcSwapOutput(_actualInputBase, baseAmount, tokenAmount); // Calculate value of swapping SPARTA to the relevant underlying TOKEN
       ​uint _liquidityUnits = iUTILS(_DAO().UTILS()).calcLiquidityUnitsAsym(_actualInputBase, address(this)); // Calculate LP tokens to be minted
      ​..
       ​uint _fee = iUTILS(_DAO().UTILS()).calcSwapFee(_actualInputBase, baseAmount, tokenAmount); // Calc slip fee in TOKEN
       ​fee = iUTILS(_DAO().UTILS()).calcSpotValueInBase(TOKEN, _fee); // Convert TOKEN fee to SPARTA
       ​

function _DAO() internal view returns(iDAO) {
       ​return iBASE(BASE).DAO();
   ​}

//https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L624
​function UTILS() public view returns(iUTILS){
       ​if(daoHasMoved){
           ​return Dao(DAO).UTILS();
       ​} else {
           ​return _UTILS;
       ​}
   ​}

## Tools Used

## Recommended Mitigation Step
Cache _DAO() and cache the sub functions like: _DAO().UTILS())
If called multiple times from function

