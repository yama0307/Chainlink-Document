---
layout: nodes.liquid
section: smartContract
date: Last Modified
title: "Multi-Variable Responses"
permalink: "docs/multi-variable-responses/"
whatsnext: {"Make an Existing Job Request":"/docs/existing-job-request/", "API Reference":"/docs/chainlink-framework/", "Contract Addresses":"/docs/decentralized-oracles-ethereum-mainnet/", "Large Responses": "/docs/large-responses/"}
---
This page explains how to make an HTTP GET request to an external API from a smart contract, using Chainlink's [Request & Receive Data](../request-and-receive-data/) cycle and then receive multiple responses.

This is known as **multi-variable** or **multi-word** responses.

> ⚠️ NOTE
>
> This feature is available as of Chainlink `0.10.10` but can only be customized with different URLs and responses on the node operator side. In `0.10.11` generic jobs will be available for solidity as well. Please see [Make an API Call](../make-a-http-get-request/) for more information on making API calls.

# MultiWord

To consume an API with multiple responses, your contract should inherit from [ChainlinkClient](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.8/ChainlinkClient.sol). This contract exposes a struct called `Chainlink.Request`, which your contract should use to build the API request. The request should include the oracle address, the job id, the fee, adapter parameters, and the callback function signature.

>❗️ Remember to fund your contract with LINK!
>
> Making a GET request will fail unless your deployed contract has enough LINK to pay for it. **Learn how to [Acquire testnet LINK](../acquire-link/) and [Fund your contract](../fund-your-contract/)**.
>

<div class="remix-callout">
    <a href="https://remix.ethereum.org/#url=https://docs.chain.link/samples/APIRequests/MultiWordConsumer.sol" target="_blank" class="cl-button--ghost solidity-tracked">Deploy a Multi-Word Contract Example in Remix ↗</a>
    <a href="../deploy-your-first-contract/" title="">What is Remix?</a>
</div>

```solidity Kovan
{% include samples/APIRequests/MultiWordConsumer.sol %}
```

> 📘 Note
>
> The job spec for the Chainlink node in this example can be [found here](../example-job-spec-multi-word/).

If the LINK address for targeted blockchain is not [publicly available](../link-token-contracts/) yet, replace [setPublicChainlinkToken(/)](../chainlink-framework/#setpublicchainlinktoken) with [setChainlinkToken(_address)](../chainlink-framework/#setchainlinktoken) in the constructor, where `_address` is a corresponding LINK token contract.

# Choosing an Oracle and JobId

The `oracle` keyword refers to a specific Chainlink node that a contract makes an API call from, and the `specId` refers to a specific job for that node to run. Each job is unique and returns different types of data.

For example, a job that returns a `bytes32` variable from an API would have a different `specId` than a job that retrieved the same data, but in the form of a `uint256` variable.

[market.link](https://market.link/) provides a searchable catalogue of Oracles, Jobs and their subsequent return types.

# Choosing an Oracle Job without specifying the URL

If your contract is calling a public API endpoint, an Oracle job may already exist for it. If so, it could mean you do not need to add the URL, or other adapter parameters into the request, since the job already configured to return the desired data. This makes your smart contract code more succinct. To see an example of a contract using an existing job which calls the [CoinGecko API]("https://www.coingecko.com/en/api#explore-api"), see [Make an Existing Job Request](../existing-job-request/).

For more information about the functions in `ChainlinkClient`, visit [ChainlinkClient API Reference](../chainlink-framework/).
