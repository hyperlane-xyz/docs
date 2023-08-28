---
description: >-
  A major version change of the Mailbox, v3, is coming soon - offering a single
  call to the Hyperlane API and modular dispatch transport and security
---

# Coming Soon: Hyperlane v3

If you’re building on Hyperlane, we have exciting news. We’ve improved the Mailbox to unlock modularity in how you transport messages, while simplifying the interface.

{% hint style="warning" %}
**Note:** v3 will be a breaking change to v2 - you can expect this contract to go live in October after auditing & testing. We will be sharing the final spec closer to launch.
{% endhint %}

### Mailbox Interface

A major version change of the Mailbox, v3, is emerging with these improvements:

1. modularity of domain specific transport and security layers
2. simplifying the Hyperlane API to a single call (and address)
3. configurable dispatch time requirements such as fee payments

This proposal unifies interchain gas payments, protocol fees, and any message dispatch time behavior into a streamlined interface.

```solidity
interface IMailbox {
    function quoteDispatch(
        uint32 destination,
	bytes32 recipient,
	bytes body
    ) external returns (uint256 fee);
    function dispatch(
        uint32 destination,
	bytes32 recipient,
	bytes body
    ) external payable; // will revert if msg.value < quoted fee
}
```

This is a departure from the existing [interchain gas payment](https://docs.hyperlane.xyz/docs/apis-and-sdks/interchain-gas-paymaster-api) and [hook](https://docs.hyperlane.xyz/docs/apis-and-sdks/hooks) interfaces but should be almost entirely backwards compatible. Mirroring the [interchain security modules](https://docs.hyperlane.xyz/docs/protocol/sovereign-consensus) pre-handle lifecycle hook, the architectural change is introducing post-dispatch lifecycle hooks which receive authenticated message content from the mailbox and perform some additional task, like paying interchain gas to relayers or using a different transport layer.

![](<../.gitbook/assets/Screenshot 2023-08-28 at 1.35.30 PM.png>)

The mailbox will have a required hook needed by the protocol and a default hook that can be overridden by the message sender, further modularizing the mailbox interface. We expect the required hook to include enforcing protocol fees. These hooks will be governable, enabling Mailbox operators to add message-specific capabilities as default or required behavior incrementally without mutating message content. This mirrors default ISM governance and post-dispatch hook behavior should reflect requirements of the message destination’s ISM, default or otherwise.

### Post Dispatch Hook Interface

The post dispatch hook interface is designed for maximal generality and extensibility of implementation (quite similar to the ISM verification interface). In addition to the entire message content, metadata can be provided by the dispatcher for further expression of preferences. For hooks that charge for use, a quote interface is necessary to inform the message sender how much value must be passed to the mailbox dispatch call.

```solidity
interface IPostDispatchHook {
    function postDispatch(
        bytes metadata,
        bytes message
    ) external payable;
    function quoteDispatch(
        bytes metadata,
        bytes message
    ) external returns (uint256 fee);
}
```

### Usage Patterns

This new interface should greatly simplify the integration experience and prevent confusion for first time users.

**Before**

```solidity
// lookup gas amount, igp, and hook for destination
uint256 gasAmount = ...;
IInterchainGasPaymaster igp = ...;
IMessageHook hook = ...;

uint256 quote = igp.quoteGasPayment(destination, gasAmount);
bytes32 messageId = mailbox.dispatch(destination, recipient, body);
igp.payForGas{value: quote}(messageId, destination, gasAmount, msg.sender);
hook.postDispatch(destination, messageId); // old interface that does not allow for behavior dynamic with message content
```

**After**

```solidity
uint256 quote = mailbox.quoteDispatch(destination, recipient, body);
mailbox.dispatch{value: quote}(destination, recipient, body);
```

Alternatively, any overpayment will be refunded so quoting is not strictly necessary but dispatch may revert if payment is insufficient.

```solidity
mailbox.dispatch{value: msg.value}(destination, recipient, body);
```

If you want post-dispatch behavior on a non-default chain, such as using an OpStack bridge on messages outbound from ethereum, simply pass the corresponding hook.

```solidity
// overrides default hook with opStackHook
mailbox.dispatch{value: msg.value}(destination, recipient, body, opStackHook);
```

#### Permissionless Interoperability Composition

![](<../.gitbook/assets/Screenshot 2023-08-28 at 1.35.45 PM.png>)

## Implementation Guide

We have [implemented these changes for the EVM implementation](https://github.com/hyperlane-xyz/hyperlane-monorepo/tree/v3) of Hyperlane and are sending them off for audit, targeting mid-September for mainnet launch. The following summarizes the logical changes (in pseudocode) as a reference for other execution environments.

{% hint style="info" %}
**`param?`** indicates an optional parameter \


**`optional ?? fallback`** resolves to **`optional`** if some value is specified, **`fallback`** if it is not
{% endhint %}

### Mailbox Implementation

The incremental merkle tree has been migrated out of the mailbox and replaced by `nonce` and `latestDispatchedId` storage variables. The nonce is necessary for maintaining message uniqueness. The latest dispatched message ID is useful for hooks which need to authenticate message content independent of the caller (`msg.sender`) address.

```solidity
contract Mailbox {
    // monotonically increasing outbound message nonce
    uint256 nonce;

    // commits to latest dispatched message content
    bytes32 latestDispatchedId;

    // implements defaults that can be overriden (overhead IGP, etc)
    IPostDispatchHook public defaultHook;

    // enforces protocol fees or any future required hooks
    IPostDispatchHook public requiredHook;

    function dispatch(
        uint32 destination,
        bytes32 recipient,
        bytes calldata body,
        IPostDispatchHook customHook?, // optional
        bytes calldata hookMetadata?   // optional
    ) external payable returns (bytes32 messageId) {
        ...
        // effects
        nonce += 1;
        bytes32 messageId = message.id();
        latestDispatchedId = messageId;
        emit Dispatch(message);
        emit DispatchId(messageId);

        // interactions
        requiredHook.postDispatch{value: msg.value}(hookMetadata, message);
        (customHook ?? defaultHook).postDispatch{value: msg.value}(hookMetadata, message);

        return messageId;
    }

    function quoteDispatch(
        uint32 destination,
        bytes32 recipient,
        bytes calldata body,
        IPostDispatchHook customHook?, // optional
        bytes calldata hookMetadata?   // optional
    ) external view returns (uint256 fee) {
        fee = (customHook ?? defaultHook).quoteDispatch(hookMetadata ?? "", message)
                    + requiredHook.quoteDispatch(hookMetadata ?? "", message);
    }
}
```

### Interchain Gas Paymaster Hook Implementation

As a motivating example, the default hook may contain an interchain gas payment hook with defaults for gas limit and refund address that may be overridden via metadata.

```solidity
contract IgpHook is InterchainGasPaymaster {
    uint256 immutable DEFAULT_GAS_LIMIT;

    function postDispatch(
        bytes metadata,
        bytes message
    ) external payable {
        uint256 gasLimit = metadata.gasLimit() ?? DEFAULT_GAS_LIMIT;
        address refundAddress = metadata.refundAddress() ?? message.sender();
        payGasFor(
            message.id(),
            message.destination(),
            gasLimit, 
    refundAddress
        );
    }
}
```

### Your Feedback

We welcome feedback or questions about how this change impacts you. Please do reach out in the #developers channel in Discord.
