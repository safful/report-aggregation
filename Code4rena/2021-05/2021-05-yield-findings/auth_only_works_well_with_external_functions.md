## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [auth only works well with external functions](https://github.com/code-423n4/2021-05-yield-findings/issues/4) 

# Handle

gpersoon


# Vulnerability details

## Impact
The auth modifier of AccessControl.sol doesn't work as you would expect. 
It checks if you are authorized for "msg.sig", however msg.sig is the signature of the first function you have called, not of the current function.
So if you call function A, which calls function B, the "auth" modifier of function B checks if you are authorized for function A!

There is a difference between external an public functions. For external functions this works as expected because a fresh call (with a new msg.sig) is always made.
However with a public functions, which are called from within the same contract, this doesn't happen and the problem described above occurs.
See in the proof of concept for a piece of code which shows the problem.
In the code there are several functions which have public and auth combined, see also in the proof of concept .

In the current codebase I couldn't find a problem situation, however this could be accidentally introduced with future changes.
If could also be introduced via the _moduleCall of Ladle.sol, which allows functions to be defined which might call the public functions.

## Proof of Concept
### auth
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/utils/access/AccessControl.sol#L90
modifier auth() {
        require (_hasRole(msg.sig, msg.sender), "Access denied");
        _;
    }

### example
pragma solidity ^0.8.0;
contract TestMsgSig { 
    event log(bytes4);

    function setFeePublic(uint256) public  {
         emit log(this.setFeePublic.selector);
         emit log(msg.sig);
    }
    function setFeeExternal(uint256) external  {
         emit log(this.setFeeExternal.selector);
         emit log(msg.sig);
    }

    function TestPublic() public {
        setFeePublic(2);
    }

    function TestExternal() public {
       this.setFeeExternal(2);
    }
}
### occurrences of public auth
Wand.sol:   function addAsset(bytes6 assetId,address asset) public auth {
Wand.sol:    function makeBase(bytes6 assetId, IMultiOracleGov oracle, address rateSource, address chiSource) public auth {
Wand.sol:    function makeIlk(bytes6 baseId, bytes6 ilkId, IMultiOracleGov oracle, address spotSource, uint32 ratio, uint96 max, uint24 min, uint8 dec) public auth {
Wand.sol:     function addSeries(...  ) public auth {
Witch.sol:    function setAuctionTime(uint128 auctionTime_) public auth {
Witch.sol:    function setInitialProportion(uint128 initialProportion_) public auth { 
Ladle.sol:     function setFee(uint256 fee)         public        auth     
Join.sol:    function setFlashFeeFactor(uint256 flashFeeFactor_) public   auth     { 
oracles\chainlink\ChainlinkMultiOracle.sol:    function setSource(bytes6 base, bytes6 quote, address source) public auth {
oracles\chainlink\ChainlinkMultiOracle.sol:    function setSources(bytes6[] memory bases, bytes6[] memory quotes, address[] memory sources_) public auth {
oracles\compound\CompoundMultiOracle.sol:    function setSource(bytes6 base, bytes6 kind, address source) public auth {
oracles\compound\CompoundMultiOracle.sol:    function setSources(bytes6[] memory bases, bytes6[] memory kinds, address[] memory sources_) public auth {
oracles\uniswap\UniswapV3Oracle.sol:    function setSecondsAgo(uint32 secondsAgo_) public auth {
oracles\uniswap\UniswapV3Oracle.sol:    function setSource(bytes6 base, bytes6 quote, address source) public auth {
oracles\uniswap\UniswapV3Oracle.sol:    function setSources(bytes6[] memory bases, bytes6[] memory quotes, address[] memory sources_) public auth {
fytoken.sol:  function setOracle(IOracle oracle_)  public  auth     {
 
## Tools Used
grep

## Recommended Mitigation Steps
make sure all auth functions use external  (still error prone)
or change the modifier to something like:

   modifier auth(bytes4 fs) {
        require (msg.sig == fs,"Wrong selector");
        require (_hasRole(msg.sig, msg.sender), "Access denied");
        _;
    }

    function setFee(uint256) public auth(this.setFee.selector) {
       .....
    }



