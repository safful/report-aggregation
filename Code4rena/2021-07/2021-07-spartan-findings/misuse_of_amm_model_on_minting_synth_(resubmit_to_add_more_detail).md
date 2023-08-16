## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Misuse of AMM model on minting Synth (resubmit to add more detail)](https://github.com/code-423n4/2021-07-spartan-findings/issues/59) 

# Handle

jonah1005


# Vulnerability details

## Impact
`Pool` calculates the amount to be minted based on `token_amount` and `sparta_amount` of the Pool. However, since `token_amount` in the pool would not decrease when users mint Synth, it's always cheaper to mint synth than swap the tokens.

The synthetics would be really hard to be on peg. Or, there would be a flash-loan attacker to win all the arbitrage space.

## Proof of Concept
Pool's mint synth
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L229-L242

The synth amount is calculated at L:232
```
uint output = iUTILS(_DAO().UTILS()).calcSwapOutput(_actualInputBase, baseAmount, tokenAmount);
```
which is the same as swapping base to token at L:287
```
uint256 _X = baseAmount;
uint256 _Y = tokenAmount;
_y =  iUTILS(_DAO().UTILS()).calcSwapOutput(_x, _X, _Y); // Calc TOKEN output
```

However, while swapping tokens decrease pool's token, mint just mint it out of the air.

Here's a POC:
Swap sparta to token for ten times
```python
for i in range(10):
    amount = 10 * 10**18
    transfer_amount = int(amount/10)
    base.functions.transfer(token_pool.address, transfer_amount).transact()
    token_pool.functions.swapTo(token.address, user).transact()
```

Mint Synth for ten times
```python
for i in range(10):
    amount = 10 * 10**18
    transfer_amount = int(amount/10)
    base.functions.transfer(token_pool.address, transfer_amount).transact()
    token_pool.functions.mintSynth(token_synth.address, user).transact()
```
The Pool was initialized with 10000:10000 in both cases. While the first case(swap token) gets `4744.4059` and the second case gets `6223.758`.

## Tools Used
None
## Recommended Mitigation Steps
The debt should be considered in the AMM pool.

I recommend to maintain a debt variable in the Pool and use `tokenAmount - debt` when the Pool calculates the token price.
Here's some idea of it.
```
    uint256 public debt;
    function _tokenAmount() returns (uint256) {
        return tokenAmount - debt;
    }

    // Swap SPARTA for Synths
    function mintSynth(address synthOut, address member) external returns(uint outputAmount, uint fee) {
        require(iSYNTHFACTORY(_DAO().SYNTHFACTORY()).isSynth(synthOut) == true, "!synth"); // Must be a valid Synth
        uint256 _actualInputBase = _getAddedBaseAmount(); // Get received SPARTA amount

        // Use tokenAmount - debt to calculate the value
        uint output = iUTILS(_DAO().UTILS()).calcSwapOutput(_actualInputBase, baseAmount, _tokenAmount()); // Calculate value of swapping SPARTA to the relevant underlying TOKEN

        // increment the debt
        debt += output

        uint _liquidityUnits = iUTILS(_DAO().UTILS()).calcLiquidityUnitsAsym(_actualInputBase, address(this)); // Calculate LP tokens to be minted
        _incrementPoolBalances(_actualInputBase, 0); // Update recorded SPARTA amount
        uint _fee = iUTILS(_DAO().UTILS()).calcSwapFee(_actualInputBase, baseAmount, tokenAmount); // Calc slip fee in TOKEN
        fee = iUTILS(_DAO().UTILS()).calcSpotValueInBase(TOKEN, _fee); // Convert TOKEN fee to SPARTA
        _mint(synthOut, _liquidityUnits); // Mint the LP tokens directly to the Synth contract to hold
        iSYNTH(synthOut).mintSynth(member, output); // Mint the Synth tokens directly to the user
        _addPoolMetrics(fee); // Add slip fee to the revenue metrics
        emit MintSynth(member, BASE, _actualInputBase, TOKEN, outputAmount);
      return (output, fee);
    }

```


