---
description: Leverage security from the Wormhole guardians
---

# Wormhole ISM

{% hint style="warning" %}
`WormholeISM` is coming soon and is not yet implemented. This page is shown for informational purposes only. Details may change as the design matures.
{% endhint %}

The `WormholeISM` encodes the security model used by [Wormhole's](https://wormhole.com/) interchain communication protocol.

This ISM requires that 13 of the 19 [Wormhole guardians](https://wormhole.com/network/#guardians) attest to the validity of a Hyperlane message.

## Compose

Developers can compose a Wormhole ISM with a [multisig-ism.md](multisig-ism.md "mention")using an [aggregation-ism-1.md](aggregation-ism-1.md "mention") to require that m of n Hyperlane validators **and** 13-of-19 Wormhole guardians verify a message.
