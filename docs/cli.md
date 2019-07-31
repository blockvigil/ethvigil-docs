---
id: cli
title: Getting Started
sidebar_label: Setup account and deploy contracts
---

The guide will introduce you to the EthVigil API endpoints with the help of a CLI tool that abstracts away the underlying HTTP requests.

We have also included examples of equivalent cURL requests. If you wish to setup the CLI tool later, skip the Requirements and Installation sections.
## Requirements for the CLI tool

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
