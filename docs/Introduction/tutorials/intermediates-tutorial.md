---
layout: nodes.liquid
section: smartContract
date: Last Modified
title: "Intermediates - Random Numbers"
permalink: "docs/intermediates-tutorial/"
excerpt: "Using Chainlink VRF"
whatsnext: {"Get a Random Number":"/docs/get-a-random-number/", "Advanced - API Calls":"/docs/advanced-tutorial/"}metadata: 
  title: "Intermediates Tutorial"
  description: "Learn how to use randomness in your smart contracts using Chainlink VRF."
  image: 
    0: "/files/2a242f1-link.png"
---

<p>
  https://www.youtube.com/watch?v=JqZWariqh5s
</p>

# Introduction

> 👍 Assumed knowledge
>
> This tutorial assumes some basic knowledge around Ethereum, and writing smart contracts. If you're brand new to smart contract development, we recommend working through our [Beginners Tutorial](../beginners-tutorial/) before this one.

Randomness is very difficult to generate on blockchains. The reason for this is because every node must come to the same conclusion, forming a consensus. There's no way to generate random numbers natively in smart contracts, which is unfortunate because they can be very useful for a wide range of applications. Fortunately, Chainlink provides [Chainlink VRF](../chainlink-vrf/), AKA Chainlink Verifiable Random Function.

If you've walked through the [Beginners Tutorial](../beginners-tutorial/), you'll know how to write smart contracts, use [Chainlink Price Feeds](../using-chainlink-reference-contracts/), and how to deploy a contract to a testnet. If not, head there and come back once you've finished.

In this tutorial, we go through:
- The Chainlink request & receive cycle
- Using the LINK token
- How to use request & receive with Chainlink Oracles
- Consuming random numbers with Chainlink VRF in smart contracts

# 1. Request & Receive

The previous tutorial went through how to consume Chainlink Price Feeds, which consists of reference data posted on-chain by oracles. This data is stored in a contract and can be referenced by consumers until the price is updated by the oracle.

Randomness, on the other hand, cannot be reference data. If the result of randomness is stored on-chain, any actor could see the value and predict the outcome. Instead, randomness must be requested from an oracle, which generates a number and a cryptographic proof then returns that result to the contract that requested it. This sequence is what's known as the [Request and Receive](../architecture-request-model/) cycle.

# 2. Using LINK

In return for providing this service of generating a random number, Oracles need to be paid in [LINK](../link-token-contracts/). This is paid by the contract that requests the randomness, and payment occurs during the request.
[block:callout]
{
  "type": "info",
  "title": "ERC-677 Token Standard",
  "body": "LINK conforms to the ERC-677 token standard, and extension of ERC-20. This standard is what enables data to be encoded in token transfers. This is integral to the Request and Receive cycle. <a href=\"https://github.com/ethereum/EIPs/issues/677\" target=\"_blank\">Click here</a> to learn more about ERC-677."
}
[/block]
# 3. Interacting with Chainlink Oracles

As mentioned in the previous tutorial, smart contracts have all the capabilities that wallets have, in that they are able to own and interact with tokens. The contract that requests randomness from Chainlink VRF must have a LINK balance equivalent to, or greater than the cost of making the request, in order to pay for it.

For example, if the current price of Chainlink VRF is 0.1 LINK, our contract must hold at least that much to pay for the request. Once the request transaction has completed, the oracle begins the process of generating the random number and sending the result back.

# 4. Using Chainlink VRF

To see a basic implementation of Chainlink VRF, see [Get a Random Number](../get-a-random-number/).

In this example, we'll create a contract with a Game of Thrones theme. It will request randomness from Chainlink VRF, the result of which it will transform into a number between 1 and 20, mimicking the rolling of a 20 sided dice. Each number represents a Game of Thrones house. So, if you land a 1, you are assigned house Targaryan, 2 is Lannister, and so on.

When rolling the dice, it will accept an `address` variable to track which address is assigned to each house.

The contract will have the following functions:
- `rollDice`: This submits a randomness request to Chainlink VRF
- `fulfillRandomness`: The function that is used by the Oracle to send the result back to
- `house`: To see the assigned house of an address
[block:callout]
{
  "type": "info",
  "title": "Open Full Contract",
  "body": "To jump straight to the entire implementation, you can <a href=\"https://remix.ethereum.org/#version=soljson-v0.6.7+commit.b8d736ae.js&optimize=false&evmVersion=null&gist=55c1263fcfc710f834aa38b7bbd21dc1\" target=\"_blank\" class=\"solidity-tracked\">open this contract in remix</a>."
}
[/block]
## 4a. Importing `VRFConsumerBase`

