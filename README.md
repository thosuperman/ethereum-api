# Ethereum/Token API

## Table of Contents

- [Quick Start](#quick-start)
- Getting Started
  - [Run Private Test Node](#run-private-test-node)
  - [Config](#config)
  - [Environment Variables](#environment-variablese)
  - [Truffle Token Contract](#truffle-token-contract)
  - [Development local node](#development-local-node)
  - [Testnet node](#testnet-node)
  - [Run truffle commands](#run-truffle-commands)
  - [Server](#server)
  - [Adding Networks](#adding-networks)
- [Notes](#notes)
    - [Token](#token)
    - [Ethereum](#ethereum)
- [Scripts](#scripts)
- [Bash](#bash)
- [Helpers](#helpers)
- [Keystore](#keystore)
- [Error Debugging](#error-debugging)
- [Sample Contracts](#sample-contracts)
- [Version Control]($version-control)
- [Connections](#Connections)
- [Bugs to Fix](#bugs-to-fix)
- [Improvements](#improvements)
- [ERC20 Token Endpoint Notes](#erc20-token-endpoint-notes)
    - [Owner Token API](#owner-token-api)
    - [ERC20 Multi-Token API](#erc20-multi-token-api)
        - [Setup](#setup)
        - [Get Balance](#get-balance)
        - [Transfer](#transfer)
    - [Token Transaction Verification](#token-transaction-verification)
  - [Ethereum Endpoint Notes](#ethereum-endpoint-notes)
      - [Get Block Info](#get-block-info)
      - [Create address](#create-address)
      - [Get Transaction Info](#get-transaction-info)
      - [Send Transaction](#send-transaction)
      - [Sign and Send Manually](#sign-and-send-manually)
          - [Get Transaction Info](#get-transaction-info)
          - [Sign Transaction](#sign-transaction)
          - [Send Signed Transaction](#send-signed-transaction)
- [Webhook](#webhook)
- [Websockets](#websockets)
- [List of All API Endpoints](#endpoints)

## Quick Start

Install [Ganache](http://truffleframework.com/ganache/) to have a local blockchain and start it. You will see 10 address and a mnemonic phrase. Copy `RPC Server` URL, e.g `HTTP://127.0.0.1:7545`

```
brew cask install ganache
```

Create `.env` in root

```
NODE_URL=http://127.0.0.1:7545
```

Clone repo and start server

```
git clone git@github.com:ar-to/ethereum-api.git
cd ethereum-api
npm install
npm start
```
in new terminal check if connected and what accounts you have on the node
```
curl http://localhost:3000/api/eth/syncing
{"nodeSynced":true}
http://localhost:3000/api/eth/accounts
{"accounts":["0xAb7faf7bDAE1B9D0F757e2a8aB120619b388C4c6","0x451E62137891156215d9D58BeDdc6dE3f30218e7","0x22B55D4cc5cE3E32Ee31B0684172E2BCE9F722e7","0x71c9625B0005F20d264775cfF8fc9FB1BEf96525","0x48E5A9807A1C862CeB00a9867c1b57dF02b8F1Fe","0xb7B3FaD7d81C5D2e09Dc464Fec36AC6b4e1B04d3","0x16e665134A1A3b048b2d9aFdB612Bd34CAc0F35C","0xcCFf4FEa69126b9E2c7ce02d69d7c5205657e722","0x032D0Fa0AD21aa5a50C6c6e13D9d14a9550457C5","0x20032730927fB07C46e20FD3725C1f77b04cd4ee"]}
```


## Getting Started

Please read [notes](#notes) for important information before developing.


### Run Private Test Node

The easiest method to run a local private ethereum node that will also mine blocks to facilitate creating transactions and to tests this api is via [Ganache](http://truffleframework.com/ganache/) GUI application. 

```
brew cask install ganache
```
Open application and it will automatically create 10 addresses. Save the mnemonic phrase if you want to use it again. It will be set the url to `http://127.0.0.1:7545`

### Config
Add connections to `config/connections.json`, and set the name of the network you want to use for the API using `"connectApi": "network-name",`

```
{
  "networks": {
    "connectApi": "ganache",
    "ganache":{
      "url": "http://127.0.0.1:7545",
      "networkId": "5777",
      "networkName": "ganache",
      "networkType": "testrpc",
      "token": {
        "ownerAddress": "0x451E62137891156215d9D58BeDdc6dE3f30218e7",
        "tokenContractAddress": "0x4c59b696552863429d25917b52263532af6e6078",
        "migrateContractAddress": "0xcf068555df7eab0a9bad97829aa1a187bbffbdba"
      },
      "erc20Tokens": []
    },
    "ropsten":{},
    "mainnet":{}
  }
}
```

Copy the same parameters from ganache above to other networks.

### Environment Variables

You can change the port for the server and overwrite the node url from the connections.json by adding it in a `.env` file at the root directory.

```
PORT=4000
NODE_URL=http://127.0.0.1:7545
```

**Remember to restart your server after changing environment variables. If running nodemon run `npm run nodemon` again**

### Truffle Token Contract

This requires familiarization with truffle framework and open-zeppelin for making ERC20 token contracts.

#### Development local node

First add the same OWNER_ACCOUNT address from the environment variable into the `truffle.js` file `from` parameter so the owner to the contract is set to this address when deployed.

```
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*", // Match any network id
      from: "0x451E62137891156215d9D58BeDdc6dE3f30218e7"
    },
  }
```

#### Testnet node 

(e.g. Ropsten, etc)

See section on adding new networks

#### Run truffle commands
Make sure you have truffle installed glabally before trying these commands.

Run Compile when changes are made to the `token-contract/contracts` directory.
```
bash bin/truffle-compile
```

Run Migrate to deploy contract to node. Truffle will migrate new deployments inside `token-contract/migrations` directory but ignore those already sent.

```
bash bin/truffle-migrate
```

Resetting to overwrite the existing contracts can be done with:
```
bash bin/truffle-migrate-reset
```
See [scripts](#scripts) for all available scripts.

**Remember to update the addresses in the `.env` file so the api will know which address to use for the token contract and owner.**

### Server

start the server
```
git clone url
cd projectname
npm install
npm start"
```
start server with custom port and in debug mode
```
npm run dev
```
To run server and watch for changes and run debugger via `--inspect`
```
npm run nodemon
```

## Adding Networks

Configuring testnets and mainnet is more complicated than with local nodes. You need to have a node that is connected to those blockchains and a url that is allowed to receive RPC calls. Then to connect it with this api, create a new JSON file for the network you are adding to `network-wallets/ropsten.config.json`

Then copy the following configuration and change it with the appropriate keystore and password:
```
{
  "keystore": {
    "address": "008aeeda4d805471df9b2a5b0f38a0c3bcba786b"
    "crypto" : {
      "cipher" : "aes-128-ctr",
      "cipherparams" : {
          "iv" : "83dbcc02d8ccb40e466191a123791e0e"
      },
      "ciphertext" : "d172bf743a674da9cdad04534d56926ef8358534d458fffccd4e6ad2fbde479c",
      "kdf" : "scrypt",
      "kdfparams" : {
          "dklen" : 32,
          "n" : 262144,
          "r" : 1,
          "p" : 8,
          "salt" : "ab0c7876052600dd703518d6fc3fe8984592145b591fc8fb5c6d43190334ba19"
      },
      "mac" : "2103ac29920d71da29f15d75b4a16dbe95cfd7ff8faea1056c33131d846e3097"
    },
    "id" : "3198bc9c-6672-5ab3-d995-4942343ae5b6",
    "version" : 3
  },
  "password": "password",
  "url": "http://127.0.0.1:8545"
}
```
Read about what is a [keystore](#keystore)

Then inside `truffle.js` add the following to the top

```
// ROPSTEN testnet
const ropstenConfig = require("../network-wallets/ropsten.config.json");
var ropstenProvider;
if(ropstenConfig){
  const keystore = ropstenConfig.keystore;
  const pass = ropstenConfig.password;
  const url = ropstenConfig.url
  console.log('keystore', url)
  var wallet = require('ethereumjs-wallet').fromV3(keystore, pass);
  ropstenProvider = new WalletProvider(wallet, url);
}
```

still inside `truffle.js` add a new network:

```
ropsten: {
  provider: ropstenProvider,
  network_id: '3',
  gas: 2249435,
  gasPrice: 20000000000,
},
```
Grab the gas and gas price from [helpers](#helpers) directory, using the `token-contract/helpers/gasEstimate.js ` file.

Migrate/deploy with truffle. You can also add a script inside `bin/`

```
bash bin/truffle-migrate-ropsten
```

**Do same for mainnet using `network-wallets/mainnet.config.json`**

## Production

[PM2](http://pm2.keymetrics.io/) is used to handle starting the server, watching for changes in the directories and restarting the server if it crashes. 

Deploy your repo file to a remote server and install pm2 globally

```
npm i -g pm2@latest
cd ethereum-api
npm install
npm run prod
```
This runs `pm2 start ecosystem.config.js` which start the server and watches for changes.

Pm2 helpful commands

- `pm2 logs API` - this shows the logs for requests and errors. You can see these logs in `~/.pm2/logs/API-error-0.log`
- `pm2 show API` - shows information about process
- `pm2 stop API` - this stops process.
- `pm2 delete API` - delete process after its stopped.  NOTE: may be necessary if the server does not restart after files are updated

**To restart server when machine reboots**

## Notes

1. This api uses web3 for interacting with the node, but manual curl commands can be used via [RPC calls](https://github.com/ethereum/wiki/wiki/JSON-RPC). Test the `bash bin/rpc-call` to test an rpc call.

2. Install a solidity extension into your choosen editor when developing contracts

3. the contracts are compiled into the `token-contract/build/contracts` directory and it was added to `/gitignore` as a **comment** to facilitate migrating new contracts during development when using a local private node. But when coming back to the default contract simply compile and migrate or migrate with reset to overwrite existing contracts. 

4. Extra directories: 
- `deployed-contracts` directory that can store the artifact files (e.g.abi, Token.json) if you want to keep different deployments to interact with your contracts in the future. Read [here](#vc) for storing contracts in version control.

### Token

1. This API is meant ot be used with an ERC20 token. Future release may add other tokens or feel free to experiment and pull request a new token.
2. To use a custom abi for interacting with the erc20 add `"customAbi": "abi.js"` to the `config/connections.json`. The value refers to the name of the file (requires the extension) and this file need to be added inside the `deployed-contracts` directory.

```
"customAbi": "abi.js"
```


### Ethereum

1. This api implement web3 v1.0 instead of manual rpc calls to the node.

## Scripts

Scripts are added for convenience to allow commands to be performs from the root and for testing.

- rpc-call - a curl command via an rpc call to communicated to a node. Remember to change the url:port to connect to the correct node and address when using the `eth_getBalance` method or pass it as an argument:
```
sh bin/rpc-call http://127.0.0.1:7545
```

- bash bin/truffle-compile

- bash bin/truffle-migrate

- bash bin/truffle-migrate-reset

- bash bin/truffle-migrate-ropsten

- bash bin/truffle-gas-estimate

## Bash
There are scripts that perform operations and follow this tips to create new ones.

Create a new bash or node script 
```
#!/usr/bin/env bash
#!/usr/bin/env node
```
```
chmod +x ./bin/newscript
```
Add it to package.json is optional
```
  "scripts": {
    "script": "./bin/newscript"
  }
```

## Helpers

The `token-contract/helpers` directory has files that can help with contract,truffle or web3.

- gasEstimate.js - estimate the gas, gas price and total gas that a contract will consume. This information is needed for deploying to testnets and mainnet. 

run
```
bash bin/truffle-gas-estimate
```

output:

```
Using network 'development'.

Gas Price is 20000000000 wei
Gas Price is = 20 gwei
gas estimation = 2249435 units
gas cost estimation = 44988700000000000 wei
gas cost estimation = 0.0449887 ether
```

## Keystore

Read about what is a [keystore](https://medium.com/@julien.m./what-is-an-ethereum-keystore-file-86c8c5917b97) file and how to make one. Above keystore taken from [here](https://github.com/hashcat/hashcat/issues/1228) If using the geth client for creating your node, you can run `geth account new`, enter your password and check the `keystore/` directory for the keystore file. This directory is next to the chaindata where you are storing the blockchain data.

## Error Debugging

Setting up can be somewhat complicated if not tedious. Maybe in future releases there will be much more efficient way to setup but for now fixing and understanding error is best to guarantee everything is running correctly. Below are a few warnings or error you may see and possible ways to solve them. More added as they are found. Keep in mind error do not necessarily mean bugs, but it does not mean a bug is not present. If an error turns out to be a bug file an issue and your solution. Thanks.

- `Invalid JSON RPC response: ""` - This may show as a response from testing the api endpoint the first time you are connecting to a node. It is a failure to connect by web3 provider. Solution:
    1. restart your server
    2. Check you have the correct node url inside the `.env` file
    3. Make an RPC call to the node using the `bin/rpc-call` script to test for connection. See [scripts section](#scripts) for details
    4. wait until the connection is successful and try the endpoint you got the error again
- {package} `was compiled against a different Node.js version using` - best solution is to rebuild packages by first updating npm and node and then `npm rebuild`



## Sample Contracts

Sample constracts are used for reference and can be found in `token-contract/samples`. 

- advancedToken.sol - sample token code that has been fixed to meet solidity v0.4.23. This can be tested by adding it into Mist app. Deploy using TokenERC20 token name.

## Version Control
Question about storing artifact files (e.g. Token.json) after truffle compiles contracts with abi and contract addresses for different networks. Community in gitter (chat app) recommended to add to VC but optional not to.

- [stackexchange](https://ethereum.stackexchange.com/questions/19486/storing-a-truffle-contract-interface-in-version-control) - recommendation to store outside
- [artifact-updates](https://github.com/trufflesuite/artifact-updates) - community repo for updating some issues with doing this.

## Connections

Full example of `connections.json` and available parameters

```
{
  "networks": {
    "connectApi": "ropsten",
    "ganache":{
      "url": "http://127.0.0.1:7545",
      "websocketUrl": "'ws://localhost:7545",
      "networkId": "5777",
      "networkName": "ganache",
      "networkType": "testrpc",
      "token": {
        "ownerAddress": "0x451E62137891156215d9D58BeDdc6dE3f30218e7",
        "tokenContractAddress": "0x4c59b696552863429d25917b52263532af6e6078",
        "migrateContractAddress": "0xcf068555df7eab0a9bad97829aa1a187bbffbdba"
      }
    },
    "ropsten":{
      "url": "http://10.10.0.163:8545",
      "websocketUrl": "ws://10.10.0.163:8546",
      "blockWebhookUrl": "http://4f5a64ba.ngrok.io/api/eth/webhook",
      "syncingWebhookUrl": "http://4f5a64ba.ngrok.io/api/eth/webhook",
      "testWebhookUrl": "http://4f5a64ba.ngrok.io/api/eth/webhook",
      "networkId": 3,
      "networkName": "ropsten",
      "networkType": "testnet",
      "token": {
        "ownerAddress": "0x83634a8eaadc34b860b4553e0daf1fac1cb43b1e",
        "tokenContractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb",
        "migrateContractAddress": "0xa8ebf36b0a34acf98395bc5163103efc37621052",
        "customAbi": "abi.js"
      },
      "erc20Tokens": [
        {
          "name": "threshodl",
          "contractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb"
        },
        {
          "name": "golem",
          "contractAddress": "0x9a0027f3c0fc4fab7825fcf50dd55dfdcca07cd6"
        },
        {
          "name": "bokky",
          "contractAddress": "0x583cbBb8a8443B38aBcC0c956beCe47340ea1367"
        }
      ]
    },
    "mainnet":{}
  }
}
```

## Bugs to Fix

transferOwnership & kill functions both catch an 'invalid address'. The response it 200 with `false` boolean indicating request failed. 

- `api/eth/tx-from-block/hashStringOrNumber` endpoint seems to fail when connected to Ropsten Testnet but not with ganache local node

- subscription does not seem to work and or do

## Improvements

There is always room for improvement, but below is a list of tasks to tackle first.

- error handling needs to be better and standarized. Using combination of promises/catch and try/catch but only some error are sent as a response, most stop the server and the requests gets timedout.
- add a way to calculate default gas when transferring tokens for multi-token and owner-token apis
- Get `transferFrom` branch working and debugged, which also included payments to the contract. This branch is for the owner api but can be integrated into the multi-tokens api once it works correctly.
- Add unit tests via mocha, jest
- Add logging via [winston package](https://github.com/winstonjs/winston) and rotate daily
- setup a quick one command program that will create a private Ethereum node and connect node to it. Possible adding flags for the type (testnet, main, or specific network like ropsten). Test [bitcore](https://bitcore.io/) does this so test and use as reference on how  a config is setup and everything is created.
- setup a docker container to quickly run server

## ERC20 Token Endpoint Notes

There is currently two ways to communicate to tokens, both requiring erc20 tokens. The first is for general requests and owner specific requests. The second, is meant to communicate to multiple tokens by simply adding a name for the endpoint and the token contract used for communication. 

### Owner Token API
This api is used for communications with the erc20 contract deployed within this repo and given in the `connections.json`:

```
      "token": {
        "ownerAddress": "0x83634a8eaadc34b860b4553e0daf1fac1cb43b1e",
        "tokenContractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb",
	...
      },
```

The paths that can be used that are specific to this api are:

```
http://localhost:3000/api/token/:options
/owner
/node-accounts
/balance
/balance/:address
/owner
/add-tokens/:amount
/transfer-tokens
/transfer-owner
/kill-token
```

### ERC20 Multi-Token API

This API supports multiple erc20 token connections. This means, you can communicate to different tokens by simply changing a path parameter in the url. See the instructions below Below you change the `tokenName` and `erc20Method` and

The paths that this api features are below:
```
http://localhost:3000/api/token/:tokenName/:erc20Method
/:tokenName
/:tokenName/getbalance/:address
/:tokenName/transfer
/:tokenName/request-transfer
```

#### Setup 

Add the `erc20Tokens` array parameter to the `config/connections.json` to add new tokens. Then add each new token as an object with a name and contractAddress parameter.

```
{
  "networks": {
    "connectApi": "ropsten",
    "ganache":{
...
    },
    "ropsten":{
...
      "token": {
...
      },
      "erc20Tokens": [
        {
          "name": "threshodl",
          "contractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb"
        },
        {
          "name": "golem",
          "contractAddress": "0x9a0027f3c0fc4fab7825fcf50dd55dfdcca07cd6"
        },
        {
          "name": "bokky",
          "contractAddress": "0x583cbBb8a8443B38aBcC0c956beCe47340ea1367"
        }
      ]
    },
    "mainnet":{}
  }
}
```

Then test is by adding it the name path param

Request (GET method):
```
http://localhost:3000/api/token/threshodl
```

Response:
```
{
    "method": "test",
    "params": {
        "tokenName": "threshodl"
    },
    "erc20Available": [
        {
            "name": "threshodl",
            "contractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb"
        }
    ],
    "success": "Token threshodl is available on this api!"
}
```

You can now use the rest of the api endpoints.

#### Get Balance

Get the balance for an address with the token selected.

**Sample**
Request (GET method):
```
http://localhost:3000/api/token/threshodl/getbalance/0x07CE1F5852f222cc261ca803a1DA4a4016154539
```

Response:

```
{
    "method": "getbalance",
    "params": {
        "tokenName": "threshodl",
        "address": "0x83634a8eaadc34b860b4553e0daf1fac1cb43b1e"
    },
    "erc20Available": [
        {
            "name": "threshodl",
            "contractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb"
        }
    ],
    "tokenBalance": "11300"
}
```

#### Request Transfer

This endpoint transfers tokens from the owner to any address, hence the request of tokens. The caveat is to unlock the owner address provided in the `connections.json` because it takes this as the global owner. This endpoint could be part of the owner-token api but because it can be made by any token it is best left here.

#### Transfer

Transfer tokens between any two addresses. 

Url:
```
http://localhost:3000/api/token/threshodl/transfer
```

Options: 

- gas[optional] - Example: "gas": 41000. Defaults to gas from web3.eth.estimateGas()
- fromAddres[required]
- toAddress[required]
- value[required] - number/string. Keep in mind the decimal spaces for the token contract. Usually is 18. Example below is 2, so 100 is actually 100 tokens.
- privateKey[required] - sender(fromAddress) private key. 
- priority[required] - low, medium, high. Based on thi [api](https://ethgasstation.info/json/ethgasAPI.json) and fast, average and safelow are divided by 10.

Request:
```
{
	"toAddress": "0x07CE1F5852f222cc261ca803a1DA4a4016154539",
	"value": 100,
	"gas": 41000,
	"privateKey": "0x7f74657374320000000000000000000000000000000000000000000000000057"
}
```

Response:
```
{
    "method": "transfer",
    "params": {
        "tokenName": "threshodl"
    },
    "body": {
        "toAddress": "0x07CE1F5852f222cc261ca803a1DA4a4016154539",
        "value": 100,
        "gas": 41000,
        "privateKey": "0xa6b7d3fb531567b199d025ded92cb86c685dba5247d4455041319bb0108985e7"
    },
    "erc20Available": [
        {
            "name": "threshodl",
            "contractAddress": "0x3e672122bfd3d6548ee1cc4f1fa111174e8465fb"
        }
    ],
    "transferABI": "0xa9059cbb00000000000000000000000007ce1f5852f222cc261ca803a1da4a40161545390000000000000000000000000000000000000000000000000000000000000064",
    "txSignature": {
        "messageHash": "0x28f36a05a927a8d245119baef7a0f0333e441dca6256601b42fba8c827b87b6b",
        "v": "0x2a",
        "r": "0xac47468f77fa1c862e66a28e76672e1a9586bca5074df3c2af8d0c53b9dfff36",
        "s": "0x1d8b87ff4d719103aff2a4384444a356f2d03bcbbeec6208acedfbcd0e6b569c",
        "rawTransaction": "0xf8a841843b9aca0082a028943e672122bfd3d6548ee1cc4f1fa111174e8465fb80b844a9059cbb00000000000000000000000007ce1f5852f222cc261ca803a1da4a401615453900000000000000000000000000000000000000000000000000000000000000642aa0ac47468f77fa1c862e66a28e76672e1a9586bca5074df3c2af8d0c53b9dfff36a01d8b87ff4d719103aff2a4384444a356f2d03bcbbeec6208acedfbcd0e6b569c"
    },
    "sendTxHash": {
        "txHash": "0x3d556ffdb7faa9577eec1251a84477d1ddaa5a12e64920d4f2bad2fc64d6cf9d"
    }
}
```

### Token Transaction Verification

Verifying a token transaction can be done fast by looking at [Ropsten Etherscan](https://ropsten.etherscan.io) for example. Go to your address and check the transaction was successful, then check the contract address and receiver address for the same transaction.

To do this with this api and potentially create a check using blocks and transactions you need to do the following:

- perform the transfer
- get the block and transactions for that block via a GET request to this endpoint `http://localhost:3000/api/eth/block/{block-number}?showTx=true` and replace the block number with the once you see in Etherscan or via a custom webhook that receives new blocks. The response will show all the details for each transaction. Then use the transaction hash you got from the transfer and search it the response.
- Use the "to" address from that transaction and perform a contract check to verify that it is a contract. Send a GET request to `http://localhost:3000/api/token/check-for-contract/{address}`. The response will give you a ` "isContract": true,` and the contract bytecode if its a contract.
- Check the "from" address in that transaction and you will see the address tokens would have been received at. This and the contract check is the way to verify a token transfer. 

## Ethereum Endpoint Notes

The ethereum endpoint for this API uses the [web3js](http://web3js.readthedocs.io/en/1.0/) so parameters for a web3 method is normally supported by the endpoint unless otherwise specified. 

### Get Block Info

The following endpoints can be used to get information about blocks and even transactions.

- `/api/eth/tx:txHash` : get transaction details by the transaction hash
- `api/eth/tx-from-block/hashStringOrNumber` : hashStringOrNumber can be same as in the [web3 docs](http://web3js.readthedocs.io/en/1.0/web3-eth.html#gettransactionfromblock), e.g. `/latest` or `0xa343d89bb0447c8a22c8ce73bf35504d9363e234b2a1a8229d40b69ce3439fc5`. See [bug](#bugs-to-fix) for problem when using it in Ropsten

- `api/eth/block/2?showTx=true&useString=latest` : 1 is the blocknumber, the query `showTx` will show the transaction object in the output when set to true, and the query `useString` will overwrite the block number and use one of the strings used in the [getBlock api](http://web3js.readthedocs.io/en/1.0/web3-eth.html#getblock) such as `latest` to get the most recent block.

### Create address

GET request
```
http://localhost:3000/api/eth/create-account
```

### Get public address from private key

GET request
```
http://localhost:3000/api/eth/get-account/{privatekey}
```

### Get Transaction info


You can get information about the transaction you want to send before sending it. Information like gas, gas price, amount in wei and ether, etc.

send a POST request with transaction object below to the `api/eth/send-tx-info` endpoint
```
{
  "from": "0x451E62137891156215d9D58BeDdc6dE3f30218e7",
  "to": "0xAb7faf7bDAE1B9D0F757e2a8aB120619b388C4c6",
  "value": "0.1",
  "gas":"41000",
  "gasPrice": "high"
}
```
output
```
{
    "amountToSendEther": "0.1",
    "amountToSendWei": "100000000000000000",
    "gasPrices": {
        "low": 7,
        "medium": 9,
        "high": 14
    },
    "gasPriceTypeDefault": "low",
    "gasPriceDefault": 7000000000,
    "gasPriceTypeCustom": "high",
    "gasPrice": 14000000000,
    "estimatedGas": 21000,
    "gas": "41000",
    "params": {
        "from": "0x451e62137891156215d9d58beddc6de3f30218e7",
        "to": "0xab7faf7bdae1b9d0f757e2a8ab120619b388c4c6",
        "gasPrice": "0x342770c00",
        "value": "0x16345785d8a0000"
    },
    "paramsUpdated": {
        "from": "0x451E62137891156215d9D58BeDdc6dE3f30218e7",
        "to": "0xAb7faf7bDAE1B9D0F757e2a8aB120619b388C4c6",
        "gas": "41000",
        "gasPrice": 14000000000,
        "value": "100000000000000000"
    }
}
```

You can change the following:

- from - takes a valid ethereum address
- to - takes a valid ethereum address
- value - takes a string number in ether
- gas - you can send it without gas to see the estimatedGas and then modify it from there
- gasPrice - ["low", "medium", "high"] are the options and you can see that in the gasPrice{} object that is outputted. So if you don't pass `"gasPrice": "high"` from the above example the transaction will return `gasPriceDefault` value.

The rest of the parameters are for your information:

- gasPrices{} - taken from `https://ethgasstation.info/json/ethgasAPI.json`
- gasPriceTypeDefault - default gas using `gasPrices.low * 1000000000;`
- estimatedGas - taken from `web3.eth.estimateGas(txObject)`, where `txObject` is created from the other parameters and is shown as `params`
- params - transaction object that is used to calculate the estimated gas
- paramsUpdated - is the final transaction object that will be used for sending the transaction

Note: you may notice that `nonce` is not a parameter accepted for changing because it is optional for the `sendTransaction()` function. If required it needs to be updated in the api to accept it as a parameter.

### Send Transaction

Sending a transaction uses the `paramsUpdated` object and passes it to `web3.eth.sendTransaction(paramsUpdated)` and **requires a the from address to be an unlocked keystore address from inside the node**

output 

```
{
    "transactionHash": "0x7c948c9d17c80ae0e89074eddab614c2ce24d6185b21f9aa2d9fb03d393d3dec",
    "transactionIndex": 0,
    "blockHash": "0x5ea250d8c2a1e4e9cead27a457f79cce084a494f9f3009e9f806d7a81030296d",
    "blockNumber": 12,
    "gasUsed": 21000,
    "cumulativeGasUsed": 21000,
    "contractAddress": null,
    "logs": [],
    "status": true,
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
}
```

### Sign and Send Manually

To sign with a private key and send the raw transaction follow these instructions for constructing your request:

#### Get Transaction Info

`http://localhost:3000/api/eth/send-tx-info` POST request with the following json to see your data:

```
{
  "from": "0xA8D26e47CD1CB6Fc0803cA7D64D395bb38bc33de",
  "to": "0x87E426ff6Deb0dF15CD98CAae0F63cf027D711b3",
  "value": "0.002"
}
```

If you are sending transactions from the same `from` address consecutively, you will need to increase the nonce by one so add it to your request

```
  "nonce": 1
```
You will see the a change in the nonce. You want to pay attention to the `paramsToSign` parameter because it will be used to sign the transaction next.

#### Sign transaction

`http://localhost:3000/api/eth/sign-tx` and pass the POST request json:

```
{
  "from": "0xA8D26e47CD1CB6Fc0803cA7D64D395bb38bc33de",
  "to": "0x87E426ff6Deb0dF15CD98CAae0F63cf027D711b3",
  "value": "0.002",
  "privateKey": "0x7f74657374320000000000000000000000000000000000000000000000000057"
}
```
See [web3 api](http://web3js.readthedocs.io/en/1.0/web3-eth-accounts.html#signtransaction) for details on the out put and grab the `rawTransaction` parameter hex encoded transaction


#### Send Signed Transaction

`http://localhost:3000/api/eth/send-signed-tx` POST request with json:

```
{
	"rawTransaction": "0xf86b04850218711a008252089487e426ff6deb0df15cd98caae0f63cf027d711b387071afd498d00008029a063542de27e66ea4a92f1270cbef3cd6ed9ad0b8bafbdcd0cedfcc25e2d731da7a05a7fb9dec086a6b1e4c0c19410b1b2f10a76c9f6198351731e92096534b67624"
}
```

## Webhook

Webhooks allow data to be sent as POST request to an external url specified in the `config/connections.json` inside the `blockWebhookUrl` paramater. 

**To use this webhook you need to have websockets connected see [websockets](#websockets) for more information**

```
"ropsten":{
  ...
  "blockWebhookUrl": "http://4f5a64ba.ngrok.io/api/eth/webhook",
  "testWebhookUrl": "http://4f5a64ba.ngrok.io/api/eth/webhook",
  ...
},
```

The `testWebhookUrl` parameter is used for testing but following these instructions:

Run this api server and then run these

```
brew cask install ngrok
ngrok http 3000
```
grab the ngrok url and add it the `testWebhookUrl` parameter e.g.`http://4f5a64ba.ngrok.io/api/eth/webhook`.

Then open the debug url for ngrok e.g. `http://127.0.0.1:4040`

Send a post request to `http://localhost:3000/api/eth/post-to-webhook` and watch two request show, one for the request and the second that send another POST request to the `/api/eth/webhook` endpoint

## Websockets

A websocket connection to the node is used to subscribe to events (e.g. new blocks, transactions, see [web3 api](http://web3js.readthedocs.io/en/1.0/web3-eth-subscribe.html)) and required to use the webhook feature which sends these subscriptions to a specified url.

**Setup**

You need to enable a websocket connection inside the node and add that url to the `connections.json`

```
"ropsten":{
  ...
  "websocketUrl": "ws://10.10.0.163:8546",
  ...
},
```

or `.env` files which will overwrite the connections.json parameter

```
WEBSOCKET_URL=ws://127.0.0.1:8546
```

Websockets are an experimental feature, and the code can be found inside `server/app/helpers/web3-websocket.js`
There is one `subscriptionType` that is corrently tested and working:

```
  blockSubscription
```

Another is to get status on the syncing of the node with the most current block. **Needs more testing so use with caution**
 
```
  syncingSubscription
```

**Create subscription**
To create a subscription to block headers so every new block will be emitted and POSTed to a webhook url.

```
http://localhost:3000/api/eth/subscribe-block
```

**Remove Subscription**

```
http://localhost:3000/api/eth/close-subscriptions/blockSubscription
http://localhost:3000/api/eth/close-subscriptions/syncingSubscription
```


## Endpoints

To get this endpoints run:

Request (GET method):
```
http://localhost:3000/api-endpoints
```

Response: 
```
[
    {
        "path": "/",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api",
        "methods": [
            "GET",
            "POST"
        ]
    },
    {
        "path": "/api/network",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/get-web3-provider",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/get-contract",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/get-contract-instance",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/check-for-contract/:address",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/node-accounts",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/balance",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/balance/:address",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/owner",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/add-tokens/:amount",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/token/transfer-tokens",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/token/transfer-owner",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/token/kill-token",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/transfer-from",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/token/:tokenName",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/:tokenName/getbalance/:address",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/token/:tokenName/transfer",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/token/:tokenName/request-transfer",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/token/:tokenName/decode-tx-input",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/syncing",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/balance/:address",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/block",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/block/:blockNumber",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/tx/:transactionHash",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/tx-receipt/:transactionHash",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/tx-from-block/:hashStringOrNumber",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/create-account",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/get-account/:privateKey",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/accounts",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/wallet",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/wallet/create",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/wallet/add/:privateKey",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/wallet/remove/:publicKey",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/wallet/clear",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/send-tx-info",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/sign-tx",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/send-signed-tx",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/send-tx",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/subscribe-syncing",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/subscribe-block",
        "methods": [
            "GET"
        ]
    },
    {
        "path": "/api/eth/close-subscriptions/:subscriptionType",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/post-to-webhook",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api/eth/webhook",
        "methods": [
            "POST"
        ]
    },
    {
        "path": "/api-endpoints",
        "methods": [
            "GET"
        ]
    }
]
```