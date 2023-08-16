## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Use openzeppelin ECDA for erecover](https://github.com/code-423n4/2021-05-88mph-findings/issues/20) 

# Handle

a_delamo


# Vulnerability details

## Impact

In `Sponsorable.sol` is using erecover directly to verify the signature.
Being such a critical piece of the protocol, I would recommend using the ECDSA from openzeppelin as it does more validations when verifying the signature. 

```
 // Currently

 address recoveredAddress =
      ecrecover(digest, sponsorship.v, sponsorship.r, sponsorship.s);
    require(
      recoveredAddress != address(0) && recoveredAddress == sponsorship.sender,
      "Sponsorable: invalid sig"
    );


  //ECDSA

   function recover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) internal pure returns (address) {
        // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
        // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
        // the valid range for s in (281): 0 < s < secp256k1n ÷ 2 + 1, and for v in (282): v ∈ {27, 28}. Most
        // signatures from current libraries generate a unique signature with an s-value in the lower half order.
        //
        // If your library generates malleable signatures, such as s-values in the upper range, calculate a new s-value
        // with 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1 and flip v from 27 to 28 or
        // vice versa. If your library also generates signatures with 0/1 for v instead 27/28, add 27 to v to accept
        // these malleable signatures as well.
        require(uint256(s) <= 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0, "ECDSA: invalid signature 's' value");
        require(v == 27 || v == 28, "ECDSA: invalid signature 'v' value");

        // If the signature is valid (and not malleable), return the signer address
        address signer = ecrecover(hash, v, r, s);
        require(signer != address(0), "ECDSA: invalid signature");

        return signer;
    }
  
```
## Tools Used

None


