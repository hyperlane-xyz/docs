---
description: Manually pay a relayer to deliver messages
---

# Pay for interchain gas

{% hint style="info" %}
Read up on [interchain-gas-payments.md](../../protocol/interchain-gas-payments.md "mention") and the [interchain-gas-paymaster-api.md](../../apis-and-sdks/interchain-gas-paymaster-api.md "mention")
{% endhint %}

This guide will show you how to make a manual gas payment.

Typically, the smart contract that sent the interchain message will [paying-for-interchain-gas.md](paying-for-interchain-gas.md "mention").&#x20;

In some cases (e.g. [quickstarts](../quickstarts/ "mention")) it may be useful to make gas payments manually.

To make a manual gas payment, you will need

- The `$MESSAGE_ID` of the interchain message you are paying for
- The `$DESTINATION_DOMAIN` of the interchain message you are paying for
- The `$GAS_AMOUNT` that you need in order to deliver your message on the destination chain

### Finding your message ID

If you have the hash of the transaction that sent your interchain message, you can use a block explorer to find the message ID. Navigate to the transaction in a block explorer, open the "Logs" tab, and find the `DispatchId` log. The "Topic 1" is your message ID. Use the dropdown to select "Hex", and use this value. For example:

<figure><img src="../../.gitbook/assets/Screen Shot 2023-01-26 at 10.47.06 AM.png" alt=""><figcaption><p>Finding the message ID from the <code>DispatchId</code> log</p></figcaption></figure>

### Paying for interchain gas

{% tabs %}
{% tab title="Using Metamask" %}
**Getting the interchain gas payment quote**

1. Navigate to the [#defaultisminterchaingaspaymaster](../../resources/addresses/#defaultisminterchaingaspaymaster "mention") contract page on [Etherscan](https://etherscan.io/address/0x56f52c0A1ddcD557285f7CBc782D3d83096CE1Cc) (or its equivalent if you're sending from a non-Ethereum chain).
2. Under the `Contract` tab, select `Read Contract`.
3. Expand the `quoteGasPayment` function.
4. For destination domain, enter `$DESTINATION_DOMAIN`.
5. For gas amount, enter `$GAS_AMOUNT`.
6. Click `Query` and make note of the amount returned as `$GAS_PAYMENT_QUOTE`. For example, at the time of writing, the quote is `1` wei.

<figure><img src="../../.gitbook/assets/Screen Shot 2023-01-30 at 2.05.19 PM.png" alt=""><figcaption></figcaption></figure>

**Make the interchain gas payment**

1.  Still on the `DefaultIsmInterchainGasPaymaster` contract page on Etherscan, select `Write Contract`.
2.  Click on the `Connect to Web3` button to connect your Wallet (i.e. Metamask). Make sure that you are on the correct network.
3.  Expand the `payForGas` function.
4.  For the payable amount, Etherscan expects an amount quoted in ether, while our `$GAS_PAYMENT_QUOTE` is in wei. To convert from wei to ether, input the amount `$GAS_PAYMENT_QUOTE`, which is in wei, into [https://eth-converter.com/](https://eth-converter.com/) and copy the ether amount. Use this ether amount as the payable amount.
5.  For the message ID, input your `$MESSAGE_ID`.
6.  For the destination domain, input your `$DESTINATION_DOMAIN`.
7.  For gas amount, enter `$GAS_AMOUNT`.
8.  For the refund address, input the address of the account you will sign the transaction with. This will receive a potential refund if you overpay for interchain gas.
9.  Click "Write" and submit the transaction via your wallet/Metamask.

        <figure><img src="../../.gitbook/assets/Screen Shot 2023-01-30 at 2.05.42 PM.png" alt=""><figcaption></figcaption></figure>

    {% endtab %}

{% tab title="Using cast" %}
**Getting the interchain gas payment quote**

First, get a quote for how much your gas payment will cost, and save this in an environment variable called `$GAS_PAYMENT_QUOTE`:

{% code overflow="wrap" %}

```shell
cast call $IGP_ADDRESS "quoteGasPayment(uint32,uint256)" $DESTINATION_DOMAIN $GAS_AMOUNT --rpc-url $RPC_URL
```

{% endcode %}

**Make the interchain gas payment**

Now, we can call `payGasFor`, and we supply the gas payment quote as value in the transaction. The final parameter, `$MY_ADDRESS`, is the address of the account whose private key you're signing with. This address will be refunded any overpayment.

<pre class="language-shell" data-overflow="wrap"><code class="lang-shell"><strong>cast send $IGP_ADDRESS "payForGas(bytes32,uint32,uint256,address)" $MESSAGE_ID $DESTINATION_DOMAIN $GAS_AMOUNT $MY_ADDRESS --rpc-url $RPC_URL
</strong><strong>--private-key $PRIVATE_KEY --value $GAS_PAYMENT_QUOTE
</strong></code></pre>

{% endtab %}
{% endtabs %}

### Confirm delivery

After you've paid for interchain gas, you should be able to see a corresponding transaction delivering your message on the destination chain. You can watch for this transaction on [Hyperlane's Message Explorer](https://explorer.hyperlane.xyz/) by entering the transaction hash or the sender/recipient address in the input field.\
\
You can see an example message delivery transaction [here](https://explorer.hyperlane.xyz/message/24275).

<figure><img src="../../.gitbook/assets/Test Message Sent -- Hyperlane Explorer.png" alt=""><figcaption><p>This transaction sent a "Hello World" message from Goerli to Alfajores</p></figcaption></figure>

{% hint style="warning" %}
Message not delivered? See [observability.md](../explorer/observability.md "mention") for tips and tricks
{% endhint %}
