---
description: Create an interchain route for your token
---

# Deploy a Warp Route

A Warp Route is a type of [router.md](../../sdks/building-applications/writing-contracts/router.md "mention") application, requiring a `HypERC20` or `HypERC721` token contract to be deployed on each chain you wish to support.

The [hyperlane-deploy](https://github.com/hyperlane-xyz/hyperlane-deploy) repo includes a script to configure and deploy a Warp Route for your desired token.

## 1. Setup

Clone the [hyperlane-deploy](https://github.com/hyperlane-xyz/hyperlane-deploy) repo and run the following commands:

```
$ yarn install
$ yarn build
```

## 2. Configuration

### Warp Route config

You will need to create a `WarpRouteConfig` in [`hyperlane-deploy/config/warp_tokens.ts`](https://github.com/hyperlane-xyz/hyperlane-deploy/blob/main/config/warp\_tokens.ts) to define your Warp Route. This will include information such as:

* Which token, on which chain, is this Warp Route being created for?
* _Optional:_ Hyperlane connection details including contract addresses for [messaging.md](../../protocol/messaging.md "mention"), [#interchaingaspaymasters](../../protocol/interchain-gas-payments.md#interchaingaspaymasters "mention"), and [sovereign-consensus](../../protocol/sovereign-consensus/ "mention").
* _Optional:_ The token standard - fungible tokens using ERC20 or NFTs using ERC721. Defaults to ERC20.

#### Base

Your `WarpRouteConfig` must have exactly one `base` entry. Here you will configure details about the token for which you are creating a warp route.

* **chainName**: Set this equal to the chain on which your token exists
* **type**: Set this to `TokenType.collateral` to create a warp route for an ERC20/ERC721 token, or `TokenType.native` to create a warp route for a native token (e.g. ether)
* **address:** If using `TokenType.collateral`, the address of the ERC20/ERC721 contract for which to create a route
* **isNft:** If using `TokenType.collateral` for an ERC721 contract, set to `true`.

#### Synthetics

Your `WarpRouteConfig` must have at least one `synthetics` entry. Here you will configure details about the remote chains supported by your warp route.

* **chainName:** Set this equal to the chain on which you want a wrapped version of your token

#### Options

You may specify the following optional values in your `base` and `synthetics` entries. If no values are provided, defaults will be populated from `hyperlane-deploy/artifacts/addresses.json` and the [SDK](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/typescript/sdk/src/consts/environments/mainnet.json) (if present).

* **mailbox:** The address of the [messaging.md](../../protocol/messaging.md "mention")contract to use to send and receive messages
* **interchainSecurityModule:** The address of an [sovereign-consensus](../../protocol/sovereign-consensus/ "mention") to verify interchain messages
* **interchainGasPaymaster:** The address of a [interchain-gas-payments.md](../../protocol/interchain-gas-payments.md "mention") to pay for the gas needed to deliver interchain messages

#### Example

An example `WarpRouteConfig` is provided in [`hyperlane-deploy/config/warp_tokens.ts`](https://github.com/hyperlane-xyz/hyperlane-deploy/blob/main/config/warp\_tokens.ts)that defines a warp route for a native token between two local chains.

This Warp Route is secured by the default [sovereign-consensus](../../protocol/sovereign-consensus/ "mention") that are set on the `Mailboxes` for those chains.

```typescript
import { TokenType } from '@hyperlane-xyz/hyperlane-token';

import type { WarpRouteConfig } from '../src/warp/config';

export const warpRouteConfig: WarpRouteConfig = {
  base: {
    // Chain name must be in the Hyperlane SDK or in the chains.ts config
    chainName: 'anvil1',
    type: TokenType.native, //  TokenType.native or TokenType.collateral
    // If type is collateral, a token address is required:
    // address: '0x123...'

    // Optionally, specify owner, mailbox, and interchainGasPaymaster addresses
    // If not specified, the Permissionless Deployment artifacts or the SDK's defaults will be used
  },
  synthetics: [
    {
      chainName: 'anvil2',

      // Optionally, specify owner, mailbox, and interchainGasPaymaster addresses
      // If not specified, the Permissionless Deployment artifacts or the SDK's defaults will be used
    },
  ],
};
```

### Chain config

The Warp Route deployer will be aware of the connection details (e.g. RPC URL) for many standard chains.

If you would like to deploy a Warp Route to a chain that is not included in the Hyperlane SDK, you can specify a [`ChainMetadata`](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/typescript/sdk/src/consts/chainMetadata.ts#L21) entry in [`hyperlane-deploy/config/chains.ts`](https://github.com/hyperlane-xyz/hyperlane-deploy/blob/main/config/chains.ts).

An example has been populated for you for [`anvil`](https://book.getfoundry.sh/anvil/).

#### Example

An example chain config for a Warp Route is shown below. The `blocks` and `blockExplorers` properties are optional.

```typescript
export const chains: ChainMap<ChainMetadata> = {
  // ----------- Your chains here -----------------
  anvil1: {
    name: 'anvil1',
    // anvil default chain id
    chainId: 31337,
    publicRpcUrls: [
      {
        http: 'http://localhost:8545',
      },
    ],
  },
};
```

## 3. Deployment

Run the following script to deploy your Warp Route. You will need to provide the following arguments:

* `key`: A hexadecimal private key for transaction signing

```bash
DEBUG=hyperlane* yarn ts-node scripts/deploy-warp-routes.ts \
  --key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

When the command finishes, it will output the list of contracts addresses to `hyperlane-deploy/artifacts/warp-token-addresses.json.`

The deployer will also output a token list file to `hyperlane-deploy/artifacts/warp-ui-token-list.ts` which can be used to [deploy-the-ui-for-your-warp-route.md](deploy-the-ui-for-your-warp-route.md "mention").

## 4. Testing

Run the following script to test your Warp Route by transferring tokens from one chain to another. You will need to provide the following arguments:

* `origin`: The name of the chain that you are sending tokens from
* `destination`: The name of the chain that you are sending tokens to
* `wei`: The value of tokens to transfer, in wei
* `recipient`: The address to send the tokens to on the destination chain
* `key`: A hexadecimal private key for transaction signing

```bash
yarn ts-node scripts/test-warp-transfer.ts \
  --origin anvil1 --destination anvil2 --wei 1 \
  --recipient 0xac0974bec39a17e36ba4a6b4d238ff944bacb4a5 \
  --key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80
```

If everything goes well, you should see the following output:

```
Waiting for message delivery on destination chain
Waiting for message delivery on destination chain
Waiting for message delivery on destination chain
Waiting for message delivery on destination chain
Waiting for message delivery on destination chain
Waiting for message delivery on destination chain
Message delivered on destination chain!
Confirmed balance increase
Warp test transfer complete
```

{% content-ref url="deploy-the-ui-for-your-warp-route.md" %}
[deploy-the-ui-for-your-warp-route.md](deploy-the-ui-for-your-warp-route.md)
{% endcontent-ref %}
