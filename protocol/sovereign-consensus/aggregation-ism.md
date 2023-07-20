---
description: Aggregate security from multiple ISMs
---

# Aggregation ISM

Developers can use an `AggregationISM` to combine security from multiple ISMs. Simply put, an `AggregationISM` requires that `m` of `n` ISMs verify a particular interchain message.

## Interface

`AggregationISMs` must implement the `IAggregationIsm` interface.

```solidity
// SPDX-License-Identifier: MIT OR Apache-2.0
pragma solidity >=0.6.11;

import {IInterchainSecurityModule} from "../IInterchainSecurityModule.sol";

interface IAggregationIsm is IInterchainSecurityModule {
    /**
     * @notice Returns the set of modules responsible for verifying _message
     * and the number of modules that must verify
     * @dev Can change based on the content of _message
     * @param _message Hyperlane formatted interchain message
     * @return modules The array of ISM addresses
     * @return threshold The number of modules needed to verify
     */
    function modulesAndThreshold(bytes calldata _message)
        external
        view
        returns (address[] memory modules, uint8 threshold);
}

```

## Configure

The hyperlane-monorepo contains an `AggregationISM` implementation that application developers can deploy off-the-shelf, specifying their desired configuration.

Developers can configure, for each origin chain, a set of `n` ISMs, and the number of ISMs needed to verify a message.

`AggregationISMs` can aggregate the security of any ISMs. For example, users can deploy a [multisig-ism.md](multisig-ism.md "mention") with their own validator set, and deploy an `AggregationISM` that aggregates that ISM with the Hyperlane default ISM.

The [hyperlane-deploy repo](https://github.com/hyperlane-xyz/hyperlane-deploy) contains the tooling and instructions needed to deploy and configure an `AggregationISM`.

## Customize

The hyperlane-monorepo contains an abstract `AggregationISM` implementation that application developers can fork.

Developers simply need to implement the `modulesAndThreshold()` function.

By creating a custom implementation, application developers can tailor the security provided by a `AggregationISM` to the needs of their application.

For example, a custom implementation could require that low value messages be verified by a [multisig-ism.md](multisig-ism.md "mention"), and require that high value messages _also_ be verified by a [wormhole-ism.md](wormhole-ism.md "mention").
