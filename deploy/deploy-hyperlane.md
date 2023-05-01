---
description: Deploy Hyperlane to any smart contract environment of your choice.
---

# Deploy Hyperlane

Hyperlane is designed to be deployed to new chains by anyone, at any time. Read on to learn how to deploy Hyperlane to your favorite EVM chain.

## Overview

This tutorial is intended for users who want to deploy Hyperlane to a new EVM chain.

By the end of this guide you will have deployed and configured the Hyperlane [messaging-api](../apis/messaging-api/ "mention"), allowing developers to send interchain messages to and from your chain.

At a high level, deploying Hyperlane requires the following actions:



[#1.-setup-keys](deploy-hyperlane.md#1.-setup-keys "mention") that you will use to deploy contracts and run validators and relayers

[#2.-deploy-contracts](deploy-hyperlane.md#2.-deploy-contracts "mention") to the local chain and to every remote chain with which the local chain will be able to send and receive messages.

[#3.-run-validators](deploy-hyperlane.md#3.-run-validators "mention") on your local chain, to provide the signatures needed for the [sovereign-consensus](../protocol/sovereign-consensus/ "mention") you deployed in step (2)

[#4.-run-relayers](deploy-hyperlane.md#4.-run-relayers "mention") for the local chain and for every remote chain with which the local chain will be able to send and receive messages.

[#5.-send-test-messages](deploy-hyperlane.md#5.-send-test-messages "mention") to confirm that your relayers able to deliver messages from and to each pair of chains



## 1. Setup keys

In order to complete this guide, you first need to set up the following:

* **A deployer key**: a 32 byte hexadecimal private key that is funded on all the chains on which we need to deploy contracts.
* **Validator addresses**: a list of validator addresses for your local chain are needed to configure the [multisig-ism.md](../protocol/sovereign-consensus/multisig-ism.md "mention")s. The configuration is a list of `n` local validator addresses and the threshold `m`, the minimum number of validators needed to verify outbound messages from the local chain.
* **Relayer keys**: You'll be running one relayer for the local chain and each of the remote chains. Each relayer instance must be configured with a key that has a balance on all the other chains. Use the chains' faucet to get tokens if they are testnets, for the core chains, you can find a list of faucets under [token-sources-and-faucets.md](../resources/token-sources-and-faucets.md "mention")

{% hint style="info" %}
For instructions on how to generate keys, see [agent-keys](../operators/agent-keys/ "mention"). Your deployer key **must** be a hexadecimal key, while validator and relayer keys can be hexadecimal or AWS KMS.
{% endhint %}

{% content-ref url="../operators/agent-keys/" %}
[agent-keys](../operators/agent-keys/)
{% endcontent-ref %}

{% hint style="warning" %}
**Do not proceed to the next section until you have set up deployer, validator, and relayer keys.**
{% endhint %}

##

## 2. Deploy contracts

### Overview

In this step we will be deploying Hyperlane smart contracts to the local and remote chains.

On the local chain, we will deploy:

* The core contracts, including a [messaging.md](../protocol/messaging.md "mention") that can be used to send and receive messages

On all chains, we will deploy:

* A [multisig-ism.md](../protocol/sovereign-consensus/multisig-ism.md "mention") that can be used to verify inbound messages
* An [interchain-gas-paymasters.md](../build-with-hyperlane/guides/paying-for-interchain-gas/interchain-gas-paymasters.md "mention"), which can be used to pay our relayer for delivering interchain messages
* A `TestRecipient`, which we will send messages to, in order to test that everything is working correctly

### Setup

{% hint style="info" %}
If you have not yet set up your deployer, validator, and relayer keys, see [#1.-setup-keys](deploy-hyperlane.md#1.-setup-keys "mention")
{% endhint %}

First, set up the `hyperlane-deploy` repo. This repo contain scripts to deploy Hyperlane contracts. You will need to install [`yarn`](https://yarnpkg.com/getting-started/install) if you haven't already.

```bash
git clone git@github.com:hyperlane-xyz/hyperlane-deploy.git
cd hyperlane-deploy
yarn install
```

Next, add a [`ChainMetadata`](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/typescript/sdk/src/consts/chainMetadata.ts#L21) entry for your local chain to `hyperlane-deploy/config/chains.ts`. An example has been populated for you for [`anvil`](https://book.getfoundry.sh/anvil/). Any chains that already have Hyperlane deployments do not need to be configured here (see [domains.md](../resources/domains.md "mention") for the list of already supported chains).

```typescript
export const chains: ChainMap<ChainMetadata> = {
  // ----------- Your chains here -----------------
  anvil: {
    name: 'anvil',
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

Finally, add the `MultisigIsmConfig` entry for your local chain to `hyperlane-deploy/config/multisig_ism.ts`. An example with a single validator has been populated for you for `anvil`. Any chains that already have Hyperlane deployments do not need to be configured here (see [domains.md](../resources/domains.md "mention") for the list of already supported chains).

```typescript
export const multisigIsmConfig: ChainMap<MultisigIsmConfig> = {
  // ----------- Your chains here -----------------
  anvil: {
    threshold: 1,
    validators: [
      // Last anvil address
      '0xa0ee7a142d267c1f36714e4a8f75612f20a79720',
    ],
  },
};
```

### Deploy

You can then run `yarn ts-node scripts/deploy-hyperlane.ts` to deploy the Hyperlane contracts. You will need to provide the following arguments:

* `local`: The local chain on which Hyperlane is being deployed
* `remotes`: The chains with which 'local' will be able to send and receive messages
* `key`: A hexadecimal private key for transaction signing

An example deployment command to `anvil` that supports communication with `goerli` and `sepolia` is shown below:

```bash
DEBUG=hyperlane* yarn ts-node scripts/deploy-hyperlane.ts --local anvil \
  --remotes goerli sepolia \
  --key 0x6f0311f4a0722954c46050bb9f088c4890999e16b64ad02784d24b5fd6d09061
```

{% hint style="warning" %}
The deploy script will only accept chains for which configuration has been provided. If you do not see your desired chain in the list of choices, you may be missing config in `chains.ts` or `multisig_ism.ts`
{% endhint %}

## Verify

If everything ran successfully, congrats! You should see contract addresses written to `artifacts/addresses.json`, and agent config written to `artifacts/agent_config.json`

```bash
$ head -n 19 artifacts/addresses.json
{
  "anvil": {
    "validatorAnnounce": "0x177B784C94d85f6645a35BfD14175D44045e573f",
    "proxyAdmin": "0x5Dfb392D946d0F3b6af599705541050A6ca6A870",
    "mailbox": "0x74756B469390CAee600F332184895ACbf86C4396",
    "multisigIsm": "0x1dD5a3E037be5C46839192a5b83a260166751409",
    "testRecipient": "0x7672E92386B49717D32946214A10B1988542F660",
    "storageGasOracle": "0xbA033019Fe072beda2389259e05bEB042bAb8fF6",
    "interchainGasPaymaster": "0x9368C1f2B6BE2869018622a9aB43a5D8ED27Fba2",
    "defaultIsmInterchainGasPaymaster": "0xc1FD390F3aB9d0e2bb8394B0DeCE48D31fC44121"
  },

$  head -n 22 artifacts/agent_config.json
{
  "chains": {
    "anvil": {
      "name": "anvil",
      "domain": 31337,
      "addresses": {
        "mailbox": "0x74756B469390CAee600F332184895ACbf86C4396",
        "interchainGasPaymaster": "0x9368C1f2B6BE2869018622a9aB43a5D8ED27Fba2",
        "validatorAnnounce": "0x177B784C94d85f6645a35BfD14175D44045e573f"
      },
      "signer": null,
      "protocol": "ethereum",
      "finalityBlocks": 1,
      "connection": {
        "type": "http",
        "url": ""
      },
      "index": {
        "from": 17
      }
    }
  },
```

## 3. Run validators

Follow the [validators](../operators/validators/ "mention") guide to run validators for the [messaging.md](../protocol/messaging.md "mention")on your local chain. These validators will provide the security for messages sent **from** your chain **to** remote chains.

Include the agent config from the [#2.-deploy-contracts](deploy-hyperlane.md#2.-deploy-contracts "mention") step in `CONFIG_FILES`. If you're using Docker, you will need to mount the file into the container.

You should have already set up your validator keys in step [#1.-setup-keys](deploy-hyperlane.md#1.-setup-keys "mention"), so you can skip that part of the [validators](../operators/validators/ "mention") guide.

{% hint style="warning" %}
Make sure these validators match the addresses you provided when in your `MultisigIsmConfig`. Otherwise, the [multisig-ism.md](../protocol/sovereign-consensus/multisig-ism.md "mention")s you deployed in the previous step will not be able to verify messages sent from your chain.
{% endhint %}

{% content-ref url="../operators/validators/" %}
[validators](../operators/validators/)
{% endcontent-ref %}

## 4. Run relayers

Follow the [relayers](../operators/relayers/ "mention") guide to run a relayer for the local chain, and each of the remote chains. These relayers will deliver interchain messages sent between the local and remote chains.

Remember to set the `HYP_RELAYER_DESTINATIONCHAINNAMES` appropriately.

Include the agent config from the [#2.-deploy-contracts](deploy-hyperlane.md#2.-deploy-contracts "mention") step in `CONFIG_FILES`. If you're using Docker, you will need to mount the file into the container.

You should have already set up your relayer keys in step [#1.-setup-keys](deploy-hyperlane.md#1.-setup-keys "mention"), so you can skip that part of the [relayers](../operators/relayers/ "mention") guide.

{% content-ref url="../operators/relayers/" %}
[relayers](../operators/relayers/)
{% endcontent-ref %}

## 5. Send test messages

You can check everything is working correctly by sending a test message between each pair of chains.

You can run `yarn ts-node scripts/test-messages.ts` to send test messages. You will need to provide the following arguments:

* `chains`: Test messages will be sent between each pair of these chains
* `key`: A hexadecimal private key for transaction signing

An example command that tests that messages can be sent between any pair of `anvil`, `goerli`, and `sepolia` is shown below:

```bash
DEBUG=hyperlane* yarn ts-node scripts/test-messages.ts \
  --chains anvil goerli sepolia \
  --key 0x6f0311f4a0722954c46050bb9f088c4890999e16b64ad02784d24b5fd6d09061
```



If everything ran successfully, congrats! You should something similar to the following output:

```bash
$ yarn ts-node scripts/test-messages.ts --chains anvil goerli --key 0x6f0311f4a0722954c46050bb9f088c4890999e16b64ad02784d24b5fd6d09060
Sent message from anvil to 0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE on goerli with message ID 0x5ad21a8dcfe2cd91d3e59e26f2ef7f01f6ab1850ef5922233c7776eacff8d8b0
Sent message from goerli to 0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE on anvil with message ID 0x27f8fcf9151c7bcc50408b2ca1df027346740f0b40b8e516b04b4a09a6757f69
Message from anvil to goerli with ID 0x5ad21a8dcfe2cd91d3e59e26f2ef7f01f6ab1850ef5922233c7776eacff8d8b0 has not yet been delivered
Message from goerli to anvil with ID 0x27f8fcf9151c7bcc50408b2ca1df027346740f0b40b8e516b04b4a09a6757f69 has not yet been delivered
Message from anvil to goerli with ID 0x5ad21a8dcfe2cd91d3e59e26f2ef7f01f6ab1850ef5922233c7776eacff8d8b0 has not yet been delivered
Message from goerli to anvil with ID 0x27f8fcf9151c7bcc50408b2ca1df027346740f0b40b8e516b04b4a09a6757f69 was delivered
Message from anvil to goerli with ID 0x5ad21a8dcfe2cd91d3e59e26f2ef7f01f6ab1850ef5922233c7776eacff8d8b0 was delivered
Testing complete
```

If you've waited a while and messages still aren't being delivered, take a look at the origin chain relayer logs and reach out on [discord](https://discord.gg/hyperlane).
