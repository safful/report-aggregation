## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-07-fractional-findings/issues/628) 

# QA Report

## Table of Contents

- [summary](#summary)

### Low
- [hash collision with abi.encodePacked](#hash-collision-with-abi.encodepacked)
- [Native `transfer` should be avoided](#native-transfer-should-be-avoided)
- [Return value of ERC20.transferFrom unchecked](#return-value-of-erc20.transferfrom-unchecked)
- [Setters and constructors should check the input value](#setters-and-constructors-should-check-the-input-value)
- [Unused `receive()` functions](#`unused-receive-functions`)

### Non-critical
- [Constants instead of magic numbers](#constants-instead-of-magic-numbers)
- [Events indexing](#events-indexing)
- [Event should be emitted in setters](#event-should-be-emitted-in-setters)
- [Public functions can be external](#public-functions-can-be-external)
- [Redundant cast](#redundant-cast)
- [Signature malleability](#signature-malleability)
- [TODOs](#todos)
- [Visibility should be explicit](#visibility-should-be-explicit)





# summary

> Few vulnerabilities were found examining the contracts. The main concerns are with:

# Low issues

# hash collision with abi.encodePacked

## IMPACT

strings and bytes are encoded with padding when using `abi.encodePacked`. This can lead to [hash collision](https://docs.soliditylang.org/en/v0.8.15/abi-spec.html#non-standard-packed-mode) when passing the result to `keccak256`

## SEVERITY

Low

## PROOF OF CONCEPT

Instances include:

### src/FERC1155.sol


```cpp
394:             keccak256(
395:                 abi.encodePacked("\x19\x01", _domainSeparator, _structHash)
396:             );
```


## TOOLS USED

Manual Analysis

## MITIGATION

Use `abi.encode()` instead.


# Native `transfer` should be avoided

## IMPACT


In `Migration`, the `.transfer()` method is used to transfer ETH. 

The `transfer()` call requires that the recipient has a payable callback, only provides 2300 gas for its operation. This means the following cases can cause the transfer to fail:

- The contract does not have a payable callback
- The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)
- The contract is called through a proxy which itself uses up the 2300 gas

## SEVERITY

Low

## PROOF OF CONCEPT

Instances include:

### src/modules/Migration.sol

```cpp
172:         payable(msg.sender).transfer(ethAmount);
325:         payable(msg.sender).transfer(ethAmount);
```


## TOOLS USED

Manual Analysis

## MITIGATION

Use `.call()` to send ETH instead.

# Return value of ERC20.transferFrom unchecked

## IMPACT


Some ERC20 implementations do not revert upon a fail `transfer/transferFrom` call, but return `false` instead. Not checking the return values of these calls can hence lead to silent failures of tokens transfers.

## SEVERITY

Low

## PROOF OF CONCEPT

Instances include:

### src/modules/protoforms/BaseVault.sol

```cpp
65:             IERC20(_tokens[i]).transferFrom(_from, _to, _amounts[i]);
```


## TOOLS USED

Manual Analysis

## MITIGATION

Check the return value of these calls to ensure they are not `0`

# Setters and constructors should check the input value

## PROBLEM

Setters and constructors should check the input value for addresses - ie revert if `address(0)` is assigned to `address` variables.



## SEVERITY

Low




## PROOF OF CONCEPT

Instances include:


### src/modules/protoforms/BaseVault.sol

```cpp
24:     constructor(address _registry, address _supply) Minter(_supply) {
25:         registry = _registry;
26:     }
```

### src/modules/Buyout.sol

```cpp
42:     constructor(
43:         address _registry,
44:         address _supply,
45:         address _transfer
46:     ) {
47:         registry = _registry;
48:         supply = _supply;
49:         transfer = _transfer;
50:     }
```

### src/modules/Migration.sol

```cpp
58:         buyout = payable(_buyout);
59:         registry = _registry;
```

### src/modules/Minter.sol

```cpp
17:     constructor(address _supply) {
18:         supply = _supply;
19:     }
```

### src/references/SupplyReference.sol

```cpp
15:     constructor(address _registry) {
16:         registry = _registry;
17:     }
```

### src/targets/Supply.sol

```cpp
16:     constructor(address _registry) {
17:         registry = _registry;
18:     }
```

## TOOLS USED

Manual Analysis



## MITIGATION

Add non-zero checks


# Unused `receive()` functions

## IMPACT


`Vault` and `Buyout` have an empty `receive()` function, but do not have any withdrawal function. Any ETH mistakenly sent to these contracts with empty `msg.data` would be locked.



## SEVERITY

Low



## PROOF OF CONCEPT

2 instances include:


### src/Vault.sol

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/Vault.sol#L32
```cpp
32:     receive() external payable {}
```

### src/modules/Buyout.sol#L53

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Buyout.sol#L53
```cpp
53:     receive() external payable {}
```


## TOOLS USED

Manual Analysis



## MITIGATION

Removes these functions or implement the appropriate logic in these empty blocks

# Non-critical issues

# Constants instead of magic numbers

## PROBLEM

It is best practice to use constant variables rather than literal values (100, 1000, etc) to make the code easier to understand and maintain.

## SEVERITY

Non-Critical

## PROOF OF CONCEPT

7 instances include:

### src/FERC1155.sol

```cpp
247:         royaltyAmount = (_salePrice * royaltyPercent[_id]) / 100;
```

### src/modules/Buyout.sol

```cpp
86:         uint256 buyoutPrice = (msg.value * 100) /
87:             (100 - ((depositAmount * 100) / totalSupply));
```

```cpp
208:         if (
209:             (tokenBalance * 1000) /
210:                 IVaultRegistry(registry).totalSupply(_vault) >
211:             500
212:         )
```

### src/modules/Migration.sol

```cpp
198:         uint256 currentPrice = _calculateTotal(
199:             100,
200:             IVaultRegistry(registry).totalSupply(_vault),
201:             proposal.totalEth,
202:             proposal.totalFractions
203:         )
```

## TOOLS USED

Manual Analysis

## MITIGATION

Define constant variables for the literal values aforementioned.


# Events indexing

## PROBLEM

Events should use the maximum amount of indexed fields: up to three parameters. This makes it easier to filter for specific values in front-ends.

## SEVERITY

Non-Critical

## PROOF OF CONCEPT

Instances include:


### src/interfaces/IBuyout.sol

```cpp
55: event Start(
56:         address indexed _vault,
57:         address indexed _proposer,
58:         uint256 _startTime,
59:         uint256 _buyoutPrice,
60:         uint256 _fractionPrice
61:     );

65:     event SellFractions(address indexed _seller, uint256 _amount);

69:     event BuyFractions(address indexed _buyer, uint256 _amount);

74:     event End(address _vault, State _state, address indexed _proposer);

79:     event Cash(address _vault, address indexed _casher, uint256 _amount);

83:     event Redeem(address _vault, address indexed _redeemer);
```

### src/interfaces/IFERC1155.sol

```cpp
21:     event SetMetadata(address indexed _metadata, uint256 _id);
```

```cpp
26:     event SetRoyalty(
27:         address indexed _receiver,
28:         uint256 _id,
29:         uint256 _percentage
30:     );
```

```cpp
36:     event SingleApproval(
37:         address indexed _owner,
38:         address indexed _operator,
39:         uint256 _id,
40:         bool _approved
41:     );
```

```cpp
61:     event FractionsMigrated(
62:         address indexed _oldVault,
63:         address indexed _newVault,
64:         uint256 _proposalId,
65:         uint256 _amount
66:     );
```

```cpp
74:     event VaultMigrated(
75:         address indexed _oldVault,
76:         address indexed _newVault,
77:         uint256 _proposalId,
78:         address[] _modules,
79:         address[] _plugins,
80:         bytes4[] _selectors
81:     );
```

### src/interfaces/IVault.sol

```cpp
25:     event Execute(address indexed _target, bytes _data, bytes _response);
```

```cpp
33:     event TransferOwnership(
34:         address indexed _oldOwner,
35:         address indexed _newOwner
36:     );
```

### src/interfaces/IVaultRegistry.sol

```cpp
33:     event VaultDeployed(
34:         address indexed _vault,
35:         address indexed _token,
36:         uint256 _id
37:     );
```

## TOOLS USED

Manual Analysis

## MITIGATION

Add indexed fields to these events so that they have the maximum number of indexed fields possible.


# Event should be emitted in setters

## PROBLEM

Setters should emit an event so that Dapps can detect important changes to storage


## SEVERITY

Non-Critical




## PROOF OF CONCEPT

Instances include:


### src/FERC1155.sol

```cpp
198:     function setContractURI(string calldata _uri) external onlyController
```

### src/Vault.sol

```cpp
86: function setMerkleRoot(bytes32 _rootHash) external 
```

## TOOLS USED

Manual Analysis



## MITIGATION

Emit an event in all setters.


# Public functions can be external

## PROBLEM

It is good practice to mark functions as `external` instead of `public` if they are not called by the contract where they are defined.

## SEVERITY

Non-Critical

## PROOF OF CONCEPT

Instances include:

### src/utils/MerkleBase.sol

```cpp
43:     function verifyProof(
44:         bytes32 _root,
45:         bytes32[] memory _proof,
46:         bytes32 _valueToProve
47:     ) public pure returns (bool)
```
```cpp
61:     function getRoot(bytes32[] memory _data) public pure returns (bytes32)
```
```cpp
73:     function getProof(bytes32[] memory _data, uint256 _node)
74:         public
75:         pure
76:         returns (bytes32[] memory)
```

## TOOLS USED

Manual Analysis

## MITIGATION

Declare these functions as `external` instead of `public`


# Redundant cast

## PROBLEM

In `Migration.commit()`, `buyout` is cast to type `address`, which is redundant as it is already of type `address`.

### src/modules/Migration.sol

```cpp
208:             IFERC1155(token).setApprovalFor(address(buyout), id, true);
```



## SEVERITY

Non-Critical


## TOOLS USED

Manual Analysis



## MITIGATION

```diff
-208:             IFERC1155(token).setApprovalFor(address(buyout), id, true);
+208:             IFERC1155(token).setApprovalFor(buyout, id, true);
```


# Scientific notation

## PROBLEM

For readability, it is best to use scientific notation (e.g `10e5`) rather than decimal literals(`100000`) or exponentiation(`10**5`)

## SEVERITY

Non-Critical

## PROOF OF CONCEPT

Instances include:

### src/modules/Buyout.sol

```cpp
208:         if (
209:             (tokenBalance * 1000) /
```

## TOOLS USED

Manual Analysis

## MITIGATION

Replace `1000` with `10e3`

# Signature malleability

## PROBLEM

`permit` and `permitAll` in `FERC1155` use Solidity's `ecrecover` to verify signatures. The EVM opcode associated with this function allows for malleable signatures and thus is susceptible to replay attacks. There is no direct threat to the protocol - these functions only approve operators - but it is still a good practice to avoid signature malleability.



## SEVERITY

Non-Critical




## PROOF OF CONCEPT

2 instances:


### src/FERC1155.sol

```cpp
126:             address signer = ecrecover(digest, _v, _r, _s);
```

```cpp
171:             address signer = ecrecover(digest, _v, _r, _s);
```



## TOOLS USED

Manual Analysis



## MITIGATION

Use OpenZeppelin's `ECDSA`'s [library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol)



# TODOS

## PROBLEM

There is an open TODO in `MerkleBase.sol`. It is merely a gas optimisation issue, but it should still be resolved before contract deployments



## SEVERITY

Non-Critical




## PROOF OF CONCEPT

Instances include:


### src/utils/MerkleBase.sol

```cpp
24:             // TODO: This can be aesthetically simplified with a switch. Not sure it will
25:             // save much gas but there are other optimizations to be had in here.
```



## TOOLS USED

Manual Analysis



## MITIGATION

Remove the TODO comment



# Visibility should be explicit

## PROBLEM

Visibility of variables should be explicitly set.

## SEVERITY

Non-Critical




## PROOF OF CONCEPT

2 instances:


### src/references/SupplyReference.sol

```cpp
12:     address immutable registry;
```

### src/targets/Supply.sol

```cpp
13:     address immutable registry;
```



## TOOLS USED

Manual Analysis

