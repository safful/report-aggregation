## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- valid

# [Untyped data signing](https://github.com/code-423n4/2022-08-rigor-findings/issues/75) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/e35f5f61be9ff4b8dc5153e313419ac42964d1fd/contracts/Community.sol#L175
https://github.com/code-423n4/2022-08-rigor/blob/e35f5f61be9ff4b8dc5153e313419ac42964d1fd/contracts/Community.sol#L213
https://github.com/code-423n4/2022-08-rigor/blob/e35f5f61be9ff4b8dc5153e313419ac42964d1fd/contracts/Community.sol#L530
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Disputes.sol#L91
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L142
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L167
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L235
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L286
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L346
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L402
https://github.com/code-423n4/2022-08-rigor/blob/f2498c86dbd0e265f82ec76d9ec576442e896a87/contracts/Project.sol#L499


# Vulnerability details

## Impact & Proof Of Concepts
In many places of the project (see affected code), untyped application data is directly hashed and signed. This is strongly disencouraged, as it enables different attacks (that each could be considered their own issue / vulnerability, but I submitted it as one, as they have all the same root cause):

1.) Signature reuse across different Rigor projects:
While some signature contain the project address, not all do. For instance, `updateProjectHash` only contains a `_hash` and a `_nonce`. Therefore, we can have the following scenario: Bob is the owner of project A and signs / submit `updateProjectHash` with nonce 0 and some hash. Then, a project B that also has Bob as the owner is created. Attacker Charlie can simply take the `_data` and `_signature` that Bob previously submitted to project A and send it to project B. As this project will have a nonce of 0 (fresh created), it will accept it. `updateTaskHash` is also affected by this.
2.) Signature reuse across different chains:
Because the chain ID is not included in the data, all signatures are also valid when the project is launched on a chain with another chain ID. For instance, let's say it is also launched on Polygon. An attacker can now use all of the Ethereum signatures there. Because the Polygon addresses of user's (and potentially contracts, when the nonces for creating are the same) are often identical, there can be situations where the payload is meaningful on both chains.
3.) Signature reuse across Rigor functions:
Some functions accept and decode data / signatures that were intended for other functions. For instance, see this example of providing the data & signature that was intended for `inviteContractor` to `setComplete`:
```diff
diff --git a/test/utils/projectTests.ts b/test/utils/projectTests.ts
index ae9e202..752e01f 100644
--- a/test/utils/projectTests.ts
+++ b/test/utils/projectTests.ts
@@ -441,7 +441,7 @@ export const projectTests = async ({
     }
   });

-  it('should be able to invite contractor', async () => {
+  it.only('should be able to invite contractor', async () => {
     expect(await project.contractor()).to.equal(ethers.constants.AddressZero);
     const data = {
       types: ['address', 'address'],
@@ -452,6 +452,7 @@ export const projectTests = async ({
       signers[1],
     ]);
     const tx = await project.inviteContractor(encodedData, signature);
+    const tx2 = await project.setComplete(encodedData, signature);
     await expect(tx)
       .to.emit(project, 'ContractorInvited')
       .withArgs(signers[1].address);
```
While this reverts because there is no task that corresponds to the address that is signed there, this is not always the case.
4.) Signature reuse from different Ethereum projects & phishing
Because the payload of these signatures is very generic (two addresses, a byte and two uints), there might be situations where a user has already signed data with the same format for a completely different Ethereum application. Furthermore, an attacker could set up a DApp that uses the same format and trick someone into signing the data. Even a very security-conscious owner that has audited the contract of this DApp (that does not have any vulnerabilities and is not malicious, it simply consumes signatures that happen to have the same format) might be willing to sign data for this DApp, as he does not anticipate that this puts his Rigor project in danger.

## Recommended Mitigation Steps
I strongly recommend to follow [EIP-712](https://eips.ethereum.org/EIPS/eip-712) and not implement your own standard / solution. While this also improves the user experience, this topic is very complex and not easy to get right, so it is recommended to use a battle-tested approach that people have thought in detail about. All of the mentioned attacks are not possible with EIP-712:
1.) There is always a domain separator that includes the contract address.
2.) The chain ID is included in the domain separator
3.) There is a type hash (of the function name / parameters)
4.) The domain separator does not allow reuse across different projects, phishing with an innocent DApp is no longer possible (it would be shown to the user that he is signing data for Rigor, which he would off course not do on a different site)