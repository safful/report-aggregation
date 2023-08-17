## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed
- old-submission-method
- valid

# [New subcontractor can be set for a SCConfirmed task without current subcontractor consent](https://github.com/code-423n4/2022-08-rigor-findings/issues/378) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L295-L316


# Vulnerability details

Malicious builder/contractor can change the subcontractor for any task even if all the terms was agreed upon and work was started/finished, but the task wasn't set to completed yet, i.e. it's `SCConfirmed`, `getAlerts(_taskID)[2] == true`. This condition is not checked by inviteSC().

For example, a contractor can create a subcontractor of her own and front run valid setComplete() call with a sequence of `inviteSC(task, own_subcontractor) -> setComplete()` with a signatory from the `own_subcontractor`, stealing the task budget from the subcontractor who did the job. Contractor will not breach any duties with the community as the task will be done, while raiseDispute() will not work for a real subcontractor as the task record will be already changed.

Setting the severity to be high as this creates an attack vector to fully steal task budget from the subcontractor as at the moment of any valid setComplete() call the task budget belongs to subcontractor as the job completion is already verified by all the parties.

## Proof of Concept

inviteSC() requires either builder or contractor to call for the change and verify nothing else:

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L295-L316

```solidity
    /// @inheritdoc IProject
    function inviteSC(uint256[] calldata _taskList, address[] calldata _scList)
        external
        override
    {
        // Revert if sender is neither builder nor contractor.
        require(
            _msgSender() == builder || _msgSender() == contractor,
            "Project::!Builder||!GC"
        );

        // Revert if taskList array length not equal to scList array length.
        uint256 _length = _taskList.length;
        require(_length == _scList.length, "Project::Lengths !match");

        // Invite subcontractor for each task.
        for (uint256 i = 0; i < _length; i++) {
            _inviteSC(_taskList[i], _scList[i], false);
        }

        emit MultipleSCInvited(_taskList, _scList);
    }
```

_inviteSC() only checks non-zero address and calls inviteSubcontractor():

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L747-L762

```solidity
    function _inviteSC(
        uint256 _taskID,
        address _sc,
        bool _emitEvent
    ) internal {
        // Revert if sc to invite is address 0
        require(_sc != address(0), "Project::0 address");

        // Internal call to tasks invite contractor
        tasks[_taskID].inviteSubcontractor(_sc);

        // If `_emitEvent` is true (called via changeOrder) then emit event
        if (_emitEvent) {
            emit SingleSCInvited(_taskID, _sc);
        }
    }
```

inviteSubcontractor() just sets the new value:

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/libraries/Tasks.sol#L106-L111

```solidity
    function inviteSubcontractor(Task storage _self, address _sc)
        internal
        onlyInactive(_self)
    {
        _self.subcontractor = _sc;
    }
```

Task is paid only on completion by setComplete():

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L349-L356

```solidity
        // Mark task as complete. Only works when task is active.
        tasks[_taskID].setComplete();

        // Transfer funds to subcontractor.
        currency.safeTransfer(
            tasks[_taskID].subcontractor,
            tasks[_taskID].cost
        );
```

This way the absence of `getAlerts(_taskID)[2]` check and checkSignatureTask() call in inviteSC() provides a way for builder or contractor to steal task budget from a subcontractor.


## Recommended Mitigation Steps

Consider calling checkSignatureTask() when `getAlerts(_taskID)[2]` is true, schematically:

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L310-L313

```solidity
        // Invite subcontractor for each task.
        for (uint256 i = 0; i < _length; i++) {
+           if (getAlerts(_taskList[i])[2])
+               checkSignatureTask(_data_with_scList[i], _signature, _taskList[i]);        
            _inviteSC(_taskList[i], _scList[i], false);
        }
```

This approach is already implemented in changeOrder() where `_newSC` is a part of hash that has to be signed by all the parties:

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L386-L403

```solidity
    function changeOrder(bytes calldata _data, bytes calldata _signature)
        external
        override
        nonReentrant
    {
        // Decode params from _data
        (
            uint256 _taskID,
            address _newSC,
            uint256 _newCost,
            address _project
        ) = abi.decode(_data, (uint256, address, uint256, address));

        // If the sender is disputes contract, then do not check for signatures.
        if (_msgSender() != disputes) {
            // Check for required signatures.
            checkSignatureTask(_data, _signature, _taskID);
        }
```

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L477-L481

```solidity
            // If new subcontractor is not zero address.
            if (_newSC != address(0)) {
                // Invite the new subcontractor for the task.
                _inviteSC(_taskID, _newSC, true);
            }
```

checkSignatureTask() checks all the signatures:

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L855-L861

```solidity
            // When builder has not delegated rights to contractor
            else {
                // Check for B, SC and GC signatures
                checkSignatureValidity(builder, _hash, _signature, 0);
                checkSignatureValidity(contractor, _hash, _signature, 1);
                checkSignatureValidity(_sc, _hash, _signature, 2);
            }
```

