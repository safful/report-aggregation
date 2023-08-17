## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ `getUnderlyingPrice()` should return `0` when errored](https://github.com/code-423n4/2022-09-canto-findings/issues/93) 

# Lines of code

https://github.com/code-423n4/2022-09-canto/blob/65fbb8b9de22cf8f8f3d742b38b4be41ee35c468/src/Swap/BaseV1-periphery.sol#L487-L522


# Vulnerability details

## Impact

The `Comptroller` is expecting `oracle.getUnderlyingPrice` to return `0` for errors (Compound style returns, no revert). The current implementation will throw errors, resulting in the consumer of the oracle getting unexpected errors.

## Proof of Concept

```solidity
function getUnderlyingPrice(CToken ctoken) external override view returns(uint) {
         address underlying;
        { //manual scope to pop symbol off of stack
        string memory symbol = ctoken.symbol();
        if (compareStrings(symbol, "cCANTO")) {
            underlying = address(wcanto);
            return getPriceNote(address(wcanto), false);
        } else {
            underlying = address(ICErc20(address(ctoken)).underlying()); // We are getting the price for a CErc20 lending market
        }
        //set price statically to 1 when the Comptroller is retrieving Price
        if (compareStrings(symbol, "cNOTE")) { // note in terms of note will always be 1 
            return 1e18; // Stable coins supported by the lending market are instantiated by governance and their price will always be 1 note
        } 
        else if (compareStrings(symbol, "cUSDT") && (msg.sender == Comptroller )) {
            uint decimals = erc20(underlying).decimals();
            return 1e18 * 1e18 / (10 ** decimals); //Scale Price as a mantissa to maintain precision in comptroller
        } 
        else if (compareStrings(symbol, "cUSDC") && (msg.sender == Comptroller)) {
            uint decimals = erc20(underlying).decimals();
            return 1e18 * 1e18 / (10 ** decimals); //Scale Price as a mantissa to maintain precision in comptroller
        }
        }
        
        if (isPair(underlying)) { // this is an LP Token
            return getPriceLP(IBaseV1Pair(underlying));
        }
        // this is not an LP Token
        else {
            if (isStable[underlying]) {
                return getPriceNote(underlying, true); // value has already been scaled
            }

            return getPriceCanto(underlying) * getPriceNote(address(wcanto), false) / 1e18;
        }   
    }
```


The `Comptroller` is expecting `oracle.getUnderlyingPrice` to return `0` for errors (Compound style returns, no revert).

However, the current implementation will revert when errored:

https://github.com/code-423n4/2022-09-canto/blob/65fbb8b9de22cf8f8f3d742b38b4be41ee35c468/src/Swap/BaseV1-periphery.sol#L549-L593

```solidity
function getPriceLP(IBaseV1Pair pair) internal view returns(uint) {
        uint[] memory supply = pair.sampleSupply(8, 1);
        uint[] memory prices; 
        uint[] memory unitReserves; 
        uint[] memory assetReserves; 
        address token0 = pair.token0();
        address token1 = pair.token1();
        uint decimals;
```

https://github.com/code-423n4/2022-09-canto/blob/65fbb8b9de22cf8f8f3d742b38b4be41ee35c468/src/Swap/BaseV1-core.sol#L271-L289

```solidity
function sampleSupply(uint points, uint window) public view returns (uint[] memory) {
        uint[] memory _totalSupply = new uint[](points);
        
        uint lastIndex = observations.length-1;
        require(lastIndex >= points * window, "PAIR::NOT READY FOR PRICING");
        uint i = lastIndex - (points * window); // point from which to begin the sample
        uint nextIndex = 0;
        uint index = 0;
        uint timeElapsed;

        for(; i < lastIndex; i+=window) {
            nextIndex = i + window;
            timeElapsed = observations[nextIndex].timestamp - observations[i].timestamp;
            _totalSupply[index] = (observations[nextIndex].totalSupplyCumulative - observations[i].totalSupplyCumulative) / timeElapsed;
            index = index + 1;
        }

        return _totalSupply;
    }
```

## Recommended Mitigation Steps

Consider using `try catch` and return 0 when errored.