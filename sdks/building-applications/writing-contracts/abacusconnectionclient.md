---
description: The easiest way to integrate with Hyperlane
---

# HyperlaneConnectionClient

Inheriting from [`HyperlaneConnectionClient`](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/HyperlaneConnectionClient.sol) is a simple way to ensure your contract knows where to send or receive interchain messages to or from.

This mix-in contract maintains a pointers to the three contracts Hyperlane developers may need to interact with:

1. [`Mailbox`](../../../protocol/messaging.md) (required)
2. [`InterchainGasPaymaster`](broken-reference) (optional)
3. [`InterchainSecurityModule`](../../../protocol/sovereign-consensus/) (optional)

`HyperlaneConnectionClient` exposes functions that allow subclasses to easily send messages to the `Mailbox` via the `mailbox` storage variable, and permission message delivery via the `onlyMailbox` modifier.

```solidity
import {HyperlaneConnectionClient} from "@hyperlane-xyz/core/contracts/HyperlaneConnectionClient.sol";

contract HelloWorld is HyperlaneConnectionClient {
  
  /**
   * @notice Sends a "hello world" message to an address on a remote chain.
   * @param _destination The ID of the chain we're sending the message to.
   * @param _recipient The address of the recipient we're sending the message to.
   */
  function sendHelloWorld(uint32 _destination, address _recipient) external {
    // The message that we're sending.
    bytes memory _message = "hello world";
    // Send the message! 
    mailbox.dispatch(_destination, _recipient, _message);
  }

  /**
   * @notice Emits a HelloWorld event upon receipt of an interchain message
   * @param _origin The chain ID from which the message was sent
   * @param _sender The address that sent the message
   * @param _message The contents of the message
   */
  function handle(
    uint32 _origin,
    bytes32 _sender,
    bytes memory _message
  ) external override onlyMailbox {
    emit HelloWorld(_origin, _sender, _message);
  }
}
```

\_\_
