---
id: contracts_restapi
title: Interacting with Smart Contracts on EthVigil via REST APIs
sidebar_label: Smart Contract REST APIs
---

## Prerequisites
You are familiar with deploying contracts [via the CLI tool](cli_gettingstarted.md) or [the web frontend](web_gettingstarted.md.

## OpenAPI Specification
Once you deploy a contract via EthVigil, we automatically generate REST API endpoints over HTTPS which are available as an [OpenAPI Specification](https://swagger.io/specification/).

### Get the OpenAPI spec link
#### With the CLI tool
```
ev-cli getoas 0xbbd8cda5503e1df2983f738ad131a2304528e3dd

http://192.168.99.100:8080/v0.1/swagger/0xbbd8cda5503e1df2983f738ad131a2304528e3dd/?key=80340b2a-633b-4a33-898c-06055ee10a34
```
The contract being used for this guide comes bundled with the CLI tool by the name `token.sol` and follows the standard implementation of an [ERC20 token contract](https://theethereum.wiki/w/index.php/ERC20_Token_Standard).

### Import the OpenAPI spec in Postman

Click on the import button in the top left corner and select `Import from link`

![Import EthVigil generated OpenAPI Spec in Postman](assets/oas/postman-01-ethvigil-import.png)

You will see the Contract methods populated as HTTP GET and POST endpoints.

![Contract methods listing on EthVigil](assets/oas/postman-02-ethvigil-listing.png)

### GET HTTP endpoints

EthVigil exposes the following as GET endpoints from the Solidity source code of a smart contract
* any `public` variable
* any `public` method that does not alter the state of the contract i.e. ***[`stateMutability` is `pure` or `view`](https://solidity.readthedocs.io/en/v0.5.10/abi-spec.html#json)***

Let us try out a couple of GET calls on the contract that reads the state data on chain.

**Beginning with the public variable `name`**, EthVigil automatically exposes it as a GET endpoint as if it was an accessor.

Relevant Solidity code in `token.sol`:
```
string public  name;
```

![EthVigil HTTP GET call on smart contract 'name'](assets/oas/postman-03-ethvigil-getcall.png)

**Finding out the total supply of this ERC20 contract**

![EthVigil HTTP GET call on smart contract public accessor `totalSupply`](assets/oas/postman-04-ethvigil-getsupply.png)
This tells us the total tokens in supply through this contract amounts to 1000000000000000000000000.

### POST HTTP endpoints

EthVigil exposes the following as POST endpoints from the Solidity source code of a smart contract

* any `public` method that does not alter the state of the contract i.e. ***[`stateMutability` is `payable` or `nonpayable`](https://solidity.readthedocs.io/en/v0.5.10/abi-spec.html#json)***

**We will now invoke the famed `transfer()` method on the contract.**
The request body is already populated with the necessary parameters, courtesy the tight integration between Postman and OpenAPI Specs.

![EthVigil HTTP POST call body for transfer() method](assets/oas/postman-05-ethvigil-transfer-01-body.png)

**To perform POST calls on EthVigil API, an HTTP header `X-API-KEY` is also expected.**

![EthVigil HTTP POST call headers with API key for transfer() method](assets/oas/postman-05-ethvigil-transfer-02-headers.png)

>Referring back to the section on [Subscribing to events on a contract](cli_gettingstarted.mod#subscribe-to-the-transfer-and-approval-events-on-the-contract), there was a webhook endpoint already registered to listen to a `Transfer` event

**Once entering the necessary body and header request parameters, we complete the HTTP POST to `transfer()`**

![EthVigil HTTP POST call to transfer() method](assets/oas/postman-06-transfer-txhash.png)

It returns the transaction hash sent out on the blockchain network corresponding to this request, which is, `0x532c587a2e24f481f73b58858a5849cd2247e2df324f17780a818a7e13f50436`

### Verifying the state changes

We make a call to the public mapping `balanceOf` on the ERC20 smart contract that holds a mapping of Ethereum addresses to their allocated tokens.

![EthVigil HTTP GET call to get balanceOf the address to which transfer was made](assets/oas/postman-07-getbalance.png)

### Verifying the event data payload

The `transfer()` method emits a `Transfer` event which holds necessary information about the call in [*event log topics*](https://media.consensys.net/technical-introduction-to-events-and-logs-in-ethereum-a074d65dd61e).

Checking back on our registered webhook endpoint, we see we have received the following payload:

![EthVigil webhook integration event data payload](assets/oas/postman-08-webhook.png)
