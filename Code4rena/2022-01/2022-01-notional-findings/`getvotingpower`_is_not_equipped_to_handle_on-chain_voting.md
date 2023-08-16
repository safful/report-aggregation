## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`getVotingPower` Is Not Equipped To Handle On-Chain Voting](https://github.com/code-423n4/2022-01-notional-findings/issues/165) 

# Handle

leastwood


# Vulnerability details

## Impact

As `NOTE` continues to be staked in the `sNOTE` contract, it is important that Notional's governance is able to correctly handle on-chain voting by calculating the relative power `sNOTE` has in terms of its equivalent `NOTE` amount. 

`getVotingPower` is a useful function in tracking the relative voting power a staker has, however, it does not utilise any checkpointing mechanism to ensure the user's voting power is a snapshot of a specific block number. As a result, it would be possible to manipulate a user's voting power by casting a vote on-chain and then have them transfer their `sNOTE` to another account to then vote again.

## Proof of Concept

https://github.com/code-423n4/2022-01-notional/blob/main/contracts/sNOTE.sol#L271-L293
```
function getVotingPower(uint256 sNOTEAmount) public view returns (uint256) {
    // Gets the BPT token price (in ETH)
    uint256 bptPrice = IPriceOracle(address(BALANCER_POOL_TOKEN)).getLatest(IPriceOracle.Variable.BPT_PRICE);
    // Gets the NOTE token price (in ETH)
    uint256 notePrice = IPriceOracle(address(BALANCER_POOL_TOKEN)).getLatest(IPriceOracle.Variable.PAIR_PRICE);
    
    // Since both bptPrice and notePrice are denominated in ETH, we can use
    // this formula to calculate noteAmount
    // bptBalance * bptPrice = notePrice * noteAmount
    // noteAmount = bptPrice/notePrice * bptBalance
    uint256 priceRatio = bptPrice * 1e18 / notePrice;
    uint256 bptBalance = BALANCER_POOL_TOKEN.balanceOf(address(this));

    // Amount_note = Price_NOTE_per_BPT * BPT_supply * 80% (80/20 pool)
    uint256 noteAmount = priceRatio * bptBalance * 80 / 100;

    // Reduce precision down to 1e8 (NOTE token)
    // priceRatio and bptBalance are both 1e18 (1e36 total)
    // we divide by 1e28 to get to 1e8
    noteAmount /= 1e28;

    return (noteAmount * sNOTEAmount) / totalSupply();
}
```

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider implementing a `getPriorVotingPower` function which takes in a `blockNumber` argument and returns the correct balance at that specific block.

