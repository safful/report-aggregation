## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- old-submission-method
- valid

# [Add members to the not yet created community](https://github.com/code-423n4/2022-08-rigor-findings/issues/298) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/Community.sol#L187
https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/Community.sol#L179
https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/Community.sol#L878
https://github.com/code-423n4/2022-08-rigor/blob/main/contracts/libraries/SignatureDecoder.sol#L39


# Vulnerability details

## Impact
There is a `addMember` function in the `Community`.  The function accepts `_data` that should be signed by the `_community.owner` and `_newMemberAddr`. 

```
        // Compute hash from bytes
        bytes32 _hash = keccak256(_data);

        // Decode params from _data
        (
            uint256 _communityID,
            address _newMemberAddr,
            bytes memory _messageHash
        ) = abi.decode(_data, (uint256, address, bytes));

        CommunityStruct storage _community = _communities[_communityID];

        // check signatures
        checkSignatureValidity(_community.owner, _hash, _signature, 0); // must be community owner
        checkSignatureValidity(_newMemberAddr, _hash, _signature, 1); // must be new member
```

The code above shows exactly what the contract logic looks like. 

1) `_communityID` is taken from the data provided by user, so it can arbitrarily. Specifically,  community with selected `_communityID` can be not yet created. For instance, it can be equal to the `communityCount + 1`, thus the next created community will have this `_communityID`. 

2) `_communities[_communityID]` will store null values for all fields, for a selected `_communityID`. That means, `_community.owner == address(0)`

3) `checkSignatureValidity` with a parameters `address(0), _hash, _signature, 0` will not revert a call if an attacker provide incorrect `_signature`.

let's see the implementation of `checkSignatureValidity`:

```
        // Decode signer
        address _recoveredSignature = SignatureDecoder.recoverKey(
            _hash,
            _signature,
            _signatureIndex
        );

        // Revert if decoded signer does not match expected address
        // Or if hash is not approved by the expected address.
        require(
            _recoveredSignature == _address || approvedHashes[_address][_hash],
            "Community::invalid signature"
        );

        // Delete from approvedHash. So that signature cannot be reused.
        delete approvedHashes[_address][_hash];
```

No restrictions on `_recoveredSignature` or `_address`. Moreover, if `SignatureDecoder.recoverKey` can return zero value, then there will be no revert. 

```
       if (messageSignatures.length % 65 != 0) {
            return (address(0));
        }

        uint8 v;
        bytes32 r;
        bytes32 s;
        (v, r, s) = signatureSplit(messageSignatures, pos);

        // If the version is correct return the signer address
        if (v != 27 && v != 28) {
            return (address(0));
        } else {
            // solium-disable-next-line arg-overflow
            return ecrecover(toEthSignedMessageHash(messageHash), v, r, s);
        }
```

As we can see bellow, `recoverKey` function can return zero value, if an `ecrecover` return zero value or if `v != 27 || v != 28`. Both cases are completely dependent on the input parameters to the function, namely from `signature` that is provided by attacker.

4) `checkSignatureValidity(_newMemberAddr, _hash, _signature, 1)` will not revert the call if an attacker provide correct signature in the function. It is obviously possible.

All in all, an attacker can add as many members as they want, BEFORE the `community` will be created. 

## Tools Used

## Recommended Mitigation Steps


1) `checkSignatureValidity`/`recoverKey` should revert the call if an `address == 0`.
2) `addMember` should have a `require(_communityId <= communityCount)`

