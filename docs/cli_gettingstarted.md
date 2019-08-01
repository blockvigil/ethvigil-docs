---
id: cli_onboarding
title: Getting Started
sidebar_label: Using the CLI tool
---

The guide will introduce you to the EthVigil API endpoints with the help of a CLI tool that abstracts away the underlying HTTP requests. By the end of this guide, you shall be able to perform the following without a graphical frontend:
* sign up using your exclusive EthVigil invite code
* access your account information on EthVigil
* deploy a Solidity smart contract through the CLI tool

>We have also included examples of equivalent cURL/Python requests. If you wish to setup the CLI tool later, skip this section of the guide to learn about the HTTP request endpoints ⏩ ⏩ [Using the HTTP APIs](http_gettingstarted.md)

## Requirements for installing the CLI tool
* Python 3

## Recommended Installation (Pyenv with virtualenv support)

### On macOS
Credits: [this guide](https://medium.com/python-every-day/python-development-on-macos-with-pyenv-2509c694a808?)
```bash
# Install Homebrew if it isn't already available
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# Install pyenv
brew install pyenv
# Add pyenv initializer to shell startup script
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
# Reload your profile
source ~/.bash_profile
# pyenv install <python version>
pyenv install 3.6.5
# set global python
pyenv global 3.6.5
# Install pyenv-virtualenv
brew install pyenv-virtualenv
# Add pyenv-virtualenv initializer to shell startup script
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
# Reload your profile
source ~/.bash_profile
# Create a new virtual environment
pyenv virtualenv 3.6.5 my-hackpad
# Activate Virtual Environment
pyenv activate my-hackpad
```

## CLI tool Installation (via pip)

```bash
# clone the git repo
git clone
cd ethvigil-cli
pip install -e .
```
## Sign up with invite code
```bash
ev-cli init
```

On the next prompt you will be asked for your invite code. The invite code should have been sent to the email address you used to register.

`Enter your invite code: <invite-code-goes-here>`

You should see something like the following

```
http://alpha-api.ethvigil.com/v0.1/signup
{"success":true,"data":{"status":"active","email":"anomitghosh@gmail.com"}}
Sign up succeeded...
Logging in with your credentials...
You have signed up and logged in successfully to EthVigil Alpha
---YOU MIGHT WANT TO COPY THESE DETAILS TO A SEPARATE FILE---
===Private key (that signs messages to interact with EthVigil APIs===
0xprivatekeyhexstring
===ETHEREUM hexadecimal address corresponding to above private key===
0xdFaFF6081f4544fEb76d213DEB2f9DC3C8453b6
```
On EthVigil APIs, you are primarily idenitified by the above 160 bit address that we have generated locally for you.
`0xdFaFF6081f4544fEb76d213DEB2f9DC3C8453b6` in this case.

You can import the corresponding private key to a wallet solution like MetaMask.

>The keys are locally stored and EthVigil does not have access to them. You can check the `settings.json` file once init is complete.

## Get your EthVigil account information

Once initialized, you can find information related to your EthVigil account through the CLI tool.

```
ev-cli accountinfo

Contracts events fired to registered hooks: 	 {'usage': 0, 'limit': 1000}
=============

Registered integrations/hooks: 	 {'usage': 0, 'limit': 10}
=============

EthVigil API key: 	 80340b2a-633b-4a33-898c-06055ee10a34
=============

REST API prefix: 	 http://localhost:9000/api/v0.1
=============

Contracts deployed/verified:
=============
```

## Deploy a Solidity smart contract

We have included a couple of smart contracts written in Solidity in the code repo to help you test out their deployment right away.
You can find them in `token.sol` and `SignerControlBase.sol`

The syntax to deploy a contract through the CLI tool is:

```bash
ev-cli deploy <path-to-solidity-contract> \
 --contractName=<contract-name> \
 --constructorInputs='JSON representation of the constructor arguments'
```
>Currently EthVigil API accepts **only one** Solidity file that contains the entire smart contract logic. It will be enhanced in the near future where we will allow multiple files to be uploaded to ease development with imports of other modules and libraries

### ERC20 token contract example - token.sol
```bash
ev-cli deploy token.sol --contractName=FixedSupplyToken

Contract FixedSupplyToken deployed successfully
Contract Address: 0xb8254a02fa7da599053006913bbed0a13fa0385f
Deploying tx: 0xd5318cf2bc163e267fecbb85ad2688f088fb2f45bc93baa1d9530f2d23b64a26
```
Observe that `--constructorInputs` has been left empty. It is optional for contracts that have no constructor inputs programmed.

### SignerControlBase.sol

This contract forms the base of [EthVigil's Proxy+Signer Control contract](https://medium.com/blockvigil/signer-control-cum-proxy-smart-contract-a-look-at-ethvigils-latest-offering-9ad6c098c095). Without going into the logic of the contract, let us take a look at the constructor as written in the contract.

```
constructor (address[] memory _primaryOwners, address[] memory _secondaryOwners, uint _required_confirmations)
    public
```

#### ⚠️Passing JSON serialized constructor inputs to the CLI tool
>This section deals with passing constructor inputs via the CLI tool. It is always easier to do the same with code.
There can be compounding confusion since
* individual constructor parameters that are arrays are expected by the EthVigil API to be encoded as strings
* the CLI tool itself accepts all the inputs as an array string.


An equivalent representation of the constructor inputs in JSON would look like
```json
[
  "[\"0x774246187E1E2205C5920898eEde0945016080Df\", \"0x902Abade63A5CB1b503Fe389aEA5906D18DAAF2b\"]",
  "[\"0x0228c246170f010C386f49e2dbc7aA999384B399\", \"0x5747Ca27b1031D8054cB9Cbc79F70CD2d9D2E204\"]",
  2
]
```

**Example to generate the same in Python follows**
```python
$ python
>>> import json
>>> _primary = ["0x774246187E1E2205C5920898eEde0945016080Df", "0x902Abade63A5CB1b503Fe389aEA5906D18DAAF2b"],  # primary owners
>>> _secondary = ["0x0228c246170f010C386f49e2dbc7aA999384B399", "0x5747Ca27b1031D8054cB9Cbc79F70CD2d9D2E204"] # secondary owners
>>> inputs = [
  json.dumps(_primary), json.dumps(_secondary), 2  # _required_confirmations
]
# write the entire list of arguments as a JSON serialized string into a file
>>> with open('c_inputs', 'w') as f:
        json.dump(inputs, f)
```
We dumped the JSON representation of the entire array of constructor inputs into a file, `c_inputs`.
We shall pass this next to the command line option of `--constructorInputs` in our call to deploy.


```bash
ev-cli deploy SignerControlBase.sol --contractName SignerControlBase --constructorInputs="$(< c_inputs )" --verbose=1

Got constructor inputs:
["[\"0x774246187E1E2205C5920898eEde0945016080Df\", \"0x902Abade63A5CB1b503Fe389aEA5906D18DAAF2b\"]", "[\"0x0228c246170f010C386f49e2dbc7aA999384B399\", \"0x5747Ca27b1031D8054cB9Cbc79F70CD2d9D2E204\"]", 2]
EthVigil deploy response:
{"success":true,"data":{"contract":"0x746254cb1888a0f073fca2cf397457fb3e54396f","gas":"infinite","hash":"0xcb2cb6f036e01eb22707084f4780d731ee959a50fe6b6a562643cfa40f3d5e2f"}}
Contract SignerControlBase deployed successfully
Contract Address: 0x746254cb1888a0f073fca2cf397457fb3e54396f
Deploying tx: 0xcb2cb6f036e01eb22707084f4780d731ee959a50fe6b6a562643cfa40f3d5e2f
```
## More CLI features
### Dump local settings
```
ev-cli dumpsettings

{'PRIVATEKEY': '0xprivatekeyhexstring', 'INTERNAL_API_ENDPOINT': 'http://192.168.99.100:8080/v0.1', 'REST_API_ENDPOINT': 'http://localhost:9000/api/v0.1', 'ETHVIGIL_USER_ADDRESS': '0x40b93b89f89c674fB97db61d4b2D9CE2C2Cf6EB6', 'ETHVIGIL_API_KEY': '80340b2a-633b-4a33-898c-06055ee10a34'}
```
### Reset
If you wish to begin with a fresh slate, run the tool with the `reset` argument.
>You will lose all saved state, including the private key used to identify your account on EthVigil.
You should either backup the information or request a new invite code

```bash
ev-cli reset

Do you want to reset the current EthVigil CLI configuration and state? [y/N]:
```
### Backup settings and recover later

```bash
ev-cli dumpsettings > settings-backup.json

ev-cli reset

ev-cli importsettings setting-backup.json
```
