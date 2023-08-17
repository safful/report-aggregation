## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Period Size not updated on creating new Pair](https://github.com/code-423n4/2022-09-canto-findings/issues/76) 

# Lines of code

https://github.com/code-423n4/2022-09-canto/blob/main/src/Swap/BaseV1-core.sol#L598


# Vulnerability details

## Impact
The period size is not update to current while creating a new pair. This means even if period size has been reduced from default value, this new pair will still point to the higher default value

## Proof of Concept

1. Assume Pair P1,P2 exists in BaseV1Factory with default period size as 1800

2. Admin decides to decrease the period size to 900 using [setPeriodSize](https://github.com/code-423n4/2022-09-canto/blob/main/src/Swap/BaseV1-core.sol#L560) function

```
function setPeriodSize(uint newPeriod) external {
        require(msg.sender == admin);
        require(newPeriod <= MaxPeriod);

        for (uint i; i < allPairs.length; ) {
            BaseV1Pair(allPairs[i]).setPeriodSize(newPeriod);
            unchecked {++i;}
        }
    }
```

3. This changes period size of P1, P2 to 900

4. Admin creates a new Pair P3 using [createPair](https://github.com/code-423n4/2022-09-canto/blob/main/src/Swap/BaseV1-core.sol#L598) function

```
function createPair(address tokenA, address tokenB, bool stable) external returns (address pair) {
        require(tokenA != tokenB, "IA"); // BaseV1: IDENTICAL_ADDRESSES
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        require(token0 != address(0), "ZA"); // BaseV1: ZERO_ADDRESS
        require(getPair[token0][token1][stable] == address(0), "PE"); // BaseV1: PAIR_EXISTS - single check is sufficient
        bytes32 salt = keccak256(abi.encodePacked(token0, token1, stable)); // notice salt includes stable as well, 3 parameters
        (_temp0, _temp1, _temp) = (token0, token1, stable);
        pair = address(new BaseV1Pair{salt:salt}());
        getPair[token0][token1][stable] = pair;
        getPair[token1][token0][stable] = pair; // populate mapping in the reverse direction
        allPairs.push(pair);
        isPair[pair] = true;
        emit PairCreated(token0, token1, stable, pair, allPairs.length);
    }
```

5. A new Pair is created but the period size is not updated which means P3's period size will be 1800 instead of 900 which is incorrect

## Recommended Mitigation Steps
Add a new variable which stores the updated period size. Once a pair is created, update its period size using this new variable

```
uint periodSizeUpdated=1800;

function setPeriodSize(uint newPeriod) external {
        ...
periodSizeUpdated=newPeriod;
    }

function createPair(address tokenA, address tokenB, bool stable) external returns (address pair) {
...
BaseV1Pair(pair).setPeriodSize(newPeriod);

}
```