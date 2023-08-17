## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Calculated `token0TVL` may be zero under certain scenarios](https://github.com/code-423n4/2022-09-canto-findings/issues/41) 

# Lines of code

https://github.com/code-423n4/2022-09-canto/blob/main/src/Swap/BaseV1-periphery.sol#L582


# Vulnerability details

## Details & Impact

```solidity
uint token0TVL = assetReserves[i] * (prices[i] / decimals);
```

Because of the brackets, the division of `prices[i] / decimals` is executed before multiplication, causing `token0TVL` to potentially be zero.

## Proof of Concept

Add the following test in `oracle.test.ts`. Note: `getPriceLP()` should have its visibility changed from internal to public as the test relies on it.

To summarise what the test is doing, a stablecoin of 24 decimals is deployed, whose address will be greater than the `note` address so that `token0 = note`. It will enter the following case:

```solidity
if (pair.stable()) { // stable pairs will be priced in terms of Note
  if (token0 == note) { //token0 is the unit, token1 will be priced with respect to this asset initially
      decimals = 10 ** (erc20(token1).decimals()); // we must normalize the price of token1 to 18 decimals
      prices = pair.sample(token1, decimals, 8, 1);
      (unitReserves, assetReserves) = pair.sampleReserves(8, 1);
```

such that the `prices`’s denomination is smaller than the stablecoin’s decimals of 24.

To see the difference in test results, apply the recommended fix after running the test once. In essence, the LP’s price will double from `999500000001499` to `1999999998838589`, which is expected since the LP token should be worth the combined value of both stablecoins.

```solidity
it.only("will have 0 token0TVL", async () => {
  // NOTE: change getPriceLP() from internal to public so that function can be called
  let tokenFactory = await ethers.getContractFactory("ERC20", dep)
  let stablecoin = await tokenFactory.deploy("STABLE","STABLE",ethers.utils.parseUnits("100000", "24"), 24)
  await stablecoin.deployed()
  // we want note to be token0
  // redeploy till it is
  while (stablecoin.address < note.address) {
      stablecoin = await tokenFactory.deploy("STABLE","STABLE",ethers.utils.parseUnits("100000", "24"), 24)
      await stablecoin.deployed()
  }
  // give token approvals to router
  let noteIn = ethers.utils.parseUnits("10000", "18")
  let stableIn = ethers.utils.parseUnits("10000", "24")
  await (await note.approve(router.address, ethers.constants.MaxUint256)).wait()
  await (await stablecoin.approve(router.address, ethers.constants.MaxUint256)).wait()

  // borrow note
  await (await comptroller._supportMarket(cUsdc.address)).wait()
  // set collateral factors for cCanto 
  await (await comptroller._setCollateralFactor(cUsdc.address, ethers.utils.parseUnits("0.9", "18"))).wait()
  // borrow note against usdc 
  await (await comptroller.enterMarkets([cUsdc.address, cNote.address])).wait()
  await (await usdc.approve(cUsdc.address, ethers.utils.parseUnits("1000"))).wait()
  // supply usdc
  await (await cUsdc.mint(ethers.utils.parseUnits("100000000", "6"))).wait()
  // borrow note
  await (await cNote.borrow(ethers.utils.parseUnits("9000000", "18"))).wait()

  // add liquidity
  await (await router.addLiquidity(
      note.address,
      stablecoin.address,
      true,
      noteIn,
      stableIn,
      0,
      0,
      dep.address,
      9999999999,
      )).wait()
  // get pair address
  let pairAddr = await factory.getPair(note.address, stablecoin.address, true)
  pair = await ethers.getContractAt("BaseV1Pair", pairAddr)

  //set period size to zero for instant observations
  await (await factory.setPeriodSize(0)).wait()

  // swap 10 times for price observations
  for(var i = 0; i < 10; i++) {
      if (i % 2) {
          //swap 0.01 note for stable
          await (await router.swapExactTokensForTokensSimple(
              ethers.utils.parseUnits("10", "18"),
              0,
              note.address,
              stablecoin.address,
              true,
              dep.address,
              9999999999999
          )).wait()
      } else {
          //swap stable for note
          await (await router.swapExactTokensForTokensSimple(
              ethers.utils.parseUnits("10", "24"),
              0,
              stablecoin.address,
              note.address,
              true,
              dep.address,
              9999999999999
          )).wait()
      }
  }
  // check lpToken price
  // Actual price calculated is 999500000001499
  // But expected price (after removing brackets) is 1999999998838589
  console.log((await router.getPriceLP(pairAddr)).toString());
});
```

## Recommended Mitigation Steps

```diff
- uint token0TVL = assetReserves[i] * (prices[i] / decimals);
+ uint token0TVL = assetReserves[i] * prices[i] / decimals;
```