---
id: erc20_example_code
title: Working with an ERC20 token contract
sidebar_label: Working with an ERC20 token contract
---
The guide will introduce you to working with an ERC20 token contract via the EthVigil API endpoints.

These steps are also packaged into a CLI tool designed in Python for you to play around with a pre-supplied ERC20 contract. Visit the repo at ***TODO: github python code link*** to clone and get hacking away.

Alternately, you can keep a note of
* the private key that identifies your user account uniquely on EthVigil
* the API key uniquely alloted to your user account that allows you to make authenticated HTTP requests to the REST API endpoints generated for your smart contract

...and follow along the code examples as stand alone scripts ***TODO: link to standalone scripts directory in github repo of erc20 example code***

## Prerequisites
It is absolutely critical that you would have gone through at least one of our onboarding guides that will teach you the way EthVigil handles user accounts, signing up, logging in, deploying contracts etc.

If you haven't, go check them out.
* [Getting started with the CLI tool](cli_gettingstarted.md)
* [Getting started with the web UI tool](web_gettingstarted.md)

## Deploy the ERC20 contract

You can find the Solidity source code here ***TODO: insert link to contract file from github repo***

You can deploy the contract [from the web UI](web_gettingstarted.md#deploy-a-solidity-smart-contract) or follow the code examples below.

<!--DOCUSAURUS_CODE_TABS-->

<!--Python-->
```py
from eth_account.messages import defunct_hash_message
from eth_account.account import Account
import requests
import json


def main():
    msg = "Trying to deploy"
    message_hash = defunct_hash_message(text=msg)
    private_key = "0xprivatekeyhexstring"
    constructor_inputs = ['My Token', 'SYMBOL', 18]
    sig_msg = Account.signHash(message_hash, private_key)
    with open('./ERC20Mintable.sol', 'r') as f:
        contract_code = f.read()
    deploy_params = {
        'msg': msg,
        'sig': sig_msg.signature.hex(),
        'name': 'ERC20Mintable',
        'inputs': constructor_inputs,
        'code': contract_code
    }
    print('Deploying with constructor arguments: ')
    print(constructor_inputs)
    # API call to deploy
    headers = {'accept': 'application/json', 'Content-Type': 'application/json'}
    api_endpoint = "https://beta.ethvigil.com/api"
    r = requests.post(api_endpoint + '/deploy', json=deploy_params, headers=headers)
    rj = r.json()
    print('Deployed contract results')
    print(rj)


if __name__ == '__main__':
    main()
```
<!--END_DOCUSAURUS_CODE_TABS-->

**Equivalent ERC20 CLI tool command**
```bash
python cli.py deploy
```

## Setup a webhook integration

We will set this up before we proceed any further with sending transactions to the ERC20 contract. Most of the transactions will generate multiple events and it would be nice to see a live update of the events being emitted on the Görli testnet.

>Did you know the EthVigil Beta instance runs on the Görli testnet?

### Launch the bundled webhook listener
This will launch a server locally on the port 5554.
In the next section, we are going to open up a tunnel to it so that it is accessible remotely to deliver event data payloads.

```bash
python webhook_listener.py
```
![python webhook listener](assets/code-examples/webhook-listener-launch.png)
### Launch ngrok
[ngrok](https://dashboard.ngrok.com/signup) is a free service to expose your local web server and quickly test out webhook integrations. Click on the link above to download and setup the tool if you don't have it already.

Then open a tunnel to port 5554, where the local webhook listner is running.

```bash
./ngrok http 5554
```

You will see a screen like the following:
![ngrok screen](assets/code-examples/ngrok-screen.png)

Copy the HTTPS forwarding link, `https://c019aae8.ngrok.io` in this case.

### Register ngrok endpoint as webhook
Now we register the ngrok link from above as a webhook endpoint. This returns us a ID for this endpoint against which event data and other integrations can be configured.

<!--DOCUSAURUS_CODE_TABS-->

<!--Python-->
```python
from eth_account.messages import defunct_hash_message
from eth_account.account import Account
import requests


def main():
    contract = "0xcontractAddress"
    api_key = '1122-122-23443-1133'
    headers = {'accept': 'application/json', 'Content-Type': 'application/json',
               'X-API-KEY': api_key}
    private_key = "0xprivatekeyhexstring"
    api_endpoint = "https://beta.ethvigil.com/api"
    msg = 'dummystring'
    message_hash = defunct_hash_message(text=msg)
    sig_msg = Account.signHash(message_hash, private_key)
    method_args = {
        "msg": msg,
        "sig": sig_msg.signature.hex(),
        "key": api_key,
        "type": "web",
        "contract": contract,
        "web": "https://randomstring.ngrok.io"
    }
    r = requests.post(url=f'{api_endpoint}/hooks/add', json=method_args, headers=headers)
    print(r.text)
    if r.status_code == requests.codes.ok:
        r = r.json()
        if not r['success']:
            print('Failed to register webhook with Ethvigil API...')
        else:
            hook_id = r["data"]["id"]
            print('Succeeded in registering webhook with Ethvigil API...')
            print(f'EthVigil Hook ID: {hook_id}')
    else:
        print('Failed to register webhook with Ethvigil API...')


if __name__ == '__main__':
    main()
```
<!--END_DOCUSAURUS_CODE_TABS-->

**Equivalent ERC20 CLI tool command**
```bash
python cli.py registerhook https://c019aae8.ngrok.io
{"success":true,"data":{"id":10}}
Succeeded in registering webhook with Ethvigil API...
EthVigil Hook ID: 10
```

### Setup the endpoint to receive event data

## A look at the ERC20 Interface

```
interface ERC20 {
  function totalSupply() external view returns (uint256);
  function balanceOf(address who) external view returns (uint256);
  function transfer(address to, uint256 value) external returns (bool);
  function allowance(address owner, address spender) external view returns (uint256);
  function transferFrom(address from, address to, uint256 value) external returns (bool);
  function approve(address spender, uint256 value) external returns (bool);

  event Approval(address indexed owner, address indexed spender, uint256 value);  
  event Transfer(address indexed from, address indexed to, uint256 value);
}
```
We are going to work with the methods as specified by the interface.