Chainlink maintains a <a href="https://github.com/smartcontractkit/chainlink/tree/develop/evm-contracts" target="_blank">library of contracts</a> that make consuming data from oracles easier. For Chainlink VRF, we use a contract called <a href="https://github.com/smartcontractkit/chainlink/blob/master/evm-contracts/src/v0.6/VRFConsumerBase.sol" target="_blank">`VRFConsumerBase`</a>, which needs to be imported and extended from.

```javascript
pragma solidity 0.6.6;

import "https://github.com/smartcontractkit/chainlink/blob/develop/evm-contracts/src/v0.6/VRFConsumerBase.sol";

contract VRFD20 is VRFConsumerBase {

}
```

## 4b. Contract variables

The contract will store a number of things. Firstly, it needs to store variables which tell the oracle what it is requesting. Each oracle job has a unique Key Hash, which is used to identify tasks that it should perform. The contract will store the Key Hash that identifies Chainlink VRF, and the fee amount, to use in the request.

```javascript
bytes32 private s_keyHash;
uint256 private s_fee;
```

These will be initialized in the constructor.

For the contract to keep track of addresses that roll the dice, the contract will need to use mappings. <a href="https://medium.com/upstate-interactive/mappings-in-solidity-explained-in-under-two-minutes-ecba88aff96e" target="_blank">Mappings</a> are unique `key => value` pair data structures that act like hash tables.

```javascript
mapping(bytes32 => address) private s_rollers;
mapping(address => uint256) private s_results;
```

- `s_rollers` stores a mapping between the `requestID` (returned when a request is made), and the address of the roller. This is so the contract can keep track of who to assign the result to when it comes back.
- `s_results` stores the roller, and the result of the dice roll.

## 4c. Initializing the contract

As mentioned, the fee and the key hash must be set on construction. To use `VRFConsumerBase` properly, we also need to pass certain values into its constructor.

```javascript
constructor(address vrfCoordinator, address link, bytes32 keyHash, uint256 fee)
    public
    VRFConsumerBase(vrfCoordinator, link)
{
    s_keyHash = keyHash;
    s_fee = fee;
}
```

As you can see, `VRFConsumerBase` needs to know the address of the vrfCoordinator, and the address of the LINK token. Both of which are [available in the docs](../vrf-contracts/).

## 4d. `rollDice` function

`rollDice` must do a few things:

1. It needs to check if the contract has enough LINK to pay the oracle.
2. Check if the roller has already rolled since each roller can only ever be assigned to a single house.
3. Request randomness
4. Store the `requestId` and roller address.
5. Emit an event to signal that the die is rolling.

```javascript
uint256 private constant ROLL_IN_PROGRESS = 42;

// ...
// { variables we've already written } 
// ...

event DiceRolled(bytes32 indexed requestId, address indexed roller);

/// ...
// { constructor } 
// ...

function rollDice(uint256 userProvidedSeed, address roller) public onlyOwner returns (bytes32 requestId) {
    require(LINK.balanceOf(address(this)) >= s_fee, "Not enough LINK to pay fee");
    require(s_results[roller] == 0, "Already rolled");
    requestId = requestRandomness(s_keyHash, s_fee, userProvidedSeed);
    s_rollers[requestId] = roller;
    s_results[roller] = ROLL_IN_PROGRESS;
    emit DiceRolled(requestId, roller);
}
```

Notice that we've added `ROLL_IN_PROGRESS` to signify that die has been rolled but the result has not yet returned, and a `DiceRolled` event to the contract.

## 4e. `fulfillRandomness` function

This is a special function defined within the `VRFConsumerBase` contract that ours extends from. It is the function that the coordinator sends the result back to, so we need to implement some functionality here to deal with the result.

It should:

1. Transform the result to a number between 1 and 20 inclusively.
2. Assign the transformed value to the address in the `s_results` mapping variable.
3. Emit a `DiceLanded` event.

```javascript
// ...

event DiceRolled(bytes32 indexed requestId, address indexed roller);
event DiceLanded(bytes32 indexed requestId, uint256 indexed result);

// ...

function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
    uint256 d20Value = randomness.mod(20).add(1);
    s_results[s_rollers[requestId]] = d20Value;
    emit DiceLanded(requestId, d20Value);
}
```

