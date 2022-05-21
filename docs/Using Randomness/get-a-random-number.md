---
layout: nodes.liquid
section: smartContract
date: Last Modified
title: "Get a Random Number"
permalink: "docs/get-a-random-number/"
whatsnext: {"API Reference":"/docs/chainlink-vrf-api-reference/", "Contract Addresses":"/docs/vrf-contracts/"}
metadata: 
  description: "How to generate a random number inside a smart contract using Chainlink VRF."
  image: 
    0: "/files/OpenGraph_V3.png"
---
This page explains how to get a random number inside a smart contract using Chainlink VRF.

# Random Number Consumer

Chainlink VRF follows the [Request & Receive Data](../request-and-receive-data/) cycle. To consume randomness, your contract should inherit from <a href="https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.6/VRFConsumerBase.sol" target="_blank">`VRFConsumerBase`</a> and define two required functions

1. `requestRandomness`, which makes the initial request for randomness.
2. `fulfillRandomness`, which is the function that receives and does something with verified randomness.

The contract should own enough LINK to pay the specified fee. The beginner walkthrough explains how to [fund your contract](../fund-your-contract/).

Note, the below values have to be configured correctly for VRF requests to work. You can find the respective values for your network in the [VRF Contracts page](../vrf-contracts).
- `LINK Token` - LINK token address on the corresponding network (Ethereum, Polygon, BSC, etc)
- `VRF Coordinator` - address of the Chainlink VRF Coordinator
- `Key Hash` - public key against which randomness is generated
- `Fee` - fee required to fulfill a VRF request

> 🚧 Security Considerations
>
> Be sure to look your contract over with [these security considerations](../vrf-security-considerations/) in mind!

>❗️ Remember to fund your contract with LINK!
>
> Requesting randomness will fail unless your deployed contract has enough LINK to pay for it. **Learn how to [Acquire testnet LINK](../acquire-link/) and [Fund your contract](../fund-your-contract/)**.

<div class="remix-callout">
    <a href="https://remix.ethereum.org/#version=soljson-v0.6.6+commit.6c089d02.js&optimize=false&evmVersion=null&gist=f47e4eae5f2ffa7868ef4ecd5bda9044&runs=200" target="_blank" class="cl-button--ghost solidity-tracked">Deploy this contract using Remix ↗</a>
    <a href="../deploy-your-first-contract/" title="">What is Remix?</a>
</div>

```solidity Kovan

pragma solidity 0.6.6;

import "@chainlink/contracts/src/v0.6/VRFConsumerBase.sol";

/**
 * THIS IS AN EXAMPLE CONTRACT WHICH USES HARDCODED VALUES FOR CLARITY.
 * PLEASE DO NOT USE THIS CODE IN PRODUCTION.
 */
contract RandomNumberConsumer is VRFConsumerBase {
    
    bytes32 internal keyHash;
    uint256 internal fee;
    
    uint256 public randomResult;
    
    /**
     * Constructor inherits VRFConsumerBase
     * 
     * Network: Kovan
     * Chainlink VRF Coordinator address: 0xdD3782915140c8f3b190B5D67eAc6dc5760C46E9
     * LINK token address:                0xa36085F69e2889c224210F603D836748e7dC0088
     * Key Hash: 0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4
     */
    constructor() 
        VRFConsumerBase(
            0xdD3782915140c8f3b190B5D67eAc6dc5760C46E9, // VRF Coordinator
            0xa36085F69e2889c224210F603D836748e7dC0088  // LINK Token
        ) public
    {
        keyHash = 0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4;
        fee = 0.1 * 10 ** 18; // 0.1 LINK (Varies by network)
    }
    
    /** 
     * Requests randomness 
     */
    function getRandomNumber() public returns (bytes32 requestId) {
        require(LINK.balanceOf(address(this)) >= fee, "Not enough LINK - fill contract with faucet");
        return requestRandomness(keyHash, fee);
    }

    /**
     * Callback function used by VRF Coordinator
     */
    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = randomness;
    }

    // function withdrawLink() external {} - Implement a withdraw function to avoid locking your LINK in the contract
}
```

> 🚧 Maximum Gas for Callback
>
> If your `fulfillRandomness` function uses more than 200k gas, the transaction will fail.

## Getting More Randomness

If you are looking for how to turn a single result into multiple random numbers, check out our guide on [Randomness Expansion](../chainlink-vrf-best-practices/#getting-multiple-random-numbers).