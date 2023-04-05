---
description: Receive an inbound message from any Hyperlane supported network
---

# Receive

{% hint style="warning" %}
Developers **must** implement the `IMessageRecipient` interface in order to be able to receive interchain messages.
{% endhint %}

### Interface

```solidity
// SPDX-License-Identifier: MIT OR Apache-2.0
pragma solidity >=0.6.11;

interface IMessageRecipient {
    /**
     * @notice Handle an interchain message
     * @param _origin Domain ID of the chain from which the message came
     * @param _sender Address of the message sender on the origin chain as bytes32
     * @param _body Raw bytes content of message body
     */
    function handle(
        uint32 _origin,
        bytes32 _sender,
        bytes calldata _body
    ) external;
}

```

## Security

### Access control

{% hint style="danger" %}
To ensure only valid interchain messages are accepted, it is important to require that `msg.sender` is a known Hyperlane [messaging.md](../../protocol/messaging.md "mention").
{% endhint %}

Developers must implement access control on `handle()` in order to ensure the security of interchain messages. Only a Hyperlane [messaging.md](../../protocol/messaging.md "mention") contract should have permission to call this function. An example of this access control is shown below.

```solidity
address constant mailbox = 0x0E3239277501d215e17a4d31c487F86a425E110B;
// for access control on handle implementations
modifier onlyMailbox() {
    require(msg.sender == mailbox);
    _;    
}

function handle(
    uint32 _origin,
    bytes32 _sender,
    bytes calldata _body
) external onlyMailbox {};
```

### Interchain security modules

Developers can override Hyperlane's default [multisig-ism.md](../../protocol/sovereign-consensus/multisig-ism.md "mention")-based security model by specifying their own [sovereign-consensus](../../protocol/sovereign-consensus/ "mention").

See the [sovereign-consensus](../../protocol/sovereign-consensus/ "mention") documentation for more info on how to set the ISM used by your application.



### Encoding

{% hint style="info" %}
Hyperlane message senders are left-padded to `bytes32` for compatibility with virtual machines that are addressed differently.
{% endhint %}

The following utility is provided in the [`TypeCasts` library](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/libs/TypeCasts.sol) for convenience.

```solidity
// alignment preserving cast
function bytes32ToAddress(bytes32 _buf) internal pure returns (address) {
    return address(uint160(uint256(_buf)));
}
```

### Example usage

An example `handle()` implementation for receiving messages is provided below.

```solidity
event Received(uint32 origin, address sender, bytes body);
function handle(
    uint32 _origin,
    bytes32 _sender,
    bytes memory _body
) external onlyMailbox() {
    address sender = bytes32ToAddress(_sender);
    emit Received(_origin, sender, _body);
}
```
