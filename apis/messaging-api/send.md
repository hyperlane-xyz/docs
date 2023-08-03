---
description: Send a message to any Hyperlane supported network
---

# Send

Developers can send interchain messages by calling `Mailbox.dispatch()`.

### Interface

{% hint style="warning" %}
Hyperlane can only deliver messages to smart contracts that implement the `IMessageRecipient` interface. See the [receive.md](receive.md "mention") documentation for more information.
{% endhint %}

```solidity
    /**
     * @notice Dispatches a message to the destination domain & recipient.
     * @param _destinationDomain Domain of destination chain
     * @param _recipientAddress Address of recipient on destination chain as bytes32
     * @param _body Raw bytes content of message body
     * @return The message ID inserted into the Mailbox's merkle tree
     */
    function dispatch(
        uint32 _destination,
        bytes32 _recipient,
        bytes calldata _body
    ) external returns (bytes32) {
```

See [addresses](../../resources/addresses/ "mention")and [domains](../../resources/domains/ "mention") for `Mailbox` contract addresses, and chain domain IDs, respectively.

### Encoding

{% hint style="info" %}
Recipient addresses are left-padded to `bytes32` for compatibility with virtual machines that are addressed differently.
{% endhint %}

The following utility is provided in the [`TypeCasts` library](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/libs/TypeCasts.sol) for convenience.

```solidity
// alignment preserving cast
function addressToBytes32(address _addr) internal pure returns (bytes32) {
    return bytes32(uint256(uint160(_addr)));
}
```

### Paying for Interchain Gas

Delivering a message to its destination requires submitting a transaction on the destination chain. If you want to have [relayer.md](../../protocol/agents/relayer.md "mention") deliver the message on your behalf, you can pay for the gas for this transaction on the origin chain.

Learn more about [paying-for-interchain-gas.md](../../build-with-hyperlane/guides/paying-for-interchain-gas.md "mention").

### Example Usage

The code snippet below shows an example of sending a message from Ethereum to Avalanche.

{% hint style="warning" %}
Note this example does not pay for interchain gas, and therefore would not be delivered automatically by a relayer.
{% endhint %}

```solidity
uint32 constant avalancheDomain = 43114;
address constant avalancheRecipient = 0x36FdA966CfffF8a9Cdc814f546db0e6378bFef35;
address constant ethereumMailbox = 0x2f9db5616fa3fad1ab06cb2c906830ba63d135e3;
IMailbox(ethereumMailbox).dispatch(
    avalancheDomain,
    addressToBytes32(avalancheRecipient),
    bytes("hello avalanche from ethereum")
);
```
