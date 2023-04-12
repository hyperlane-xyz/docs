---
description: Everything you need to start running a validator
---

# Validators

Hyperlane [validators](../../protocol/agents/validators.md) are stateless, do not submit transactions, and are not networked with other validators. Hyperlane validators are run on a per-origin-chain basis, and these instructions are written for a single chain.

```mermaid
%%{ init: {
  "theme": "neutral",
  "themeVariables": {
    "mainBkg": "#025AA1",
    "textColor": "white",
    "clusterBkg": "white"
  },
  "themeCSS": ".edgeLabel { color: black }",
  "flowchart": {"useMaxWidth": "true" }
}}%%

flowchart TB
    V(("Validator(s)"))
    Relayer((Relayer))

    subgraph Origin
      Sender
      M_O[(Mailbox)]
      POS[Proof of Stake]

      Sender -- "dispatch(destination, recipient, body)" --> M_O
    end

    subgraph Cloud
      aws[(Metadata\nDatabase)]
    end

    M_O -. "indexing" .-> Relayer
    M_O -. "indexing" .-> V
    POS == "staking" === V

    V -- "signatures" --> aws

    subgraph Destination
      Recipient
      M_D[(Mailbox)]
      ISM[MultisigISM]

      M_D -- "verify(metadata, [origin, sender, body])" --> ISM
      M_D -- "handle(origin, sender, body)" --> Recipient
      ISM -. "interchainSecurityModule()" .- Recipient
    end

    Relayer -- "process()" --> M_D

    aws -. "metadata" .-> Relayer
    aws -. "moduleType()" .- ISM

    POS -. "validators()" .- ISM

    click ISM https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/isms/MultisigIsm.sol

    style Sender fill:#efab17
    style Recipient fill:#efab17
```

Running a validator simply requires the following:

#### An RPC node

Validators make simple view calls to read merkle roots from the [`Mailbox`](../../protocol/messaging.md) contract on the chain they are validating for.

{% hint style="warning" %}
Operating a validator for Polygon mainnet requires access to an archive node. This is because validators should only sign roots once they've been finalized, and Polygon requires 256 block confirmations to achieve finality.
{% endhint %}

#### A secure signing key

Validators use this key to sign the `Mailbox's` latest merkle root. Securing this key is important. If it is compromised, attackers can attempt to falsify messages, causing the validator to be slashed.

The Hyperlane validator agent currently supports signing with AWS KMS keys that are accessed via API keys/secrets as well as hexadecimal plaintext keys for testing. See more under [agent-keys](../agent-keys/ "mention").

#### Publicly readable storage

Validators write their signatures off-chain to publicly accessible, highly available, storage, so that they can be aggregated by [relayers](../../protocol/agents/relayer.md).

The Hyperlane validator agent currently supports storing signatures on AWS S3 using the same AWS API key above, as well as storing signatures in the local filesystem for testing.

#### A machine to run on

Validators can compile the Rust binary themselves, or run a Docker image provided by Abacus Works. The binary is completely stateless and can be run using your favorite cloud service. You can even run multiple instances of them in different regions for high availability, as Hyperlane has no notion of "double signing".
