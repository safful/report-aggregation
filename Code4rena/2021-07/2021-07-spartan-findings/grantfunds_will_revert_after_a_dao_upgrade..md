## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [grantFunds will revert after a DAO upgrade.](https://github.com/code-423n4/2021-07-spartan-findings/issues/17) 

# Handle

gpersoon


# Vulnerability details

## Impact
When the DAO is upgraded via moveDao, it also updates the DAO address in BASE.
However it doesn't update the DAO address in the Reserve.sol contract.
This could be done with the function setIncentiveAddresses(..)

Now the next time grantFunds of DAO.sol is called, its tries to call:
 _RESERVE.grantFunds(...)

The grantFunds of Reserve.sol has the modifier onlyGrantor(), which checks the msg.sender == DAO.
However in the mean time the DAO has been updated and Reserve.sol doesn't know about it and thus the modifier will not allow access to the function.
Thus grantFunds will revert.

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/Dao.sol#L452
 function moveDao(uint _proposalID) internal {
        address _proposedAddress = mapPID_address[_proposalID]; // Get the proposed new address
        require(_proposedAddress != address(0), "!address"); // Proposed address must be valid
        DAO = _proposedAddress; // Change the DAO to point to the new DAO address
        iBASE(BASE).changeDAO(_proposedAddress); // Change the BASE contract to point to the new DAO address
        daoHasMoved = true; // Set status of this old DAO
        completeProposal(_proposalID); // Finalise the proposal
    }

    function grantFunds(uint _proposalID) internal {
        uint256 _proposedAmount = mapPID_param[_proposalID]; // Get the proposed SPARTA grant amount
        address _proposedAddress = mapPID_address[_proposalID]; // Get the proposed SPARTA grant recipient
        require(_proposedAmount != 0, "!param"); // Proposed grant amount must be valid
        require(_proposedAddress != address(0), "!address"); // Proposed recipient must be valid
        _RESERVE.grantFunds(_proposedAmount, _proposedAddress); // Grant the funds to the recipient
        completeProposal(_proposalID); // Finalise the proposal
    }

// https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/outside-scope/Reserve.sol#L17
  modifier onlyGrantor() {
        require(msg.sender == DAO || msg.sender == ROUTER || msg.sender == DEPLOYER || msg.sender == LEND || msg.sender == SYNTHVAULT, "!DAO");
        _; 
    }

  function grantFunds(uint amount, address to) external onlyGrantor {
      ....
    }

   function setIncentiveAddresses(address _router, address _lend, address _synthVault, address _Dao) external onlyGrantor {
        ROUTER = _router;
        LEND = _lend;
        SYNTHVAULT = _synthVault;
        DAO = _Dao;
    }


## Tools Used

## Recommended Mitigation Steps
Call setIncentiveAddresses(..) when a DAO upgrade is done.


