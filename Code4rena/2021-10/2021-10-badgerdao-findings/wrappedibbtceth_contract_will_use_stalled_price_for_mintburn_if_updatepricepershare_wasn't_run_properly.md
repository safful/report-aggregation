## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [WrappedIbbtcEth contract will use stalled price for mint/burn if updatePricePerShare wasn't run properly](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/86) 

# Handle

hyh


# Vulnerability details

## Impact
Malicious user can monitor SetPricePerShare event and, if it was run long enough time ago and market moved, but, since there were no SetPricePerShare fired, the contract's pricePerShare is outdated, so a user can mint() with pricePerShare that is current for contract, but outdated for market, then wait for price update and burn() with updated pricePerShare, yielding risk-free profit at expense of contract holdings.

## Proof of Concept
WrappedIbbtcEth updates pricePerShare variable by externally run updatePricePerShare function. The variable is then used in mint/burn/transfer functions without any additional checks, even if outdated/stalled. This can happen if the external function wasn't run for any reason.
The variable is used via balanceToShares function: https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L155

This is feasible as updatePricePerShare to be run by off-chain script being a part of the system, and malfunction of this script leads to contract exposure by stalling the price. The malfunction can happen both by internal reasons (bugs) and by external ones (any system-level dependencies, network outrages).
updatePricePerShare function: https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L72

## Recommended Mitigation Steps
The risk comes with system design. Wrapping price updates with contract level variable for gas costs minimization is a viable approach, but it needs to be paired with corner cases handling. One of the ways to reduce the risk is as follows:

Introduce a threshold variable for maximum time elapsed since last pricePerShare update to WrappedIbbtcEth contract.

Then 2 variants of transferFrom and transfer functions can be introduced, both check condition {now - time since last price update < threshold}. If condition holds both variants do the transfer. If it doesn't the first variant reverts, while the second do costly price update.
I.e. it will be cheap transfer, that works only if price is recent, and full transfer, that is similar to the first when price is recent, but do price update on its own when price is stalled. This way this full transfer is guaranteed to run and is usually cheap, costing more if price is stalled and it does the update.

After this whenever scheduled price update malfunctions (for example because of network conditions), the risk will be limited by market volatility during threshold time at maximum, i.e. capped.

Example code:

		// Added threshold

    uint256 public pricePerShare;
    uint256 public lastPricePerShareUpdate;
		uint256 public priceUpdateThreshold;
		
		event SetPriceUpdateThreshold(uint256 priceUpdateThreshold);
		
    /// ===== Permissioned: Price update threshold =====
    function setPriceUpdateThreshold(uint256 _priceUpdateThreshold) external onlyGovernance {
        priceUpdateThreshold = _priceUpdateThreshold;
        emit SetPriceUpdateThreshold(priceUpdateThreshold);
    }
		
		// The only difference with current transfer code is that Full versions call balanceToSharesFull

    function transferFrom(address sender, address recipient, uint256 amount) public virtual override returns (bool) {
        uint256 amountInShares = balanceToShares(amount);

        _transfer(sender, recipient, amountInShares);
        _approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amountInShares, "ERC20: transfer amount exceeds allowance"));
        return true;
    }

    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
        uint256 amountInShares = balanceToShares(amount);

        _transfer(_msgSender(), recipient, amountInShares);
        return true;
    }
		
    function transferFromFull(address sender, address recipient, uint256 amount) public virtual override returns (bool) {
        uint256 amountInShares = balanceToSharesFull(amount);

        _transfer(sender, recipient, amountInShares);
        _approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amountInShares, "ERC20: transfer amount exceeds allowance"));
        return true;
    }

    function transferFull(address recipient, uint256 amount) public virtual override returns (bool) {
        uint256 amountInShares = balanceToSharesFull(amount);

        _transfer(_msgSender(), recipient, amountInShares);
        return true;
    }
		
		// Now balanceToShares first checks if the price is stale
		// And reverts if it is
		// While balanceToSharesFull do the same check
		// But asks for price instead of reverting
		// Having guaranteed execution with increased costs sometimes
		// Which is fully deterministic, as user can track SetPricePerShare event
		// To understand whether it be usual or increased gas cost if the function be called now

    function balanceToShares(uint256 balance) public view returns (uint256) {
				require(block.timestamp < lastPricePerShareUpdate + priceUpdateThreshold, "Price is stalled");
        return balance.mul(1e18).div(pricePerShare);
    }
		
    function balanceToSharesFull(uint256 balance) public view returns (uint256) {
				if (block.timestamp >= lastPricePerShareUpdate + priceUpdateThreshold) {
						updatePricePerShare();
				}
        return balance.mul(1e18).div(pricePerShare);
    }

