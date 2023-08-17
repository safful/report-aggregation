## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-05

# [Paymaster ETH can be drained with malicious sender](https://github.com/code-423n4/2023-01-biconomy-findings/issues/151) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/paymasters/verifying/singleton/VerifyingSingletonPaymaster.sol#L97-L111


# Vulnerability details

## Impact
Paymaster's signature can be replayed to drain their deposits

## Proof of Concept

Scenario : 
- user A is happy with biconomy and behaves well biconomy gives some sponsored tx using verifyingPaymaster -- let's say paymaster's signature as sig X
- user A becomes not happy with biconomy for some reason and A wants to attack biconomy
- user A delegate calls to Upgrader and upgrade it's sender contract to MaliciousAccount.sol
- MaliciousAccount.sol does not check any nonce and everything else is same to SmartAccount(but they can also add some other details to amplify the attack, but let's just stick it this way)
- user A uses sig X(the one that used before) to initiate the same tx over and over
- user A earnes nearly nothing but paymaster will get their deposits drained


files : Upgrader.sol, MaliciousAccount.sol, test file
https://gist.github.com/leekt/d8fb59f448e10aeceafbd2306aceaab2


## Tools Used
hardhat test, verified with livingrock

## Recommended Mitigation Steps
Since `validatePaymasterUserOp` function is not limited to view function in erc4337 spec, add simple boolean data for mapping if hash is used or not 

```
mapping(bytes32 => boolean) public usedHash

    function validatePaymasterUserOp(UserOperation calldata userOp, bytes32 /*userOpHash*/, uint256 requiredPreFund)
    external override returns (bytes memory context, uint256 deadline) {
        (requiredPreFund);
        bytes32 hash = getHash(userOp);
        require(!usedHash[hash], "used hash");
        usedHash[hash] = true;
```