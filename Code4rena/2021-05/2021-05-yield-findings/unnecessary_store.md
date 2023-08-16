## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [unnecessary store](https://github.com/code-423n4/2021-05-yield-findings/issues/56) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function batch of Ladle.sol, at the operation GIVE, the value of vault is stored and is deleted directly afterwards.
So storing is unnecessary.
Maybe the solidity compiler already optimizes this.

## Proof of Concept
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/Ladle.sol#L228

 function batch(
} else if (operation == Operation.GIVE) {
               ... 
                vault = _give(vaultId, to);
                delete vault;   // Clear the cache, since the vault doesn't necessarily belong to msg.sender anymore
            
## Tools Used

## Recommended Mitigation Steps
Remove the " vault = "


