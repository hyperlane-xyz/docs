---
description: Smarter interchain security models
---

# Routing ISM

Developers can use a `RoutingISM` to delegate message verification to a different ISM. This allows developers to change security models based on message content or application context.&#x20;

## Interface

`RoutingISMs` must implement the `IRoutingIsm` interface.

```solidity
interface IRoutingIsm is IInterchainSecurityModule {
    /**
     * @notice Returns the ISM responsible for verifying _message
     * @dev Can change based on the content of _message
     * @param _message Formatted Hyperlane message (see Message.sol).
     * @return module The ISM to use to verify _message
     */
    function route(bytes calldata _message)
        external
        view
        returns (IInterchainSecurityModule);
}
```

## Configure

The hyperlane-monorepo contains a `RoutingISM` implementation, `DomainRoutingIsm`, that application developers can deploy off-the-shelf, specifying their desired configuration.

This ISM simply switches security models depending on the origin chain of the message. A simple use case for this is to use different [multisig-ism.md](multisig-ism.md "mention") validator sets for each chain.

Eventually, you could imagine a `DomainRoutingIsm` routing to different light-client-based ISMs, depending on the type of consensus protocol used on the origin chain.

The [hyperlane-deploy repo](https://github.com/hyperlane-xyz/hyperlane-deploy) contains the tooling and instructions needed to deploy and configure a `DomainRoutingIsm`.

## Customize

The hyperlane-monorepo contains an abstract `RoutingISM` implementation that application developers can fork.

Developers simply need to implement the `route()` function.

By creating a custom implementation, application developers can tailor the security provided by a `RoutingISM` to the needs of their application.

For example, a custom implementation could change security models based on the contents of the message or the state of the application receiving the message.
