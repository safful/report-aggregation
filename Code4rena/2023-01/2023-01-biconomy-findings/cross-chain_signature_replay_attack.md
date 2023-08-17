## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- selected for report
- sponsor confirmed
- M-03

# [Cross-Chain Signature Replay Attack](https://github.com/code-423n4/2023-01-biconomy-findings/issues/466) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/paymasters/verifying/singleton/VerifyingSingletonPaymaster.sol#L77-L90


# Vulnerability details

# Cross-Chain Signature Replay Attack

## Impact
User operations can be replayed on smart accounts accross different chains. This can lead to user's loosing funds or any unexpected behaviour that transaction replay attacks usually lead to.

## Proof of Concept
As specified by the [EIP4337](https://eips.ethereum.org/EIPS/eip-4337) standard `to prevent replay attacks ... the signature should depend on chainid`. In [VerifyingSingletonPaymaster.sol#getHash](https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/paymasters/verifying/singleton/VerifyingSingletonPaymaster.sol#L77-L90) the chainId is missing which means that the same UserOperation can be replayed on a different chain for the same smart contract account if the `verifyingSigner` is the same (and most likely this will be the case).

## Tools Used
Manual review

## Recommended Mitigation Steps
Add the chainId in the calculation of the UserOperation hash in [VerifyingSingletonPaymaster.sol#getHash](https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/paymasters/verifying/singleton/VerifyingSingletonPaymaster.sol#L77-L90)

```
    function getHash(UserOperation calldata userOp)
    public view returns (bytes32) { // @audit change to view
        //can't use userOp.hash(), since it contains also the paymasterAndData itself.
        return keccak256(abi.encode(
                userOp.getSender(),
                userOp.nonce,
                keccak256(userOp.initCode),
                keccak256(userOp.callData),
                userOp.callGasLimit,
                userOp.verificationGasLimit,
                userOp.preVerificationGas,
                userOp.maxFeePerGas,
                userOp.maxPriorityFeePerGas
		block.chainid // @audit add chain id
            ));
    }
```