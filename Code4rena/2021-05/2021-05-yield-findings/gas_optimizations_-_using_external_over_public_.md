## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas optimizations - using external over public ](https://github.com/code-423n4/2021-05-yield-findings/issues/60) 

# Handle

a_delamo


# Vulnerability details

## Impact
The following methods could be external instead of public

```
level(bytes12) should be declared external:
        - Cauldron.level(bytes12) (contracts/Cauldron.sol#513-521)
mature(bytes6) should be declared external:
        - Cauldron.mature(bytes6) (contracts/Cauldron.sol#524-532)
accrual(bytes6) should be declared external:
        - Cauldron.accrual(bytes6) (contracts/Cauldron.sol#546-553)
setFlashFeeFactor(uint256) should be declared external:
        - Join.setFlashFeeFactor(uint256) (contracts/Join.sol#33-36)
maxFlashLoan(address) should be declared external:
        - Join.maxFlashLoan(address) (contracts/Join.sol#90-97)
flashFee(address,uint256) should be declared external:
        - Join.flashFee(address,uint256) (contracts/Join.sol#105-113)
flashLoan(IERC3156FlashBorrower,address,uint256,bytes) should be declared external:
        - Join.flashLoan(IERC3156FlashBorrower,address,uint256,bytes) (contracts/Join.sol#132-151)
setFee(uint256) should be declared external:
        - Ladle.setFee(uint256) (contracts/Ladle.sol#102-105)
addAsset(bytes6,address) should be declared external:
        - Wand.addAsset(bytes6,address) (contracts/Wand.sol#49-61)
makeBase(bytes6,IMultiOracleGov,address,address) should be declared external:
        - Wand.makeBase(bytes6,IMultiOracleGov,address,address) (contracts/Wand.sol#65-78)
makeIlk(bytes6,bytes6,IMultiOracleGov,address,uint32,uint96,uint24,uint8) should be declared external:
        - Wand.makeIlk(bytes6,bytes6,IMultiOracleGov,address,uint32,uint96,uint24,uint8) (contracts/Wand.sol#81-94)
addSeries(bytes6,bytes6,uint32,bytes6[],string,string) should be declared external:
        - Wand.addSeries(bytes6,bytes6,uint32,bytes6[],string,string) (contracts/Wand.sol#98-154)
setAuctionTime(uint128) should be declared external:
        - Witch.setAuctionTime(uint128) (contracts/Witch.sol#41-44)
setInitialProportion(uint128) should be declared external:
        - Witch.setInitialProportion(uint128) (contracts/Witch.sol#47-51)
grab(bytes12) should be declared external:
        - Witch.grab(bytes12) (contracts/Witch.sol#54-59)
buy(bytes12,uint128,uint128) should be declared external:
        - Witch.buy(bytes12,uint128,uint128) (contracts/Witch.sol#62-99)
mint(address,uint256) should be declared external:
        - DAIMock.mint(address,uint256) (contracts/mocks/DAIMock.sol#36-38)
mint(address,uint256) should be declared external:
        - ERC20Mock.mint(address,uint256) (contracts/mocks/ERC20Mock.sol#11-13)
mint(address,uint256) should be declared external:
        - RestrictedERC20Mock.mint(address,uint256) (contracts/mocks/RestrictedERC20Mock.sol#12-14)
burn(address,uint256) should be declared external:
        - RestrictedERC20Mock.burn(address,uint256) (contracts/mocks/RestrictedERC20Mock.sol#17-19)
pull(address,uint256) should be declared external:
        - GemJoinMock.pull(address,uint256) (contracts/mocks/TLMMock.sol#14-16)
mint(address,uint256) should be declared external:
        - USDCMock.mint(address,uint256) (contracts/mocks/USDCMock.sol#13-15)
withdraw(uint256) should be declared external:
        - WETH9Mock.withdraw(uint256) (contracts/mocks/WETH9Mock.sol#21-26)
totalSupply() should be declared external:
        - WETH9Mock.totalSupply() (contracts/mocks/WETH9Mock.sol#28-30)
latestRoundData() should be declared external:
        - ChainlinkAggregatorV3Mock.latestRoundData() (contracts/mocks/oracles/chainlink/ChainlinkAggregatorV3Mock.sol#22-34)
exchangeRateCurrent() should be declared external:
        - CTokenChiMock.exchangeRateCurrent() (contracts/mocks/oracles/compound/CTokenChiMock.sol#12-14)
tickSpacing() should be declared external:
        - UniswapV3PoolMock.tickSpacing() (contracts/mocks/oracles/uniswap/UniswapV3PoolMock.sol#27-29)
maxLiquidityPerTick() should be declared external:
        - UniswapV3PoolMock.maxLiquidityPerTick() (contracts/mocks/oracles/uniswap/UniswapV3PoolMock.sol#31-33)
setSources(bytes6[],bytes6[],address[]) should be declared external:
        - ChainlinkMultiOracle.setSources(bytes6[],bytes6[],address[]) (contracts/oracles/chainlink/ChainlinkMultiOracle.sol#56-68)
peek(bytes32,bytes32,uint256) should be declared external:
        - ChainlinkMultiOracle.peek(bytes32,bytes32,uint256) (contracts/oracles/chainlink/ChainlinkMultiOracle.sol#105-113)
        - CompoundMultiOracle.peek(bytes32,bytes32,uint256) (contracts/oracles/compound/CompoundMultiOracle.sol#78-86)
        - UniswapV3Oracle.peek(bytes32,bytes32,uint256) (contracts/oracles/uniswap/UniswapV3Oracle.sol#126-132)
get(bytes32,bytes32,uint256) should be declared external:
        - ChainlinkMultiOracle.get(bytes32,bytes32,uint256) (contracts/oracles/chainlink/ChainlinkMultiOracle.sol#119-127)
        - CompoundMultiOracle.get(bytes32,bytes32,uint256) (contracts/oracles/compound/CompoundMultiOracle.sol#92-100)
        - UniswapV3Oracle.get(bytes32,bytes32,uint256) (contracts/oracles/uniswap/UniswapV3Oracle.sol#138-144)
setSources(bytes6[],bytes6[],address[]) should be declared external:
        - CompoundMultiOracle.setSources(bytes6[],bytes6[],address[]) (contracts/oracles/compound/CompoundMultiOracle.sol#37-48)
setSecondsAgo(uint32) should be declared external:
        - UniswapV3Oracle.setSecondsAgo(uint32) (contracts/oracles/uniswap/UniswapV3Oracle.sol#44-48)
setSources(bytes6[],bytes6[],address[]) should be declared external:
        - UniswapV3Oracle.setSources(bytes6[],bytes6[],address[]) (contracts/oracles/uniswap/UniswapV3Oracle.sol#73-85)
transferOwnership(address) should be declared external:
        - Ownable.transferOwnership(address) (contracts/utils/access/Ownable.sol#25-28)
tokenSymbol(address) should be declared external:
        - SafeERC20Namer.tokenSymbol(address) (contracts/utils/token/SafeERC20Namer.sol#87-95)
tokenName(address) should be declared external:
        - SafeERC20Namer.tokenName(address) (contracts/utils/token/SafeERC20Namer.sol#98-106)
setParameter(bytes32,int128) should be declared external:
        - Pool.setParameter(bytes32,int128) (contracts/yieldspace/Pool.sol#135-141)
getK() should be declared external:
        - Pool.getK() (contracts/yieldspace/Pool.sol#144-147)
getG1() should be declared external:
        - Pool.getG1() (contracts/yieldspace/Pool.sol#150-152)
getG2() should be declared external:
        - Pool.getG2() (contracts/yieldspace/Pool.sol#155-157)
getCache() should be declared external:
        - Pool.getCache() (contracts/yieldspace/Pool.sol#175-185)
fyTokenOutForBaseIn(uint128,uint128,uint128,uint128,int128,int128) should be declared external:
        - YieldMath.fyTokenOutForBaseIn(uint128,uint128,uint128,uint128,int128,int128) (contracts/yieldspace/YieldMath.sol#657-694)
baseOutForFYTokenIn(uint128,uint128,uint128,uint128,int128,int128) should be declared external:
        - YieldMath.baseOutForFYTokenIn(uint128,uint128,uint128,uint128,int128,int128) (contracts/yieldspace/YieldMath.sol#707-744)
fyTokenInForBaseOut(uint128,uint128,uint128,uint128,int128,int128) should be declared external:
        - YieldMath.fyTokenInForBaseOut(uint128,uint128,uint128,uint128,int128,int128) (contracts/yieldspace/YieldMath.sol#757-797)
baseInForFYTokenOut(uint128,uint128,uint128,uint128,int128,int128) should be declared external:
        - YieldMath.baseInForFYTokenOut(uint128,uint128,uint128,uint128,int128,int128) (contracts/yieldspace/YieldMath.sol#811-848)
```

Here more information about the gas optimizations of external vs public: https://gus-tavo-guim.medium.com/public-vs-external-functions-in-solidity-b46bcf0ba3ac

## Tools Used

Slither


