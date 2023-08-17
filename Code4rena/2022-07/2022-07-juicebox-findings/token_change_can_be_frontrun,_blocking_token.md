## Tags

- bug
- documentation
- 3 (High Risk)
- sponsor confirmed
- old-submission-method
- valid

# [Token Change Can Be Frontrun, Blocking Token](https://github.com/code-423n4/2022-07-juicebox-findings/issues/104) 

# Lines of code

https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBTokenStore.sol#L246
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBTokenStore.sol#L266
https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/828bf2f3e719873daa08081cfa0d0a6deaa5ace5/contracts/JBController.sol#L605


# Vulnerability details

## Impact
This vulnerability allows malicious actors to block other users from changing tokens of their projects. Furthermore if ownership over the token contract is transferred to the `JBTokenStore` contract prior to the change, as suggested in the [recourse section of Juicebox's 24.05.2022 post-mortem update](https://github.com/jbx-protocol/juice-contracts-v2-code4rena/blob/main/security/postmortem/5.24.2022.md#Recourse), this vulnerability would allow an attacker to become the owner of tokens being transferred. For `JBToken` based tokens this would allow an attacker to begin issuing arbitrary amounts the token that was meant to be transferred.

## Proof of Concept
**Exploit scenario:**
1. Wanting to assign their token to their JB project an unsuspecting owner / admin transfers ownership to a `JBTokenStore` contract, either directly by calling `transferOwnership` on the token or indirectly by calling the `changeFor` method on an older `JBTokenStore` contract with `_newOwner` set as the new `JBTokenStore` contract. (For the newer Juicebox contracts the `JBController` contract's `changeTokenOf` method would be called) 
2. Seeing this change an attacker submits a `changeTokenFor` calling transaction to the new `JBController` contract, triggering the `JBTokenStore` contract's `changeFor` method, linking it to one of the attacker's projects (this could be created in advance or as part of the same transaction via an attack contract)
3. The attacker can then gain ownership over the token by calling `changeTokenFor` again with the `_newOwner` set to the attacker's address
4. Assuming the token has an owner restricted `mint` method like `JBToken` based tokens the attacker can now mint an arbitrary amount of the token

## Tools Used
Manual review.

## Recommended Mitigation Steps
Before allowing a caller to change to a specific token ensure that they have control over it. This can be achieved by storing a list of trusted older JB directories and projects which are then queried. Alternatively the contract could require the caller to actually be the `.owner()`  address of the token to migrate, this would require admins to:
1. Call `changeTokenOf` with themselves as the new owner
2. Call the new change token method on the newer contract, since they are the owner they'd pass the check
3. Independently transfer the ownership to the new token store to ensure that it can issue tokens

Future migrations can be made more seamless by having older contracts directly call new contracts via a sub-call, removing a necessary transaction for the admin. The newer contracts needs to verify that the older contract is the owner address of the token that's being set and also has approval of the project owner which is being configured.


