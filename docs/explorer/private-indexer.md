---
id: private-indexer
disable_pagination: true
title: Private indexer for private and public
slug: /explorer
---

import NotificationBar from '../../src/components/docs/NotificationBar';

The tools that have been presented in this module are public: everyone can use it.
However, they are meant for public networks, and the user is dependent on the infrastructure and configuration of these services.

As seen in the **Dapp module**, these tools are extremely useful to get all the pieces of information of what is going on in the network.
Private networks are used in many use cases: foregoing such tools and remaining blind to what is going on in a private network is a deterrent.

Hopefully, all those tools are open-source and be installed and used on a private server.
The user can configure the indexer as he likes, and can set it to watch private networks

In this chapter, _tzindex_ will be used as a private indexer, for both private and public network

# Prerequisites
To index a private network, a few things are needed:
- a private tezos network
- accounts
- a smart contract
- operations


First, a private network has to be set up. 
The quickest way to set up a private Tezos network is to use Ganache (see ??? for more details).
Ganache is a npm module, that can set up a personal private network.
Ganache provides 10 accounts when it starts

The raffle smart contract, developed in the LIGO module, will be used. 
It will be migrated onto our private network with the truffle configuration detailed in the Dapp module.

## Installing the prerequisites
A github repository is available here:
lien
It contains a ganache configuration (with predefined accounts), three smart contracts, and the associated migration:
1. a dummy smart contract, with a big map as storage
2. a raffle smart contract, using a big map
3. a raffle smart contract, using a map

```shell
$ git clone
$ cd 
$ npm install -g truffle@tezos
$ npm install 
```

