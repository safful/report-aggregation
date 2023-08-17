## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-03

# [The collect() function will always TRANSFER ZERO fees, losing _feesPositions without receiving fees!](https://github.com/code-423n4/2023-01-timeswap-findings/issues/121) 

# Lines of code

https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-token/src/TimeswapV2LiquidityToken.sol#L193


# Vulnerability details

## Impact
Detailed description of the impact of this finding.
The ``collect()`` function will always transfer ZERO fees. At the same time, non-zero ``_fessPosition`` will be burned. 
```
_feesPositions[id][msg.sender].burn(long0Fees, long1Fees, shortFees);
```
As a result, the contracts will be left in an inconsistent state. The user will burn ``_feesPositions`` without receiving the the  fees!

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.
The ``collect()`` function will always transfer ZERO fees   in the following line:

```
 // transfer the fees amount to the recipient
        ITimeswapV2Pool(poolPair).transferFees(param.strike, param.maturity, param.to, long0Fees, long1Fees, shortFees);

```
This is because, at this moment, the values of  ``long0Fees``, ``long1Fees``, ``shortFees`` have not been calculated yet, actually, they will be equal to zero. Therefore, no fees will be transferred. The values of  ``long0Fees``, ``long1Fees``, ``shortFees`` are calculated afterwards by the following line:
```
(long0Fees, long1Fees, shortFees) = _feesPositions[id][msg.sender].getFees(param.long0FeesDesired, param.long1FeesDesired, param.shortFeesDesired);

```
Therefore, ``ITimeswapV2Pool(poolPair).transferFees`` must be called after this line to be correct. 

## Tools Used
Remix

## Recommended Mitigation Steps
We moved the line  ``ITimeswapV2Pool(poolPair).transferFees`` after ``long0Fees``, ``long1Fees``, ``shortFees`` have been calculated first. 

```
function collect(TimeswapV2LiquidityTokenCollectParam calldata param) external returns (uint256 long0Fees, uint256 long1Fees, uint256 shortFees, bytes memory data) {
        ParamLibrary.check(param);

        bytes32 key = TimeswapV2LiquidityTokenPosition({token0: param.token0, token1: param.token1, strike: param.strike, maturity: param.maturity}).toKey();

        // start the reentrancy guard
        raiseGuard(key);

        (, address poolPair) = PoolFactoryLibrary.getWithCheck(optionFactory, poolFactory, param.token0, param.token1);


        uint256 id = _timeswapV2LiquidityTokenPositionIds[key];

        _updateFeesPositions(msg.sender, address(0), id);

        (long0Fees, long1Fees, shortFees) = _feesPositions[id][msg.sender].getFees(param.long0FeesDesired, param.long1FeesDesired, param.shortFeesDesired);

        if (param.data.length != 0)
            data = ITimeswapV2LiquidityTokenCollectCallback(msg.sender).timeswapV2LiquidityTokenCollectCallback(
                TimeswapV2LiquidityTokenCollectCallbackParam({
                    token0: param.token0,
                    token1: param.token1,
                    strike: param.strike,
                    maturity: param.maturity,
                    long0Fees: long0Fees,
                    long1Fees: long1Fees,
                    shortFees: shortFees,
                    data: param.data
                })
            );

                // transfer the fees amount to the recipient
        ITimeswapV2Pool(poolPair).transferFees(param.strike, param.maturity, param.to, long0Fees, long1Fees, shortFees);


        // burn the desired fees from the fees position
        _feesPositions[id][msg.sender].burn(long0Fees, long1Fees, shortFees);

        if (long0Fees != 0 || long1Fees != 0 || shortFees != 0) _removeTokenEnumeration(msg.sender, address(0), id, 0);

        // stop the reentrancy guard
        lowerGuard(key);
    }
```