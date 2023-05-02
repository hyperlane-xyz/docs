---
description: Configure which messages to relay, and which to ignore
---

# Message filtering

By default, the relayer will attempt to deliver messages sent from its origin chain to any of the configured destination chains.

Relayers may want to further filter the messages they attempt to deliver. For example, someone building an interchain app may want to run a relayer that only delivers messages sent to that application. Similarly, some relayers may wish to only relay messages to a subset of available chains.

Relayers may **optionally** filter the messages they deliver by setting the `HYP_BASE_WHITELIST` or `HYP_BASE_BLACKLIST` environment variables. See also the configuration reference's[#whitelist](../../agent-configuration/configuration-reference.md#whitelist "mention") and[#blacklist](../../agent-configuration/configuration-reference.md#blacklist "mention") sections.

These environment variables are stringified JSON objects with the following format:

```typescript
// A list of matching rules. A message matches if any of the list
// elements matches the message.
type MatchingList = Array<ListElement>;

// Matches a message if any of the provided values matches.
interface ListElement {
    originDomain?: NumericFilter
    senderAddress?: HashFilter
    destinationDomain?: NumericFilter
    recipientAddress?: HashFilter
}

type NumericFilter = Wildcard | U32 | Array<U32>;
type HashFilter = Wildcard | H256 | Array<H256>;

// 32-bit unsigned integer
type U32 = number | string;
// 256-bit hash (can also be less) encoded as hex
type H256 = string;
// Matches anything
type Wildcard = "*";
```

Both the whitelist and blacklists have "any" semantics. In other words, the relayer will deliver messages that match _any_ of the whitelist filters, and ignore messages that match _any_ of the blacklist filters.

For example, the following config used as a whitelist will ensure the relayer will relay any messages sent to Ethereum, any messages sent from address `0xca7f632e91B592178D83A70B404f398c0a51581F` to either Celo or Avalanche, and any messages sent to address `0xca7f632e91B592178D83A70B404f398c0a51581F` on Arbitrum or Optimism.

```json
[
    {
        senderAddress: "*",
        destinationDomain: ["1"],
        recipientAddress: "*"
    },
    {
        senderAddress: "0xca7f632e91B592178D83A70B404f398c0a51581F",
        destinationDomain: ["42220", "43114"],
        recipientAddress: "*"
    },
    {
        senderAddress: "*",
        destinationDomain: ["42161", "420"],
        recipientAddress: "0xca7f632e91B592178D83A70B404f398c0a51581F"
    }
]
```

As an env string, a valid config may look like&#x20;

```bash
HYP_BASE_WHITELIST='[{"senderAddress":"*","destinationDomain":["1"],"recipientAddress":"*"},{"senderAddress":"0xca7f632e91B592178D83A70B404f398c0a51581F","destinationDomain":["42220","43114"],"recipientAddress":"*"},{"senderAddress":"*","destinationDomain":["42161","420"],"recipientAddress":"0xca7f632e91B592178D83A70B404f398c0a51581F"}]'
```

The blacklist supersedes the whitelist, i.e. if a message matches both the whitelist _and_ the blacklist, it will not be delivered.

### Next

Now you are ready to relay messages!

{% content-ref url="../start-validating.md" %}
[start-validating.md](../start-validating.md)
{% endcontent-ref %}
