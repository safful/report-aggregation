## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [ LiquidityFarming variable allows for hash collisions](https://github.com/code-423n4/2022-03-biconomy-findings/issues/131) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityFarming.sol#L59
https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityFarming.sol#L196:L224
https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityFarming.sol#L229:L253


# Vulnerability details

## Impact
The `nftIdsStaked` variable introduces a hash collision vulnerability into the `LiquidityFarming.sol` contract as it is employing a mapping from address to a variable length of data field.
Source: `https://github.com/code-423n4/2022-03-biconomy/blob/04751283f85c9fc94fb644ff2b489ec339cd9ffc/contracts/hyphen/LiquidityFarming.sol#L59`

The easiest attack scenario allows loss of funds for victims as the attackers can stop victims from unstaking their NFT in the `withdraw` function.
Two more complicated attack scenarios allow loss of funds as attackers can outright withdraw nfts of victims.


## Proof of Concept
This attack depends very on the address of the victim an attacker would want to attack.
Therefore let me just illustrate the problem.

When using an address mapping to some fixed length data field one can rely on keccak to prevent collisions. 
When using an address mapping with an attacker controlled length data field an attacker has a somewhat easy path to create collisions and thereby write to or read from victims data.

The first part to understanding the attack is understanding that all the nfts of a victim are going to be in continous storage locations of the contract.
Lets assume that a victims mapped to array starts at 0x1337 and the own 2 nft with the Ids: 23 and 38. The memory layout could look like this:

```
addr  | val | description
------|-----|----
0x1337| 2   | length of the array
0x1338| 23   | Id of the first owned nft
0x1339| 38   | Id of the second owned nft
```

Please note if the victim were to deposit more nfts those nftIds would be placed in the storage address 0x133a, 0x133b and so on.

### Less complex attack scenario
The easier attack scenario would involve the attacker generating addresses that result in a continous storage region below address 0x1337.
Suppose the attacker were to generate an address whose arrays storage of nftIdsStaked lands at 0x1330. The memory layout would look like this.

```
addr  | val | description
------|-----|----
0x1330| 0   | length of the array
0x1337| 2   | length of the array
0x1338| 23   | Id of the first owned nft
0x1339| 38   | Id of the second owned nft
```

It is easy to see, that they could grow their array and thereby corrupting the array of the victim. 
They could essentially write nftIds of worthless nftId into the storage of the victim, thereby overwriting their nftId and preventing them (or anybody else) from withdrawing them.

### More complex attack scenario

The more complex attack scenario involves finding two addresses that will have somewhat adjacent storage regions. 
Where the lower storage region encroaches on the length field of the array. The length field is attacker controlled, thereby allowing the attacker to call `withdraw` on arbitrary nftId.

```
addr  | val | description
------|-----|----
0x1337| 4   | length of lower array
0x1338| 37   | Id of the first owned (worthless) nft
0x1339| 39   | Id of the second owned (worthless) nft
0x133a| 40   | Id of the third owned (worthless) nft
0x133b| N    | length of higher array (attacker controlled) AND 4th nftId of lower array
0x133a| 42   | Id of the first owned (worthless) nft
```


Please note that this "feels" like a traditional bruteforce attack on keccak (because it is) but it is orders of magnitude more likely to be succesful. 
Attackers can essentially trade bruteforcing addresses with calling `deposit` a bunch of times.

As this attack scenario is somewhat less known and more similar to traditional memory corruption attacks allow me to leave a link describing the issue in more detail:
`https://xlab.tencent.com/en/2018/11/09/pay-attention-to-the-ethereum-hash-collision-problem-from-the-stealing-coins-incident/`


## Tools Used
Manual audit

## Recommended Mitigation Steps
Track nftIdsStaked like so:

```
// user address =>        nth nft =>  nft id
mapping(address => mapping(uint256 => uint256)) public nftIdsStaked;
```

