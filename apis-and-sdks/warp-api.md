---
description: Move your token between chains
---

# Warp Route API

Developers can use Hyperlane's Warp API to permissionlessly deploy "Warp Routes", contracts that allow ERC20 tokens to move effortlessly between chains.

Unlike other token wrapping protocols, Warp Routes are secured by Hyperlane's modular [sovereign-consensus](../protocol/sovereign-consensus/ "mention") protocol, allowing developers to specify the security model that governs the minting, burning, and unwrapping of their interchain token.

### Overview

A Hyperlane Warp Route allows a particular token to be moved between chains according to a  security model specified by the deployer.

Each Warp Route consists of one contract deployed on every chain that the token can move between. These contracts use the [messaging-api](../apis/messaging-api/ "mention") to send interchain messages to one another.&#x20;

When a user transfers from the _canonical_ origin chain to a _non-canonical_ destination chain, their tokens are locked in a `HypERC20Collateral` contract, which sends a message to the destination chain to mint wrapped tokens.

When a user transfers between non-canonical chains, their wrapped tokens are burned on the origin chain, which sends a message to the destination chain to mint wrapped tokens.

Finally, if a user transfers from a non-canonical origin chain back to the canonical destination chain, their wrapped tokens are burned on the origin chain, which sends a message to the destination chain to release the tokens locked in the`HypERC20Collateral` contract.

### Interface

Hyperlane Warp Route exposes the following token interface. Warp Route tokens implement this interface, in addition to the standard `ERC20` interface.

```solidity
/// @notice An interchain extension of the ERC20 interface
interface IHypERC20 is IERC20 {
  /**
    * @notice Transfers tokens to the specified recipient on a remote chain
    * @param _destination The domain ID of the destination chain
    * @param _recipient The address of the recipient, encoded as bytes32
    * @param _amount The amount of tokens to transfer
    */
  function transferRemote(
    uint32 _destination,
    bytes32 _recipient,
    uint256 _amount
  ) external payable;
}
```



### Security considerations

The deployer of a Warp Route can optionally specify the [interchain-security-modules.md](../protocol/sovereign-consensus/interchain-security-modules.md "mention") (ISMs) that are used to verify interchain transfer messages.

This means that each Warp Route may have a unique security configuration. Users transferring interchain tokens should understand the trust assumptions of a Route before using it.&#x20;

Similarly, Warp front-ends should maintain a list of known Routes so as to avoid recommending an insecure Route. The reference UI supports a minor modification to the [TokenList](https://tokenlists.org/) standard so that curators can create lists of "safe" Routes.

