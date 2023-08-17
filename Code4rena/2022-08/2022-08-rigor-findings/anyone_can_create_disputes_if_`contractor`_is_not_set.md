## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Anyone can create disputes if `contractor` is not set](https://github.com/code-423n4/2022-08-rigor-findings/issues/327) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L498-L502
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/libraries/SignatureDecoder.sol#L25


# Vulnerability details

## Impact

Disputes enable an actor to arbitrate & potentially enforce requested state changes. However, the current implementation does not properly implement authorization, thus anyone is able to create disputes and spam the system with invalid disputes.

## Proof of Concept

Calling the `Project.raiseDispute` function with an invalid `_signature`, for instance providing a `_signature` with a length of 66 will return `address(0)` as the recovered signer address.

[Project.raiseDispute](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L498-L502)

```solidity
function raiseDispute(bytes calldata _data, bytes calldata _signature)
    external
    override
{
    // Recover the signer from the signature
    address signer = SignatureDecoder.recoverKey(
        keccak256(_data),
        _signature,
        0
    );

    ...
  }
```

[SignatureDecoder.sol#L25](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/libraries/SignatureDecoder.sol#L25)

```solidity
function recoverKey(
  bytes32 messageHash,
  bytes memory messageSignatures,
  uint256 pos
) internal pure returns (address) {
  if (messageSignatures.length % 65 != 0) {
      return (address(0));
  }

  ...
}
```

If `_task` is set to `0` and the project does not have a `contractor`, the `require` checks will pass and `IDisputes(disputes).raiseDispute(_data, _signature);` is called. The same applies if a specific `_task` is given and if the task has a `subcontractor`. Then the check will also pass.

[Project.raiseDispute](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Disputes.sol#L84-L122)

```solidity
function raiseDispute(bytes calldata _data, bytes calldata _signature)
    external
    override
{
    // Recover the signer from the signature
    address signer = SignatureDecoder.recoverKey(
        keccak256(_data),
        _signature,
        0
    );

    // Decode params from _data
    (address _project, uint256 _task, , , ) = abi.decode(
        _data,
        (address, uint256, uint8, bytes, bytes)
    );

    // Revert if decoded project address does not match this contract. Indicating incorrect _data.
    require(_project == address(this), "Project::!projectAddress");

    if (_task == 0) {
        // Revet if sender is not builder or contractor
        require(
            signer == builder || signer == contractor, // @audit-info if `contractor = address(0)` and the recovered signer is also the zero-address, this check will pass
            "Project::!(GC||Builder)"
        );
    } else {
        // Revet if sender is not builder, contractor or task's subcontractor
        require(
            signer == builder ||
                signer == contractor || // @audit-info if `contractor = address(0)` and the recovered signer is also the zero-address, this check will pass
                signer == tasks[_task].subcontractor,
            "Project::!(GC||Builder||SC)"
        );

        if (signer == tasks[_task].subcontractor) {
            // If sender is task's subcontractor, revert if invitation is not accepted.
            require(getAlerts(_task)[2], "Project::!SCConfirmed");
        }
    }

    // Make a call to Disputes contract raiseDisputes.
    IDisputes(disputes).raiseDispute(_data, _signature); // @audit-info Dispute will be created. Anyone can spam the system with fake disputes
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Consider checking the recovered `signer` address in `Project.raiseDispute` to not equal the zero-address:

```solidity
function raiseDispute(bytes calldata _data, bytes calldata _signature)
    external
    override
{
    // Recover the signer from the signature
    address signer = SignatureDecoder.recoverKey(
        keccak256(_data),
        _signature,
        0
    );

    require(signer != address(0), "Zero-address"); // @audit-info Revert if signer is zero-address

    ...
  }
```
