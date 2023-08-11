---
description: A front-end for interchain token transfers
---

# Deploy a UI for your Warp Route

After you [deploy-a-warp-route.md](deploy-a-warp-route.md "mention"), you may want to deploy a UI for it. You can use the [UI template](https://github.com/hyperlane-xyz/hyperlane-warp-ui-template) and customize it to fit your needs.

### Configure & Customize the UI

Follow these [instructions](https://github.com/hyperlane-xyz/hyperlane-warp-ui-template/blob/main/CUSTOMIZE.md) for details on how to configure the UI's tokens and change the default branding assets/theme.

#### Configure Tokens

As mentioned in the [customization instructions](https://github.com/hyperlane-xyz/hyperlane-warp-ui-template/blob/main/CUSTOMIZE.md), the UI repo contains a token list (see `./src/consts/tokens.ts)`which must be updated. Here's an example:

```typescript
import { WarpTokenConfig } from '../features/tokens/types';

export const tokenList: WarpTokenConfig = [
  {
    chainId: 5,
    address: '0xb4fbf271143f4fbf7b91a5ded31805e42b2208d6',
    hypCollateralAddress: '0x145de8760021c4ac6676376691b78038d3DE9097',
    type: 'collateral' // or 'native'
    name: 'Weth',
    symbol: 'WETH',
    decimals: 18,
    logoURI: '/logos/weth.png',
  },
];

```

You can replace the `tokens` entry with the output that was written to `hyperlane-deploy/artifacts/warp-ui-token-list.ts` from the [deploy-a-warp-route.md](deploy-a-warp-route.md "mention") instructions.

#### Configure Chain

In addition, custom chains also need to be configured, in \`./src/consts/chains.ts\`. This should be the same configuration as the one used in the [#setup](../deploy-hyperlane.md#setup "mention") step when deploying Hyperlane and the [#example-1](deploy-a-warp-route.md#example-1 "mention")when deploying the Warp Route. Here's an example:

<pre class="language-typescript"><code class="lang-typescript"><strong>  anvil1: {
</strong>    chainId: 31337,
    name: 'anvil1',
    displayName: 'Anvil 1 Local',
    nativeToken: { name: 'Ether', symbol: 'ETH', decimals: 18 },
    publicRpcUrls: [{ http: 'http://127.0.0.1:8545' }],
    blocks: {
      confirmations: 1,
      reorgPeriod: 0,
      estimateBlockTime: 10,
    },
    logoURI: '/logo.svg',
  },
};

</code></pre>

### Deploy the UI

Since the UI is a Next.js app, you can use your favorite hosting service to host it. We recommend [Vercel](https://vercel.com), which works very well with Next. [Netlify](https://www.netlify.com) and [Fleek](https://fleek.co) are also a good options.

* Sign up for [Vercel](https://vercel.com/)
* Create a new project
* Connect it to your Git repo
* Hit Deploy!

And that's it! Now you and your users can use the UI to send tokens from the collateral chain to remote chains, from one remote chain to another, and from any remote chain back to the collateral chain.

#### Stranded Whale Problem

A common problem with token bridges like Warp Routes is that a user may transfer a token like USDC to a new chain, but only afterwords realize that they do not have the native gas token to move those tokens anywhere including back. You may consider modifying the UI to warn the users of this situation, or better faucet them some dust of native gas token so improve their experience. You can either do so by modifying the warp route contracts where it holds some balance of the native token and can share that with recipients, or you could build an off-chain service which just observes the[ `ReceivedTransferRemote`](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/11d9d0bf0d5ac9f2f57141e0e7549dd415e0325e/typescript/token/contracts/libs/TokenRouter.sol#L90)transfer the native gas token to the recipient.