tzindex is written in Go. You can install go by following the instructions at:
[https://golang.org/doc/install](https://golang.org/doc/install)

There are also docker images. You can install Docker by following the instruction at:
[https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
## Launching a private tezos network
In the package.json, one script is defined:
```json
  "scripts": {
    "start-sandbox": "ganache-cli --flavor tezos --seed alice"
  }
```

> There are several versions of the ganache-cli. 
> Only the version suffixed by '-tezos' support tezos blockchain.
> You can find the version used by the project in the  "devDependencies" section, in the package.json file.



This script starts up a private Tezos blockchain, with _ganache_:

```shell
$ npm run start-sandox
Ganache CLI v6.12.1-tezos.0 (ganache-core: 2.13.2-tezos.2)

Available Accounts
==================
alice 100 TEZ
  pk: edpkvGfYw3LyB1UcCahKQk4rF2tvbMUk8GFiTuMjL75uGXrpvKXhjn
  pkh: tz1VSUr8wwNhLAzempoch5d6hLRiTh8Cjcjb
bob 100 TEZ
  pk: edpkurPsQ8eUApnLUJ9ZPDvu98E8VNj4KtJa1aZr16Cr5ow5VHKnz4
  pkh: tz1aSkwEot3L2kmUvcoxzjMomb9mvBNuzFK6
eve 100 TEZ
  pk: edpku9qEgcyfNNDK6EpMvu5SqXDqWRLuxdMxdyH12ivTUuB1KXfGP4
  pkh: tz1MnmtP4uAcgMpeZN6JtyziXeFqqwQG6yn6
mallory 100 TEZ
  pk: edpkujwsG5JMrWVXQwmRMBoR9yJkokRbn6wy3monpQKBpnZTs1ogRR
  pkh: tz1R2oNqANNy2vZhnZBJc8iMEqW79t85Fv7L
trent 100 TEZ
  pk: edpkukjpYWLPEavoTXid8KqfYAY4J1rqtbWHWrzp22FgdRQqzaRkDD
  pkh: tz1TfRXkAxbQ2BFqKV2dF4kE17yZ5BmJqSAP
marketa 100 TEZ
  pk: edpktiaGKkt8Yu6m4Gse2GhMdJCVdCtcrjtqATn3y3sf7xTBDj5g2a
  pkh: tz1fhigafd7PQAh3JBvq7efZ9g6cgBkaimJX
eulalie 100 TEZ
  pk: edpkvCvic2obeedM7oMJaeyapEafg4dSdYuWvkZigKbcvc64q6ZKM7
  pkh: tz1fEqJ6rD3mfQjVat7G6XJP522V6V8wWTP2
stella 100 TEZ
  pk: edpkvRuciP6vZeoXn1KJtBuEEtaD2SpHW59wbbCGt1SEDBL94W7AUE
  pkh: tz1i3eqdPNs9zjpavVBFcF8JarJUgEErfsUK
carline 100 TEZ
  pk: edpktxefxf3dtJEQLVb72MjV8yMiLh6DfCgNJQUV81rnsYJoZhbnK8
  pkh: tz1PQP815EzZRktgLaEhtDCR22kiRrcQEurw
tabbie 100 TEZ
  pk: edpkvXobE6tQLdsm3kPu3MYT2z2XrgVchfkK2WPB9tniNXdWSRyud3
  pkh: tz1WP3xUvTP6vUWLRnexxnjNTYDiZ7QzVdxo

<...>
```

A private network has started, with funded accounts that will be used in this article.

## Migrating the raffle contract

The contract can be migrated onto the private network. 
More details can be found about the Truffle tool in the Dapp section (link to chapter).

First, the private network should be defined in the _truffle-config.js_ file.
A development subsection should be found under the networks section:
```json
    development: {
      host: "http://localhost",
      port: 8732,
      network_id: "*",
      secretKey: alice.sk,
      type: "tezos"
    }
```
<br/>
The used account is defined in the scripts/sandbox/account.js file.
The contracts can be migrated:

```shell
$ truffle migrate --network development
```

The contracts are deployed onto our private network:
1. the first contract holds a big map, and does not do anything.
This dummy contract is deployed to bypass a tzstat bug regarding big maps: 
the first big map (whose index is 0) are not fetched by the frontend
2. a raffle contract using map
3. a raffle using big map

For the second and third migrations, a raffle is opened (given the storage definition), 
and the account used for the migration has bought a ticket.


# Setting up a private tzindex (backend)

tzstat is an open-source indexer: it can freely be used and modified.
The source code is available here: [https://github.com/blockwatch-cc/tzindex](https://github.com/blockwatch-cc/tzindex)

The application will be built:
```shell
$ git clone https://github.com/blockwatch-cc/tzindex
$ cd tzindex
$ make build
$ ls tzindex
tzindex

```
A tzindex binary is created into the directory.
The indexer can now watch the private network with:
```shell
$ ./tzindex run --rpcurl 127.0.0.1:8732 --notls --enable-cors
```

- --rpcurl: url of the ganache private network rpc node
- --notls: this option is used since ganache is exposing with http
- --enable-cors: used for the frontend

tzindex will expose its API over http://localhost:8000.

A db/ folder is created when launching tzindex for the first time: 
it contains all the data indexed about the private network. 

> When restarting ganache, a blank new network is created. 
> The db/ folder have to be removed, in order to make tzindex index the new network.

# Setting up a private tzstat (frontend)

tzstat is an open-source indexer: it can freely be used and modified.
The source code is available here: [https://github.com/blockwatch-cc/tzstats](https://github.com/blockwatch-cc/tzstats)

This frontend works with tzindex: 
tzstat can interact with tzindex by setting the TZSTATS_API_URL variable
The application can be launched with:
```shell
$ git clone https://github.com/blockwatch-cc/tzstats
$ cd tzstats
$ echo 'TZSTATS_API_URL=http://localhost:8000' > development.env
$ npm install
$ yarn start
```

> An error might occur during the npm install. 
> If so, modify the `sass` field under the `scripts` in the package.json file:
> ```json
> "sass": "npx sass src/styles/scss/index.scss:src/styles/css/index.css"
> ```
> Run again the `npm install` command

A new page should open in your default web browser.

# Interactions with the private explorer

At this point, the  working context is:
- a private network running, through Ganache
- three migrated contracts
- two contract calls 
- tzindex, indexing the private network
- tzstat, connected to tzindex, and running

So, there has already been some activity within the private network. 

## Watch activity from tzstats

Go to http://localhost:3000 with your web browser.

![](../../static/img/explorer/tzstat-1.png "Welcome page of a private tzstat")

In the top-center, there is a search bar where anything can be looked for:
transactions, blocks, addresses...

On the left panel, various pieces of information are displayed.
One of them is going through each baked block. The number of block can be clicked.

So far, four blocks should have been baked:
![](../../static/img/explorer/tzstat-2.png "Block details page")

All block details are displayed, many of them can be clicked.
This block only contains one operation, which is a smart contract call.
The hash can be clicked, the operation page opens up.

![](../../static/img/explorer/tzstat-3.png "Smart contract call details page")

The sender does match alice pkh from the scripts/sandbox/account.js (in the truffle project).
The contract address does also match the returned address from the migration.

The contract can be inspected by clicking on its address:
![](../../static/img/explorer/tzstat-4.png "Smart contract details page")

All the pieces of information can be found here: the history of calls, the entrypoints, the storage...
The two calls made by Truffle are displayed: the origination of the contract, and the purchase of a ticket.

The storage page displays all the pieces of data of the smart contract: the storage definition, the type of each field,
the data held in each field.
Below is the smart contract storage after the migration: one participant is registered, 
and the contract holds one XTZ (ticket price).
![](../../static/img/explorer/tzstat-5.png "Smart contract storage page")

### About big maps
However, the *sold_tickets* big map should hold addresses as values, but it displays 1 in the storage.
Since big maps are meant to hold unbounded lists of data, it is not loaded directly into the storage.
Each big map is indexed: the number 1 that is seen is the big map number.
The data can be accessed by clicking on the "Bigmap 1" section.

![](../../static/img/explorer/tzstat-6.png "Big map section")

If maps (meant for limited data size) are used, the data is directly retrieved in the storage section.
You can go and watch the second migrated data storage for instance (baked in the fourth block):

![](../../static/img/explorer/tzstat-7.png "Storage page")

The tzstat interface is very user-friendly, all kind of information can be easily read by clicking on element.
All these pieces of information are of course retrieved from the indexer: tzstat just displays them.



## Watch activity from tzindex

In this part, the same pieces of information will be retrieved by just using the API, without the frontend.
On each page, an API call is made: each one of these API calls can be seen in the network explorer.
The network explorer can be opened by pressing F12, in the network section

![](../../static/img/explorer/tzstat-7.png "Retrieving the API call")

The _request URL_ gives the API endpoint to call:

```shell
$ GET http://127.0.0.1:8000/explorer/contract/KT1HJ8VJ9rHkoi4FfzHPburSe1VdYn8AU4AF?
{
  "address": "KT1HJ8VJ9rHkoi4FfzHPburSe1VdYn8AU4AF",
  "manager": "tz1VSUr8wwNhLAzempoch5d6hLRiTh8Cjcjb",
  "delegate": "",
  "height": 6,
  "fee": 0.003939,
  "gas_limit": 12826,
  "gas_used": 12726,
  "gas_price": 0.30952,
  "storage_limit": 2654,
  "storage_size": 2397,
  "storage_paid": 2397,
  "is_funded": true,
  "is_vesting": false,
  "is_spendable": false,
  "is_delegatable": false,
  "is_delegated": false,
  "first_in": 7,
  "first_out": 0,
  "last_in": 7,
  "last_out": 0,
  "first_seen": 6,
  "last_seen": 7,
  "delegated_since": 0,
  "first_in_time": "2021-04-17T17:06:27Z",
  "first_out_time": "0001-01-01T00:00:00Z",
  "last_in_time": "2021-04-17T17:06:27Z",
  "last_out_time": "0001-01-01T00:00:00Z",
  "first_seen_time": "2021-04-17T17:06:26Z",
  "last_seen_time": "2021-04-17T17:06:27Z",
  "delegated_since_time": "0001-01-01T00:00:00Z",
  "n_ops": 1,
  "n_ops_failed": 0,
  "n_tx": 1,
  "n_delegation": 0,
  "n_origination": 0,
  "token_gen_min": 1,
  "token_gen_max": 1,
  "bigmap_ids": [
    1
  ],
  "op_l": 3,
  "op_p": 0,
  "op_i": 0,
  "iface_hash": "d30a2146",
  "call_stats": [
    1,
    0,
    0
  ]
}


```

It gives a lot of information: some of them are displayed on the page (such as the address).
However, some others are missing. For instance, the storage data is not fetched.
Another API call has to be made:

```shell
$ GET http://127.0.0.1:8000/explorer/contract/KT1HJ8VJ9rHkoi4FfzHPburSe1VdYn8AU4AF/storage?
{
  "meta": {
    "contract": "KT1HJ8VJ9rHkoi4FfzHPburSe1VdYn8AU4AF",
    "time": "2021-04-17T17:06:27Z",
    "height": 7,
    "block": "BLwCojKb2fv7Ph88kxMoYXeTRYdtavvzCNh3ZzsJNpfFgqq5ey8"
  },
  "value": {
    "admin": "tz1cGftgD3FuBmBhcwY24RaMm5D2UXLr5LHW",
    "close_date": "1618679185827",
    "contract_name": "Raffle smart contract with big map",
    "description": "",
    "jackpot": "100",
    "players": [
      "tz1VSUr8wwNhLAzempoch5d6hLRiTh8Cjcjb"
    ],
    "raffle_is_open": "true",
    "sold_tickets": "1",
    "winning_ticket_number_hash": "00"
  }
}

```

You can find all the available endpoints here: [https://tzstats.com/docs/api#explorer-endpoints](https://tzstats.com/docs/api#explorer-endpoints)

# Setting up a private indexer for a public network

Public networks can be also be monitored with a local indexer.
It just has to monitor a node: either a local node

```shell
$ ./tzindex run --rpcurl 127.0.0.1:8732 --notls --enable-cors
```


# Conclusion

Running a private network does not mean being blind to what is going on the network:
private indexers and explorers can be set up to monitor what is going on a private network.

Tools are already set up for public networks, but private tools are also be used to that purpose.

Finally, indexers can come in handy as additional tools to taquito or wallet (as described in the Dapp module).
Indeed, big maps are not easily handled with those tools.
For instance, it is not possible to retrieve all the keys of a big map.
However, indexers solve this issue with a simple REST call.