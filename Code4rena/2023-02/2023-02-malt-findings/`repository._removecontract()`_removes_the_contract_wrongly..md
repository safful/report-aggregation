## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-09

# [`Repository._removeContract()` removes the contract wrongly.](https://github.com/code-423n4/2023-02-malt-findings/issues/25) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/Repository.sol#L229


# Vulnerability details

## Impact
After removing the contract, the `contracts` array would contain the wrong contract names.

## Proof of Concept
`Repository._removeContract()` removes the contract name from `contracts` array.

```solidity
File: 2023-02-malt\contracts\Repository.sol
223:   function _removeContract(string memory _name) internal {
224:     bytes32 hashedName = keccak256(abi.encodePacked(_name));
225:     Contract storage currentContract = globalContracts[hashedName];
226:     currentContract.contractAddress = address(0);
227:     currentContract.index = 0;
228: 
229:     uint256 index = currentContract.index; //@audit wrong index
230:     string memory lastContract = contracts[contracts.length - 1];
231:     contracts[index] = lastContract;
232:     contracts.pop();
233:     emit RemoveContract(hashedName);
234:   }
```

But it uses the already changed index(= 0) and replaces the last name with 0 index all the time.

As a result, the contracts array will still contain the removed name and remove the valid name at index 0.

## Tools Used
Manual Review

## Recommended Mitigation Steps
We should use the original index like below.

```solidity
  function _removeContract(string memory _name) internal {
    bytes32 hashedName = keccak256(abi.encodePacked(_name));
    Contract storage currentContract = globalContracts[hashedName];

    uint256 index = currentContract.index; //++++++++++++++

    currentContract.contractAddress = address(0);
    currentContract.index = 0;

    string memory lastContract = contracts[contracts.length - 1];
    contracts[index] = lastContract;
    contracts.pop();
    emit RemoveContract(hashedName);
  }
```