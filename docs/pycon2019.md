---
id: pycon2019_giveaway
title: PyCon India 2019 Ticket Giveaway Contest
sidebar_label: PyCon India 2019 Ticket Giveaway Contest
---
## How do I get my free PyCon India 2019 ticket?

We need to you recreate the first two chapters of the [Cryptozombies game tutorial](https://cryptozombies.io/) that runs on blockchain on EthVigil, our API gateway for the Ethereum blockchain.

![](assets/pycon2019-giveaway/cryptozombies-level2.png)

### But I know nothing about blockchain!
Worry not. You will be using our APIs for Ethereum that abstracts away primitives like transaction, transaction fees(gas costs), ABI, monitoring contract events etc.

Documentation here: https://ethvigil.com/docs

In essence, 
* you will be making GET/POST calls over HTTPS to automatically generated REST API endpoints.
* set up webhook endpoints to which EthVigil will push real time event data from the smart contract

### Cool. How do I get started?

We have slightly modified the source code of the main 'zombie feeding' contract to fetch 'kitty' data from a contract instance that is on the [Ethereum Görli testnet](https://github.com/goerli), not the mainnet. 

> Mainnet transactions cost real Ether monies.

**Zombie feeding contract** 
```
pragma solidity ^0.5.7;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;

    function _createZombie(string memory _name, uint _dna) internal {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        emit NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

    function createRandomZombie(string memory _name) public {
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        randDna = randDna - randDna % 100;
        _createZombie(_name, randDna);
    }
    
}

contract KittyInterface {
  function getKitty(uint256 _id) external view returns (uint256 genes);
}

contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x6feBb2A3E3AeD1691Aba98350ee6CF447E1AF9B8;
  KittyInterface kittyContract = KittyInterface(ckAddress);

  // Modify function definition here:
  function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(abi.encodePacked(_species)) == keccak256(abi.encodePacked("kitty"))) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    kittyDna = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
``` 


**Dummy cryptokitty contract**
`0x6feBb2A3E3AeD1691Aba98350ee6CF447E1AF9B8` in the code above is an instance of a dummy contract deployed on the Görli testnet that emulates the interface of the main CryptoKitties contract by supplying a `getKitty()` method implementation. 

> This is just a placeholder contract. Feel free to make it as complex and detailed as you wish to support more 'character traits'. We suggest reading through The Oatmeal's guide on cats: [How much does your cat actually kill](https://theoatmeal.com/comics/cats_actually_kill) 

``` 
pragma solidity ^0.5.7;

contract DummyCryptoKitties {
    function getKitty(uint256 _id) external view returns (uint256 genes) {
      return uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty, _id)));
  }
}
```

## Quick references to get started

### Deploying a contract
* [Using the CLI tool](cli_gettingstarted.md#deploy-a-solidity-smart-contract)
* [Using the web UI](web_gettingstarted.md#deploy-a-solidity-smart-contract)

### Adding webhook integrations

* [Using the CLI tool](cli_gettingstarted.md#webhooks)
* [Using the web UI](web_gettingstarted.md#webhooks)


### Detailed walkthroughs
We would strongly suggest going through any of the following walkthroughs we have published to get comfortable with reading from and writing to a smart contract via the API gateway as well as connecting off-chain logic through webhook integrations.

* [Working with an ERC20 token contract](erc20_example.md)
* [Signing and Verifying Ethereum messages](eth_sign.md)

 
## Rules

## Terms and Conditions