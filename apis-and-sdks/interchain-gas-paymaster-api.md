---
description: Pay for message delivery on the origin chain
---

# Interchain gas paymaster API

&#x20;Hyperlane provides an on-chain API on the origin chain that allows message senders to pay one or more [relayers](../operators/relayers/ "mention") to deliver a message on the destination chain.

{% hint style="info" %}
Learn more about [interchain-gas-payments.md](../protocol/interchain-gas-payments.md "mention") protocol [here](../protocol/interchain-gas-payments.md)
{% endhint %}

## Interface

The interchain gas paymaster (IGP) interface exposes two functions that can be used together to quote and pay for interchain gas.

1. `quoteGasPayment` is a view function that allows users to get a quote for an amount of gas on the destination chain, denominated in the origin chain's native token.
2. `payGasFor` is a payable function that pays a relayer to deliver a specific message to the destination chain

These functions do not necessarily need to be called in the same transaction as the message dispatch.

You can find deployed IGPs on [addresses](../resources/addresses/ "mention")

```solidity
// SPDX-License-Identifier: MIT OR Apache-2.0
pragma solidity >=0.6.11;

/**
 * @title IInterchainGasPaymaster
 * @notice Manages payments on a source chain to cover gas costs of relaying
 * messages to destination chains.
 */
interface IInterchainGasPaymaster {
    /**
     * @notice Emitted when a payment is made for a message's gas costs.
     * @param messageId The ID of the message to pay for.
     * @param gasAmount The amount of destination gas paid for.
     * @param payment The amount of native tokens paid.
     */
    event GasPayment(
        bytes32 indexed messageId,
        uint256 gasAmount,
        uint256 payment
    );

    /**
     * @notice Deposits msg.value as a payment for the relaying of a message
     * to its destination chain.
     * @dev Overpayment will result in a refund of native tokens to the _refundAddress.
     * Callers should be aware that this may present reentrancy issues.
     * @param _messageId The ID of the message to pay for.
     * @param _destinationDomain The domain of the message's destination chain.
     * @param _gasAmount The amount of destination gas to pay for.
     * @param _refundAddress The address to refund any overpayment to.
     */
    function payForGas(
        bytes32 _messageId,
        uint32 _destinationDomain,
        uint256 _gasAmount,
        address _refundAddress
    ) external payable;

    /**
     * @notice Quotes the amount of native tokens to pay for interchain gas.
     * @param _destinationDomain The domain of the message's destination chain.
     * @param _gasAmount The amount of destination gas to pay for.
     * @return The amount of native tokens required to pay for interchain gas.
     */
    function quoteGasPayment(uint32 _destinationDomain, uint256 _gasAmount)
        external
        view
        returns (uint256);
}
```

## Refunds

Users may call `payGasFor` with `msg.value` greater than what would be returned by `quoteGasPayment`. In this case, the IGP contract should refund the difference to the `_refundAddress` provided. This allows users to not need to call `quoteGasPayment` explicitly.

For example, if you are paying for 1M gas and pass 0.1 ETH to the IGP contract, but `quoteGasPayment` quotes a payment of 0.08 ETH, the `_refundAddress` will be refunded `0.02` ETH.

Note that refunds are only made if what is paid was greater than what was quoted. Refunds are **not** made if delivery of a message on the destination chain winds up requiring less gas than what was paid for.

#### Reentrancy Risk

Note that refunding overpayment involves the IGP contract calling the `_refundAddress`, which can present a risk of [reentrancy](https://www.certik.com/resources/blog/3K7ZUAKpOr1GW75J2i0VHh-what-is-a-reentracy-attack) for your application. Special care should be made by callers to ensure they are not vulnerable to reentrancy exploits.

#### Unsuccessful Refunds

If a refund is unsuccessful, the `payForGas` call will revert.



