## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- selected for report
- responded

# [MEV: Operator can bribe miner and steal honest operator's bond amount if gas price went high](https://github.com/code-423n4/2022-10-holograph-findings/issues/473) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L354


# Vulnerability details

## Description
Operators in Holograph do their job by calling executeJob() with the bridged in bytes from source chain. 
If the primary job operator did not execute the job during his allocated block slot, he is punished by taking a single bond amount and transfer it to the operator doing it instead. 
The docs and code state that if there was a gas spike in the operator's slot, he shall not be punished. The way a gas spike is checked is with this code in executeJob:
```
require(gasPrice >= tx.gasprice, "HOLOGRAPH: gas spike detected");
```

However, there is still a way for operator to claim primary operator's bond amount although gas price is high. Attacker can submit a flashbots bundle including the executeJob() transaction, and one additional "bribe" transaction. The bribe transaction will transfer some incentive amount to coinbase address (miner), while the executeJob is submitted with a low gasprice. Miner will accept this bundle as it is overall rewarding enough for them, and attacker will receive the base bond amount from victim operator. This threat is not theoretical because every block we see MEV bots squeezing value from such opportunities.

info about coinbase [transfer](https://docs.flashbots.net/flashbots-auction/searchers/advanced/coinbase-payment)
info about bundle [selection](https://docs.flashbots.net/flashbots-auction/searchers/advanced/bundle-pricing#bundle-ordering-formula)

## Impact

Dishonest operator can take honest operator's bond amount although gas price is above acceptable limits.

## Tools Used

Manual audit, flashbot docs

## Recommended Mitigation Steps

Do not use current tx.gasprice amount to infer gas price in a previous block. 
Probably best to use gas price oracle.