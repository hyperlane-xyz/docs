---
description: Verify messages using validator signatures
---

# Multisig ISM

The `MultisigISM` is one of the most commonly used ISM types. These ISMs verify that `m` of `n` [validators.md](../agents/validators.md "mention") have attested to the validity of a particular interchain message.

## Interface

`MultisigISMs` must implement the `IMultisigIsm` interface.

```solidity
// SPDX-License-Identifier: MIT OR Apache-2.0
pragma solidity >=0.6.0;

import {IInterchainSecurityModule} from "./IInterchainSecurityModule.sol";

interface IMultisigIsm is IInterchainSecurityModule {
    /**
     * @notice Returns the set of validators responsible for verifying _message
     * and the number of signatures required
     * @dev Can change based on the content of _message
     * @param _message Hyperlane formatted interchain message
     * @return validators The array of validator addresses
     * @return threshold The number of validator signatures needed
     */
    function validatorsAndThreshold(bytes calldata _message)
        external
        view
        returns (address[] memory validators, uint8 threshold);
}
```

## Configure

The hyperlane-monorepo contains [`MultisigISM` implementations](https://github.com/hyperlane-xyz/hyperlane-monorepo/tree/main/solidity/contracts/isms/multisig) (including a [legacy](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/isms/multisig/LegacyMultisigIsm.sol) version and more gas-efficient [versions](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/isms/multisig/StaticMultisigIsm.sol) deployable via factories) that application developers can deploy off-the-shelf, specifying their desired configuration.

Developers can configure, for each origin chain, a set of `n` [validators.md](../agents/validators.md "mention"), and the number of validator signatures (`m`) needed to verify a message.

Validator signatures are **not** specific to an ISM. In other words, developers can configure their `MultisigISM` to use **any** validator that's running on Hyperlane and it will "just work".

The [hyperlane-deploy repo](https://github.com/hyperlane-xyz/hyperlane-deploy) contains the tooling and instructions needed to deploy and configure a `MultisigISM`.

## Customize

The hyperlane-monorepo contains an abstract `MultisigISM` implementation that application developers can fork.

Developers simply need to implement the `validatorsAndThreshold()` function.

By creating a custom implementation, application developers can tailor the security provided by a `MultisigISM` to the needs of their application.

For example, a custom implementation could vary the number of signatures required, based on the content of the message being verified.
