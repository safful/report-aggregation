## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`_requireTicket()` Implementation can be simpler and save some gas](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/90) 

# Handle

WatchPug


# Vulnerability details

https://github.com/pooltogether/v4-periphery/blob/0e94c54774a6fce29daf9cb23353208f80de63eb/contracts/TwabRewards.sol#L230-L244

```solidity=230{233,237-243}
    function _requireTicket(address _ticket) internal view {
        require(_ticket != address(0), "TwabRewards/ticket-not-zero-address");

        (bool succeeded, bytes memory data) = address(_ticket).staticcall(
            abi.encodePacked(ITicket(_ticket).controller.selector)
        );

        address controllerAddress;

        if (data.length > 0) {
            controllerAddress = abi.decode(data, (address));
        }

        require(succeeded && controllerAddress != address(0), "TwabRewards/invalid-ticket");
    }
```

### Recommendation

Change to:

```solidity=230{233,237}
    function _requireTicket(address _ticket) internal view {
        require(_ticket != address(0), "TwabRewards/ticket-not-zero-address");

        (bool succeeded, bytes memory data) = _ticket.staticcall(
            abi.encodePacked(ITicket(_ticket).controller.selector)
        );

        require(succeeded && data.length > 0 && abi.decode(data, (uint160)) != 0, "TwabRewards/invalid-ticket");
    }
```

-   Removing redundant casting of `address(_ticket)` as `_ticket` is `address`;
-   `controllerAddress` is unnecessary as it's being used only once;
-   Checking if `succeeded` earlier can avoid unnecessary code execution when this check failed;
-   Replacing `abi.decode(data, (address)) != address(0)` with `abi.decode(data, (uint160)) != 0` to avoid type casting.