## 4f. `house` function

Finally, the `house` function returns the house of an address.

```javascript
function house(address player) public view returns (string memory) {
    require(s_results[player] != 0, "Dice not rolled");
    require(s_results[player] != ROLL_IN_PROGRESS, "Roll in progress");
    return getHouseName(s_results[player]);
}

function getHouseName(uint256 id) private pure returns (string memory) {
    string[20] memory houseNames = [
        "Targaryen",
        "Lannister",
        "Stark",
        "Tyrell",
        "Baratheon",
        "Martell",
        "Tully",
        "Bolton",
        "Greyjoy",
        "Arryn",
        "Frey",
        "Mormont",
        "Tarley",
        "Dayne",
        "Umber",
        "Valeryon",
        "Manderly",
        "Clegane",
        "Glover",
        "Karstark"
    ];
    return houseNames[id.sub(1)];
}
```

See the full contract in Remix! (We've added a few helper functions in there which should make using the contract easier and more flexible. Have a play around with it to understand all the internal workings).

<div class="remix-callout">
  <a href="https://remix.ethereum.org/#version=soljson-v0.6.7+commit.b8d736ae.js&optimize=false&evmVersion=null&gist=55c1263fcfc710f834aa38b7bbd21dc1" target="_blank" class="cl-button--ghost solidity-tracked">Deploy this contract using Remix ↗</a>
    <a href="../deploy-your-first-contract/" title="">What is Remix?</a>
</div>

# 5. Deployment

Time to compile and deploy the contract! If you don't know how to deploy a contract to the Kovan testnet from Remix, follow **[the Beginner Tutorial](/docs/beginners-tutorial)**.

This deployment is slightly different than the example from the beginners tutorial. In this tutorial, we have to pass in parameters to the constructor upon deployment.

Once compiled, you'll see a menu that looks like this in the deploy pane:
[block:image]
{
  "images": [
    {
      "image": [
        "/files/f6c0c2b-Screenshot_2020-12-18_at_16.23.19.png",
        "Screenshot 2020-12-18 at 16.23.19.png",
        796,
        304,
        "#343240"
      ]
    }
  ]
}
[/block]
Click the caret arrow on the right hand side of "Deploy" to expand the parameter fields, and paste the following values in:

- `0xdD3782915140c8f3b190B5D67eAc6dc5760C46E9`
- `0xa36085f69e2889c224210f603d836748e7dc0088 `
- `0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4`
- `100000000000000000`

These are the coordinator address, LINK address, key hash, and fee. Click deploy and use your Metamask account to confirm the transaction. 
[block:callout]
{
  "type": "info",
  "title": "Address, Key Hashes and more",
  "body": "For a full reference of the addresses, key hashes and fees for each network, see [VRF Contracts](../vrf-contracts/)."
}
[/block]
(Note: you should <a href="/docs/beginners-tutorial#7c-obtaining-testnet-eth" target="_blank">have some Kovan ETH in your Metamask account</a> to pay for the GAS).

Once deployed, the contract is almost ready to go! However, it can't request anything yet, since it doesn't own LINK. If we hit `rollDice` with no LINK, the transaction will revert.

# 6. Obtaining testnet LINK

Since the contract is on testnet, as with Kovan ETH, we don't need to purchase _real_ LINK. Testnet LINK can be obtained by requesting from a [faucet](../link-token-contracts/).

Use your Metamask address on the Kovan network to request LINK, then send 1 LINK to the contract address. This address can be found in Remix, under "Deployed Contracts" on the bottom left.

Note, you should add the corresponding LINK token to your MetaMask account first:
![metamask](/images/contract-devs/metamask-1.png)

If you enounter any issues, make sure to check you copied the address of the correct network:
![metamask](/images/contract-devs/metamask-2.png)

# 7. Rolling the Dice!

Opening the deployed contract tab in the bottom left, the function buttons are available. Find `rollDice` and click the caret to expand the parameter fields. Enter the seed (a series of characters of your choosing), and your Metamask address, and click roll!

Wait a few minutes for your transaction to confirm, and the response to be sent back. You can try getting your house by clicking the `house` function button with your address. Once the response has been sent back, you'll be assigned a Game of Thrones house!

# 8. Further Reading

Blog Post: <a href="https://blog.chain.link/random-number-generation-solidity/" target="_blank">Random Number Generation in Solidity</a>