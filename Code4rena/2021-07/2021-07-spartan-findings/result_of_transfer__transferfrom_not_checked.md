## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Result of transfer / transferFrom not checked](https://github.com/code-423n4/2021-07-spartan-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
A call to transferFrom or transfer is frequently done without checking the results.
For certain ERC20 tokens, if insufficient tokens are present, no revert occurs but a result of "false" is returned.
So its important to check this. If you don't you could mint tokens without have received sufficient tokens to do so. So you could loose funds.

Its also a best practice to check this.
See below for example where the result isn't checked.

Note, in some occasions the result is checked (see below for examples).

## Proof of Concept
Highest risk:
.\Dao.sol:                iBEP20(_token).transferFrom(msg.sender, address(this), _amount); // Transfer user's assets to Dao contract
.\Pool.sol:               iBEP20(TOKEN).transfer(member, outputToken); // Transfer the TOKENs to user
.\Pool.sol:               iBEP20(token).transfer(member, outputAmount); // Transfer the swap output to the selected user
.\poolFactory.sol:   iBEP20(_token).transferFrom(msg.sender, _pool, _amount);
.\Router.sol:           iBEP20(_fromToken).transfer(fromPool, iBEP20(_fromToken).balanceOf(address(this))); // Transfer TOKENs from ROUTER to fromPool
.\Router.sol:           iBEP20(_token).transfer(_pool, iBEP20(_token).balanceOf(address(this))); // Transfer TOKEN to pool
.\Router.sol:           iBEP20(_token).transferFrom(msg.sender, _pool, _amount); // Transfer TOKEN to pool
.\Router.sol:           iBEP20(_token).transfer(_recipient, _amount); // Transfer TOKEN to recipient
.\Synth.sol:             iBEP20(_token).transferFrom(msg.sender, address(this), _amount); // Transfer tokens in

less risky
.\Router.sol:           iBEP20(fromPool).transferFrom(_member, fromPool, unitsInput); // Transfer LPs from user to the pool
.\BondVault.sol:     iBEP20(_pool).transfer(member, _claimable); // Send claim amount to user
.\Router.sol:           iBEP20(_pool).transferFrom(_member, _pool, units); // Transfer LPs to the pool
.\Router.sol:           iBEP20(_pool).transferFrom(_member, _pool, units); // Transfer LPs to pool
.\Router.sol:           iBEP20(fromSynth).transferFrom(msg.sender, _poolIN, inputAmount); // Transfer synth from user to pool
.\Pool.sol:               iBEP20(synthIN).transfer(synthIN, _actualInputSynth); // Transfer SYNTH to relevant synth contract
.\Router.sol:           iBEP20(WBNB).transfer(_pool, _amount); // Transfer WBNB from ROUTER to pool
.\Dao.sol:               iBEP20(BASE).transfer(newDAO, baseBal);
.\Pool.sol:               iBEP20(BASE).transfer(member, outputBase); // Transfer the SPARTA to user
.\Pool.sol:               iBEP20(BASE).transfer(member, outputBase); // Transfer SPARTA to user
.\Router.sol:           iBEP20(BASE).transfer(toPool, iBEP20(BASE).balanceOf(address(this))); // Transfer SPARTA from ROUTER to toPool
.\Router.sol:           iBEP20(BASE).transfer(_pool, iBEP20(BASE).balanceOf(address(this))); // Transfer SPARTA to pool
.\Router.sol:           iBEP20(BASE).transfer(_pool, iBEP20(BASE).balanceOf(address(this))); // Transfer SPARTA from ROUTER to pool
.\Router.sol:           iBEP20(BASE).transferFrom(msg.sender, _pool, inputAmount); // Transfer SPARTA from ROUTER to pool

Sometimes the result is checked:
.\Dao.sol:              require(iBEP20(pool).transferFrom(msg.sender, address(_DAOVAULT), amount), "!funds"); // Send user's deposit to the DAOVault
.\Dao.sol:              require(iBEP20(BASE).transferFrom(msg.sender, address(_RESERVE), _amount), '!fee'); // User pays the new proposal fee
.\DaoVault.sol:      require(iBEP20(pool).transfer(member, _balance), "!transfer"); // Transfer user's balance to their wallet
.\synthVault.sol:    require(iBEP20(synth).transferFrom(msg.sender, address(this), amount)); // Must successfuly transfer in
.\synthVault.sol:    require(iBEP20(synth).transfer(msg.sender, redeemedAmount)); // Transfer from SynthVault to user

## Tools Used
grep

## Recommended Mitigation Steps
Always check the result of transferFrom and transfer


