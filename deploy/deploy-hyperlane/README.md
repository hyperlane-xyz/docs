---
description: >-
  Follow along these articles to learn how you can deploy Hyperlane to any smart
  contract environment of your choice.
---

# Deploy Hyperlane

Hyperlane is designed to be deployed to new chains by anyone, at any time. Read on to learn how to deploy Hyperlane to your favorite EVM chain.

## Overview

This tutorial is intended for users who want to deploy Hyperlane to a new EVM chain, so that interchain messages can be sent between that local chain and other remote chains with Hyperlane deployments.

At a high level, this requires the following actions:

1. Deploying the core smart contracts (`Mailbox, InterchainGasPaymaster`) to the local chain. This script will also deploy and configure an [Interchain Security Module](../../protocol/sovereign-consensus/#interchain-security-modules) on the local chain. Applications may use this ISM to verify interchain messages sent **to** the local chain.
2. Running one or more [validators](../../operators/validators/) for the local chain, to provide security for outgoing messages
3. Deploying and configuring [Interchain Security Modules](../../build-with-hyperlane/guides/receive-1.md#interchain-security-modules) on the remote chain(s). Applications will use these ISMs to verify interchain messages sent **from** the local chain.
4. Running a [relayer](../../operators/relayers/) for the local chain, to deliver interchain messages sent **from** the local chain **to** the remote chain(s).
5. Running a [relayer](../../operators/relayers/) for the remote chain(s), to deliver incoming messages sent **from** the remote chain(s) **to** the local chain.
6. Testing that messages can be sent from the local chain to each of the remote chains, and vice versa.&#x20;

## 1. Deploy the core smart contracts

First, set up the `hyperlane-deploy` repo. This repo contain scripts to deploy Hyperlane contracts. You will need  to install [`yarn`](https://yarnpkg.com/getting-started/install) and [`foundry`](https://github.com/foundry-rs/foundry#installation) if you haven't already.

```bash
git clone git@github.com:hyperlane-xyz/hyperlane-deploy.git
cd hyperlane-deploy
yarn install
git submodule init && git submodule update --remote
```

Next, add an empty entry for the local chain to `hyperlane-deploy/config/networks.json`, e.g.

```json
  "foochain": {
    // The Chain ID
    "id": 123456,
    // The address that will own all ownable contracts post-deployment.
    // Be sure to change this address for the chain you're deploying to.
    // For all other existing deployments, keep the owner as-is.
    // E.g. "0xfaD1C94469700833717Fa8a3017278BC1cA8031C"
    "owner": "<change me>",
    "contracts": {
      "proxyAdmin": "",
      "mailbox": "",
      "interchainGasPaymaster": "",
      "create2Factory": "",
      "testRecipient": ""
    }
  },
```

Leave all the entries for existing deployments untouched, e.g. those relating to `"mumbai"` or `"alfajores"`, etc.

You can then run the following command to deploy the core contracts to your chain.

<pre class="language-bash"><code class="lang-bash"># The name of the chain to deploy to. Used to configure the localDomain for the
# Mailbox contract.
export LOCAL=YOUR_CHAIN_NAME
# An RPC url for the chain to deploy to.
export RPC_URL=YOUR_CHAIN_RPC_URL
# Deployer private key, must have a balance to pay for gas
# Any of the following alternatives to --private-key will work
<strong># https://book.getfoundry.sh/reference/forge/forge-script#wallet-options---raw
</strong>export PRIVATE_KEY=YOUR_PRIVATE_KEY
# The comma separated name(s) of the chains to receive messages from.
# Used to configure the default MultisigIsm.
export REMOTES=alfajores,fuji,mumbai,bsctestnet,goerli,moonbasealpha,optimismgoerli,arbitrumgoerli

forge script scripts/DeployCore.s.sol --broadcast --rpc-url $RPC_URL \
    --private-key $PRIVATE_KEY
</code></pre>

{% hint style="warning" %}
If you have issues with the transaction submission, try any of the following:

* \- the `--slow` argument, as foundry may not properly account for fast-finality chains
* the `--legacy` argument if you see `Failed to get EIP-1559 fees` as foundry defaults to 1559 transactions
{% endhint %}

This script will write a partial Hyperlane agent config to `hyperlane-deploy/config/$LOCAL_agent_config.json`, which will be used in the following step.

Deployed contract addresses will be written to `hyperlane-deploy/config/networks.json`

## 2. Run validators

{% hint style="warning" %}
When using the agent config, the numerical fields `index.from` and `domain` must be converted to strings by surrounding them with quotes.
{% endhint %}

Follow the [Validators guide](../../operators/validators/) to run validators for the mailbox on your chain. Include the agent config from [#1.-deploy-the-core-smart-contracts](./#1.-deploy-the-core-smart-contracts "mention") in `CONFIG_FILES`. If you're using Docker, you will need to mount the file into the container.

Make sure to collect the validator addresses for use in the next step.

## 3. Deploy remote ISMs

Using the validator addresses from [#2.-run-validators](./#2.-run-validators "mention"), add an entry to `hyperlane-deploy/config/multisig_ism.json` for the local chain.

This config will be used to deploy a `MultisigIsm` to each remote chain that you'd like to be able to send messages **to**. Applications will be able to use this ISM to verify interchain messages sent **from** the local chain.

{% hint style="warning" %}
Applications using this ISM will only be able to verify messages sent **from** the chains specified in the `REMOTES` env var.
{% endhint %}

```bash
# Be sure to deploy a MultisigIsm to each chain that you'd like to be able to
# send messages to.

# This address will wind up owning the MultisigIsm after it's deployed.
export OWNER=0x1234
# The private key that will be used to deploy the contracts. Does not have any
# permissions post-deployment, any key with a balance will do.
# An RPC url for the chain to deploy to.
export RPC_URL=YOUR_CHAIN_RPC_URL
# Deployer private key, must have a balance to pay for gas
# Any of the following alternatives to --private-key will work
# https://book.getfoundry.sh/reference/forge/forge-script#wallet-options---raw
export PRIVATE_KEY=YOUR_PRIVATE_KEY
# The comma separated name(s) of the chain(s) to receive messages from.
# Probably just the name of your chain. 
export REMOTES=YOUR_CHAIN_NAME

forge script scripts/DeployMultisigIsm.s.sol --broadcast --rpc-url $RPC_URL \
    --private-key $PRIVATE_KEY
```

You should see the following output. Save the contract addresses for future use. Applications will be able to use the `MultisigIsm` to verify messages sent from your chain. You will use the `TestRecipient` contract to verify that everything is working properly in step [#6.-send-test-messages](./#6.-send-test-messages "mention").

```bash
[⠒] Compiling...
[⠔] Compiling 2 files with 0.8.17
[⠒] Solc 0.8.17 finished in 1.36s
Compiler run successful
Script ran successfully.

== Logs ==
  MultisigIsm deployed at address 0x5307B8c7A4e8E992ea47126A2F974e65bc43E6e0
  TestRecipient deployed at address 0x2952EAB89729522252249d08883449e7CaD21326
  InterchainGasPaymaster deployed at address 0xBC3cFeca7Df5A45d61BC60E7898E63670e1654aE
```

This config will also deploy an `InterchainGasPaymaster` that you will need to use when you  [deploy-warp-route](../deploy-warp-route/ "mention")

## 4. Run a relayer for the local chain

Similar to [#2.-run-validators](./#2.-run-validators "mention"), follow the [relayers](../../operators/relayers/ "mention") instructions to run a relayer for the local chain by including the agent config in `CONFIG_FILES`

This relayer will deliver messages sent **from** the local chain **to** each of the remote chains. Remember to set the `HYP_RELAYER_DESTINATIONCHAINNAMES`for your supported remotes appropriately.\
\
You will need the validator addresses and S3 bucket names/regions from [#2.-run-validators](./#2.-run-validators "mention")in order to properly configure the relayer.

## 5. Run relayer(s) for the remote chain(s)

Reversely, follow the [relayers](../../operators/relayers/ "mention") instructions to run a relayer for each of the remote chains.

These relayers will deliver messages sent **from** the remote chains **to** the local chain. Remember to set the `HYP_RELAYER_DESTINATIONCHAINNAMES`for your supported remotes appropriately.

You can reference the validator addresses and S3 bucket names/regions needed to configure the relayer in either of the following files: [testnet](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/typescript/infra/config/environments/testnet3/validators.ts), [mainnet](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/typescript/infra/config/environments/mainnet2/validators.ts).

## 6. Send test messages

You can check everything is working correctly by sending a test message between each pair of chains.

This script will log the message ID and a link to the [message explorer](https://explorer.hyperlane.xyz/).

{% hint style="warning" %}
The explorer may not properly display your message, as it was sent to a chain that the explorer does not know about.
{% endhint %}

<pre class="language-bash"><code class="lang-bash"># An RPC url for the origin chain
export RPC_URL=YOUR_CHAIN_RPC_URL
# Sender private key, must have a balance to pay for gas
# Any of the following alternatives to --private-key will work
# https://book.getfoundry.sh/reference/forge/forge-script#wallet-options---raw
export PRIVATE_KEY=YOUR_PRIVATE_KEY
# The name of the chain to send the message from.
export ORIGIN=ORIGIN_CHAIN_NAME
# The name of the chain to send the message to.
export DESTINATION=DESTINATION_CHAIN_NAME
# The address of the contract receiving the message.
# If sending to your chain, use the address of the TestRecipient you deployed
# in step 1.
# If sending to a remote chain, use the address of the corresponding TestRecipient
# from step 3.
export RECIPIENT=TEST_RECIPIENT_ADDRESS
# The name of the chain to send the message to, e.g. "hello world"
export BODY=BODY

<strong>forge script scripts/SendTestMessage.s.sol --broadcast --rpc-url $RPC_URL \
</strong>  --private-key $PRIVATE_KEY
</code></pre>

If your message is not showing up in the explorer, you can check its delivery status by running the following script:

```bash
# The message ID of the message you sent.
export MESSAGE_ID=YOUR_MESSAGE_ID
# An RPC url for the destination chain.
export RPC_URL=_RPC_URL
# The name of the chain the message was sent to.
export DESTINATION=DESTINATION_CHAIN_NAME

forge script scripts/CheckMessage.s.sol --rpc-url $RPC_URL
```
