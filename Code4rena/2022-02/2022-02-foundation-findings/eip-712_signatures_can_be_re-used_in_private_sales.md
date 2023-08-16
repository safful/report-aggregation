## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [EIP-712 signatures can be re-used in private sales](https://github.com/code-423n4/2022-02-foundation-findings/issues/68) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketPrivateSale.sol#L123-L174


# Vulnerability details

## Impact
Within a NFTMarketPrivateSale contract, buyers are allowed to purchase a seller's NFT. This is done through a seller providing a buyer a EIP-712 signature. The buyer can then call `#buyFromPrivateSaleFor` providing the v, r, and s values of the signature as well as any additional details to generate the message hash. If the signature is valid, then the NFT is transferred to the buyer.

 The problem with the code is that EIP-712 signatures can be re-used within a small range of time assuming that the original seller takes back ownership of the NFT. This is because the NFTMarketPrivateSale#buyFromPrivateSaleFor method has no checks to determine if the EIP-712 signature has been used before. 

## Proof of Concept

Consider the following example:

1. Joe the NFT owner sells a NFT to the malicious buyer Rachel via a private sale. 
2. Rachel through this private sale obtains the EIP-712 signature and uses it to purchase a NFT.
3. Joe the NFT owner purchases back the NFT within two days of the original sale to Rachel.
4. Joe the NFT owner puts the NFT back on sale.
5. Rachel, who has the original EIP-712 signature, can re-purchase the NFT by calling `#buyFromPrivateSaleFor` again with the same parameters they provided in the original private sale purchase in step 1.

The `#buyFromPrivateSaleFor` [function](https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketPrivateSale.sol#L123) runs several validation checks before transferring the NFT over to the buyer. The validations are as follows:

1. L#132 - The signature has expired.
2. L#135 - The deadline is beyond 48 hours from now.
3. L#143 - The amount argument is greater than msg.value.
4. L#149 - The msg.value is greater than the amount set.
5. L#171 - This checks that the EIP-712 signature comes from the NFT seller.

As you can see, there are no checks that the EIP-712 signature has been used before. If the original NFT seller purchases back the NFT, then they are susceptible to having the original buyer taking back the NFT. This can be problematic if the NFT has risen in value, as the original buyer can utilize the same purchase amount from the first transaction in this malicious transaction.

## Tools Used
Pen and paper

## Recommended Mitigation Steps
Most contracts utilize nonces when generating EIP-712 signatures to ensure that the contract hasn't been used for. When a nonce is injected into a signature, it makes it impossible for re-use, assuming of course the nonce feature is done correctly.

