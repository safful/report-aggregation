## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Basket will break and lock all user funds if not used in 100 years](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/63) 

# Handle

kenzo


# Vulnerability details

The ```handleFees``` function divides by ```(BASE - ((block.timestamp - lastFee)* licenseFee / ONE_YEAR))```.
For initial BASE of 1e18 and licenseFee of 1e16, it means that if nobody calls this function in 100 years, the function will divide by 0.

## Impact
After 100 years of no usage, handleFees will always revert and nobody will be able to mint, burn etc'.

## Proof of Concept
Vulnerable line which will divide by 0:
https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L118
To test this, you can deploy to testnet a contract, then use a time machine to travel to 100 years in the future and try to use mint().
If for some reason you don't want to use your time machine, you may use this function to simulate the passage of time:
```
async function skipTime(seconds) {
  let blockNumber = await hre.network.provider.request({
    method: "eth_blockNumber",
    params: [],
  });
  let block = await ethers.provider.getBlock(blockNumber["result"]);
  await hre.network.provider.request({
    method: "evm_mine",
    params: [block["timestamp"]+seconds],
  });
}
```

## Tools Used
Manual analysis, hardhat, time machine.

## Recommended Mitigation Steps
Tell your grandchildren to call mint(1) in 99 years.

