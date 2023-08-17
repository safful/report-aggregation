## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [It is possible to create fake ERC1155 NameWrapper token for subdomain, which is not owned by NameWrapper](https://github.com/code-423n4/2022-07-ens-findings/issues/84) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L820-L821
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L524
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L572


# Vulnerability details

## Impact

Due to re-entrancy possibility in `NameWrapper._transferAndBurnFuses` (called from `setSubnodeOwner` and `setSubnodeRecord`), it is possible to do some stuff in `onERC1155Received` right after transfer but before new owner and new fuses are set. This makes it possible, for example, to unwrap the subdomain, but owner and fuses will still be set even for unwrapped domain, creating fake `ERC1155` `NameWrapper` token for domain, which is not owned by `NameWrapper`.

Fake token creation scenario:

1. `Account1` registers and wraps `test.eth` domain
2. `Account1` calls `NameWrapper.setSubnodeOwner` for `sub.test.eth` subdomain with `Account1` as owner (to make NameWrapper owner of subdomain)
3. `Contract1` smart contract is created, which calls unwrap in its `onERC1155Received` function, and a function to send `sub.test.eth` ERC1155 NameWrapper token back to `Account1`
4. `Account1` calls `NameWrapper.setSubnodeOwner` for `sub.test.eth` with `Contract1` as new owner, which unwraps domain back to `Account1` but due to re-entrancy, NameWrapper sets fuses and ownership to `Contract1`
5. `Account1` calls function to send ERC1155 token from `Contract1` back to self.

After this sequence of events, `sub.test.eth` subdomain is owned by `Account1` both in `ENS` registry and in `NameWrapper` (with fuses and expiry correctly set to the future date). Lots (but not all) of functions in `NameWrapper` will fail to execute for this subdomain, because they expect `NameWrapper` to have ownership of the domain in `ENS`, but some functions will still work, making it possible to make the impression of good domain.

At this point, ownership in `NameWrapper` is "detached" from ownership in `ENS` and `Account1` can do all kinds of malcious stuff with its ERC1155 token. For example:

1. Sell subdomain to the other user, transfering `ERC1155` to that user and burning `PARENT_CANNOT_CONTROL` to create impression that he can't control the domain. After receiving the payment, `Account1` can wrap the domain again, which burns existing ownership record and replaces with the new one with clear fuses and `Account1` ownership, effectively stealing domain back from unsuspecting user, who thought that `ERC1155` gives him the right to the domain (and didn't expect that parent can clear fuses when `PARENT_CANNOT_CONTROL` is set).

2. Transfer subdomain to some other smart contract, which implements `onERC1155Received`, then take it back, fooling smart contract into believing that it has received the domain.


## Proof of Concept

Copy these to test/wrapper and run:
yarn test test/wrapper/NameWrapperReentrancy.js

https://gist.github.com/panprog/3cd94e3fbb0c52410a4c6609e55b863e


## Recommended Mitigation Steps

Consider adding `nonReentrant` modifiers with `ReentrancyGuard` implementation from `openzeppelin`. Alternatively just fix this individual re-entrancy issue. There are multiple ways to fix it depending on expected behaviour, for example saving `ERC1155` data and requiring it to match the data after transfer (restricting `onERC1155Received` to not change any data for the token received):

    function _transferAndBurnFuses(
        bytes32 node,
        address newOwner,
        uint32 fuses,
        uint64 expiry
    ) internal {
        (address owner, uint32 saveFuses, uint64 saveExpiry) = getData(uint256(node));
        _transfer(owner, newOwner, uint256(node), 1, "");
        uint32 curFuses;
        uint64 curExpiry;
        (owner, curFuses, curExpiry) = getData(uint256(node));
        require(owner == newOwner && saveFuses == curFuses && saveExpiry == curExpiry);
        _setFuses(node, newOwner, fuses, expiry);
    }


