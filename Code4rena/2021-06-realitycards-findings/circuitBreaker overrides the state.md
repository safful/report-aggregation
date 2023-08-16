## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [circuitBreaker overrides the state](https://github.com/code-423n4/2021-06-realitycards-findings/issues/38) 

# Handle

pauliax


# Vulnerability details

## Impact
function circuitBreaker calls _incrementState but later sets the state itself again:
    function _incrementState() internal {
        assert(uint256(state) < 4);
        state = States(uint256(state) + (1));
        emit LogStateChange(uint256(state));
    }

    function circuitBreaker() external {
        require(
            block.timestamp > (uint256(oracleResolutionTime) + (12 weeks)),
            "Too early"
        );
        _incrementState();
        orderbook.closeMarket();
        state = States.WITHDRAW;
    }

## Recommended Mitigation Steps
state = States.WITHDRAW; shouldn't be there, or another solution would be to put it before orderbook.closeMarket(); and remove _incrementState(); instead but then LogStateChange event will also need to be emitted manually.

