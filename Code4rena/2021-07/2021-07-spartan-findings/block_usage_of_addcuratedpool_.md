## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Block usage of addCuratedPool ](https://github.com/code-423n4/2021-07-spartan-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function curatedPoolCount() contains a for loop over the array arrayPools.
If arrayPools would be too big then the loop would run out of gas and curatedPoolCount() would revert.
This would mean that addCuratedPool() cannot be executed anymore (because it calls curatedPoolCount() )

The array arrayPools can be increased in size arbitrarily by repeatedly doing the following:
- create a pool with createPoolADD()  (which requires 10,000 SPARTA)
- empty the pool with remove() of Pool.sol, which gives back the SPARTA tokens
These actions will use gas to perform.

## Proof of Concept
//https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/poolFactory.sol#L45
 function createPoolADD(uint256 inputBase, uint256 inputToken, address token) external payable returns(address pool){
        require(getPool(token) == address(0)); // Must be a valid token
        require((inputToken > 0 && inputBase >= (10000*10**18)), "!min"); // User must add at least 10,000 SPARTA liquidity & ratio must be finite
        Pool newPool; address _token = token;
        if(token == address(0)){_token = WBNB;} // Handle BNB -> WBNB
        require(_token != BASE && iBEP20(_token).decimals() == 18); // Token must not be SPARTA & it's decimals must be 18
        newPool = new Pool(BASE, _token); // Deploy new pool
        pool = address(newPool); // Get address of new pool
        mapToken_Pool[_token] = pool; // Record the new pool address in PoolFactory
        _handleTransferIn(BASE, inputBase, pool); // Transfer SPARTA liquidity to new pool
        _handleTransferIn(token, inputToken, pool); // Transfer TOKEN liquidity to new pool
        arrayPools.push(pool); // Add pool address to the pool array
       ..

function curatedPoolCount() internal view returns (uint){
        uint cPoolCount; 
        for(uint i = 0; i< arrayPools.length; i++){
            if(isCuratedPool[arrayPools[i]] == true){
                cPoolCount += 1;
            }
        }
        return cPoolCount;
    }

 function addCuratedPool(address token) external onlyDAO {
        ...
        require(curatedPoolCount() < curatedPoolSize, "maxCurated"); // Must be room in the Curated list

//https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Pool.sol#L187
  function remove() external returns (uint outputBase, uint outputToken) {
        return removeForMember(msg.sender);
    } 

    // Contract removes liquidity for the user
    function removeForMember(address member) public returns (uint outputBase, uint outputToken) {
        uint256 _actualInputUnits = balanceOf(address(this)); // Get the received LP units amount
        outputBase = iUTILS(_DAO().UTILS()).calcLiquidityHoldings(_actualInputUnits, BASE, address(this)); // Get the SPARTA value of LP units
        outputToken = iUTILS(_DAO().UTILS()).calcLiquidityHoldings(_actualInputUnits, TOKEN, address(this)); // Get the TOKEN value of LP units
        _decrementPoolBalances(outputBase, outputToken); // Update recorded BASE and TOKEN amounts
        _burn(address(this), _actualInputUnits); // Burn the LP tokens
        iBEP20(BASE).transfer(member, outputBase); // Transfer the SPARTA to user
        iBEP20(TOKEN).transfer(member, outputToken); // Transfer the TOKENs to user
        emit RemoveLiquidity(member, outputBase, outputToken, _actualInputUnits);
        return (outputBase, outputToken);
    }
## Tools Used

## Recommended Mitigation Steps
Create a variable curatedPoolCount
and increase it in addCuratedPool
and decrease it in removeCuratedPool


