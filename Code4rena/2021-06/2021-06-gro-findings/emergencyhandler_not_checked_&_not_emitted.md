## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [emergencyHandler not checked & not emitted](https://github.com/code-423n4/2021-06-gro-findings/issues/5) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setWithdrawHandler allows the setting of withdrawHandler and emergencyHandler.
However emergencyHandler isn't checked for 0 (like the withdrawHandler )
The value of the emergencyHandler is also not emitted (like the withdrawHandler )

## Proof of Concept
// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/Controller.sol#L105
 function setWithdrawHandler(address _withdrawHandler, address _emergencyHandler) external onlyOwner {
        require(_withdrawHandler != address(0), "setWithdrawHandler: 0x");
        withdrawHandler = _withdrawHandler;
        emergencyHandler = _emergencyHandler;
        emit LogNewWithdrawHandler(_withdrawHandler);
    }

## Tools Used

## Recommended Mitigation Steps
Add something like:
        require(_emergencyHandler!= address(0), "setEmergencyHandler: 0x");
        event LogNewEmergencyHandler(address tokens);
        emit LogNewEmergencyHandler(_emergencyHandler);

