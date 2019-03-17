# Insight API

A LUX blockchain REST and web socket API service for [Luxcore Node](https://github.com/LUX-Core/luxcore-node).

This is a backend-only service. If you're looking for the web frontend application, take a look at https://github.com/216k155/lux-explorer.

## Getting Started

1. Install nvm https://github.com/creationix/nvm  

    ```bash
    nvm i v6
    nvm use v6
    ```  
2. Install mongo https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/  

3. Install lux-bitcore https://github.com/216k155/lux-bitcore - with ZMQ !

    ```bash
    # with ZMQ
    sudo apt-get install libzmq3-dev 
    ```  
4. Install luxcore-node

    ```bash
    npm i https://github.com/LUX-Core/luxcore-node.git#master

    $(npm bin)/luxcore-node create mynode

    cd mynode

    $(npm bin)/luxcore-node install https://github.com/LUX-Core/insight-api.git#master
    ```  
5. Edit luxcore-node.json

    ```json
    {
      "network": "livenet",
      "port": 3001,
      "services": [
        "luxd",
        "lux-insight-api",
        "web"
      ],
      "servicesConfig": {
        "lux-insight-api": {
          "routePrefix": "lux-insight-api",
          "rateLimiterOptions": {
          "whitelist": [
             "123.456.12.34",
             "::ffff:123.456.12.34"
           ],
          "whitelistLimit": 9999999,
          "limit": 200,
          "interval": 60000,
          "banInterval": 3600000
          },
          "db": {
            "host": "127.0.0.1",
            "port": "27017",
            "database": "lux-api-livenet",
            "user": "",
            "password": ""
          },
          "erc20": {
            "updateFromBlockHeight": 0
          }
        },
        "luxd": {
          "spawn": {
        	  "datadir": "/home/user/.lux",
            "exec": "/home/user/lux-bitcore/src/luxd"
          }
        }
      }
    }
    ```  
6. Edit lux.conf

    ```
    server=1
    whitelist=127.0.0.1
    txindex=1
    addressindex=1
    timestampindex=1
    spentindex=1
    zmqpubrawtx=tcp://127.0.0.1:28332
    zmqpubhashblock=tcp://127.0.0.1:28332
    rpcallowip=127.0.0.1
    rpcuser=user
    rpcpassword=password
    rpcport=9888
    reindex=1
    gen=0
    addrindex=1
    logevents=1
    ```  
7. Run Node  

    ```
    $(npm bin)/luxcore-node start
    ```  

8. The API endpoints will be available by default at: `http://localhost:3001/lux-insight-api/`

## Add-on Services

There add-on service available to extend the functionality of Luxcore:

- [LUX Explorer](https://github.com/216k155/lux-explorer)

## Prerequisites

**Note:** You can use an existing LUX data directory, however `txindex`, `addressindex`, `timestampindex` and `spentindex` needs to be set to true in `lux.conf`, as well as a few other additional fields.


## Query Rate Limit

To protect the server, lux-insight-api has a built it query rate limiter. It can be configurable in `luxcore-node.json` with:
``` json
  "servicesConfig": {
    "lux-insight-api": {
      "rateLimiterOptions": {
        "whitelist": ["::ffff:127.0.0.1"]
      }
    }
  }
```

Or disabled entirely with:
``` json
  "servicesConfig": {
    "lux-insight-api": {
      "disableRateLimiter": true
    }
  }
  ```
  
**Note:** `routePrefix` can be configurable in `luxcore-node.json` with:

``` json
  "servicesConfig": {
    "lux-insight-api": {
      "routePrefix": "insight-api",
    }
  }
  ```



## Tokens

### Token Account Balance

```
  `GET` /insight-api/tokens/{:tokenAddressBase}/addresses/{:addressBase}/balance
```
or
```
  `GET` /insight-api/tokens/{:tokenAddressBase}/addresses/{:addressBase}/balance?format=object
```

* **Query Params**

    * **Optional:**
        
            `format=object`
            
This would return:

```
1000000000000000000
```

```
{
    "balance": "1000000000000000000"
}
```



### Token Total supply
```
    `GET` /insight-api/tokens/{:tokenAddressBase}/total-supply
```
or
```
  `GET` /insight-api/tokens/{:tokenAddressBase}/total-supply?format=object
```

* **Query Params**

    * **Optional:**
        
            `format=object`
            
This would return:

```
"1000000000000000000"
```
or
```
{
    "total_supply": "1000000000000000000"
}
```

### Token Transactions

```
  `GET` /insight-api/tokens/{:tokenAddressBase}/transactions
```

* **Query Params**

    * **Optional:**
        
            `limit=<Number>`
            
                > MAX_LIMIT === 100
            
            `offset=<Number>`
            
            `from_block=<Number>`
            
            `to_block=<Number>`
            
            `from_date_time=<ISO8601 Date>`
            
            `to_date_time=<ISO8601 Date>`
            
            `addresses=<Array.<String>>`


E.g.:
```
  `GET` /insight-api/tokens/LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm/transactions?limit=20&offset=1&from_block=34101&to_block=34378&from_date_time=2017-10-27T01:23:10.000Z&to_date_time=2018-10-27T01:24:10.000Z&addresses[]=QbmrFnBhyMKUhrabXfaAWZTncSWbJA8FsG&addresses[]=QarHW2HjV8Z3njxiTuvUZU3hmqahKNZ49y
```

This would return:
```
{
    "limit": 20,
    "offset": 1,
    "addresses": [
        "LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm",
        "LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm"
    ],
    "from_block": 34101,
    "to_block": 34378,
    "from_date_time": "2017-10-27T01:23:10.000Z",
    "to_date_time": "2018-10-27T01:24:10.000Z",
    "count": 2,
    "items": [
        {
            "contract_address_base": "LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm",
            "block_height": 355214,
            "tx_hash": "e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a",
            "from": "LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm",
            "to": "LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm",
            "value": "18747",
            "block_date_time": "2018-08-09 15:43:56"
        }
    ]
}
```

## API HTTP Endpoints

### Account Info
```
  `GET` /insight-api/contracts/{:contractHash}/info
```
This would return:
```
{
    address: "LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm",
    balance: 0,
    storage: {
            03: "6f",
            00: "5374616e64617264000000000000000000000000000000000000000000000010",
            02: "5354440000000000000000000000000000000000000000000000000000000006",
            03c2cc9c95da7ca004141242c8b36d98e928596f771044fd5552d3ee8b47438f: "2710",
            04: "2710",
            01: "5374616e64617264200000000000000000000000000000000000000000000012"
        },
    code: "606060405236156100b8576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806306fdde03146100ce578063095ea7b31461016757806318160ddd146101be57806323b872dd146101e4578063313ce5671461025a5780633867ed23146102865780635a3b7e421461029857806370a082311461033157806395d89b411461037b578063a9059cbb14610414578063cae9ca5114610453578063dd62ed3e146104ed575b34156100c057fe5b6100cc5b60006000fd5b565b005b34156100d657fe5b6100de610556565b604051808060200182810382528381815181526020019150805190602001908083836000831461012d575b80518252602083111561012d57602082019150602081019050602083039250610109565b505050905090810190601f1680156101595780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b341561016f57fe5b6101a4600480803573ffffffffffffffffffffffffffffffffffffffff169060200190919080359060200190919050506105f4565b604051808215151515815260200191505060405180910390f35b34156101c657fe5b6101ce610682565b6040518082815260200191505060405180910390f35b34156101ec57fe5b610240600480803573ffffffffffffffffffffffffffffffffffffffff1690602001909190803573ffffffffffffffffffffffffffffffffffffffff16906020019091908035906020019091905050610688565b604051808215151515815260200191505060405180910390f35b341561026257fe5b61026a610984565b604051808260ff1660ff16815260200191505060405180910390f35b341561028e57fe5b610296610997565b005b34156102a057fe5b6102a86109ab565b60405180806020018281038252838181518152602001915080519060200190808383600083146102f7575b8051825260208311156102f7576020820191506020810190506020830392506102d3565b505050905090810190601f1680156103235780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b341561033957fe5b610365600480803573ffffffffffffffffffffffffffffffffffffffff16906020019091905050610a49565b6040518082815260200191505060405180910390f35b341561038357fe5b61038b610a61565b60405180806020018281038252838181518152602001915080519060200190808383600083146103da575b8051825260208311156103da576020820191506020810190506020830392506103b6565b505050905090810190601f1680156104065780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b341561041c57fe5b610451600480803573ffffffffffffffffffffffffffffffffffffffff16906020019091908035906020019091905050610aff565b005b341561045b57fe5b6104d3600480803573ffffffffffffffffffffffffffffffffffffffff1690602001909190803590602001909190803590602001908201803590602001908080601f01602080910402602001604051908101604052809392919081815260200183838082843782019150505050505091905050610cde565b604051808215151515815260200191505060405180910390f35b34156104f557fe5b610540600480803573ffffffffffffffffffffffffffffffffffffffff1690602001909190803573ffffffffffffffffffffffffffffffffffffffff16906020019091905050610e65565b6040518082815260200191505060405180910390f35b60018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156105ec5780601f106105c1576101008083540402835291602001916105ec565b820191906000526020600020905b8154815290600101906020018083116105cf57829003601f168201915b505050505081565b600081600660003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002081905550600190505b92915050565b60045481565b600081600560008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205410156106d75760006000fd5b600560008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205482600560008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020540110156107655760006000fd5b600660008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020548211156107ef5760006000fd5b81600560008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254039250508190555081600560008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254019250508190555081600660008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600082825403925050819055508273ffffffffffffffffffffffffffffffffffffffff168473ffffffffffffffffffffffffffffffffffffffff167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef846040518082815260200191505060405180910390a3600190505b9392505050565b600360009054906101000a900460ff1681565b60646004600082825401925050819055505b565b60008054600181600116156101000203166002900480601f016020809104026020016040519081016040528092919081815260200182805460018160011615610100020316600290048015610a415780601f10610a1657610100808354040283529160200191610a41565b820191906000526020600020905b815481529060010190602001808311610a2457829003601f168201915b505050505081565b60056020528060005260406000206000915090505481565b60028054600181600116156101000203166002900480601f016020809104026020016040519081016040528092919081815260200182805460018160011615610100020316600290048015610af75780601f10610acc57610100808354040283529160200191610af7565b820191906000526020600020905b815481529060010190602001808311610ada57829003601f168201915b505050505081565b80600560003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020541015610b4c5760006000fd5b600560008373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205481600560008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002054011015610bda5760006000fd5b80600560003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254039250508190555080600560008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600082825401925050819055508173ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef836040518082815260200191505060405180910390a35b5050565b60006000849050610cef85856105f4565b15610e5c578073ffffffffffffffffffffffffffffffffffffffff16638f4ffcb1338630876040518563ffffffff167c0100000000000000000000000000000000000000000000000000000000028152600401808573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018481526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200180602001828103825283818151815260200191508051906020019080838360008314610df8575b805182526020831115610df857602082019150602081019050602083039250610dd4565b505050905090810190601f168015610e245780820380516001836020036101000a031916815260200191505b5095505050505050600060405180830381600087803b1515610e4257fe5b6102c65a03f11515610e5057fe5b50505060019150610e5d565b5b509392505050565b60066020528160005260406000206020528060005260406000206000915091505054815600a165627a7a72305820ad65da1f64921dfd4eebad40d0b4c8f4a800ccd86080446b7ea98cbc56f244f80029",
    vins: [
        {
            hash: "e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a",
            vout: 0,
            amount: 0
        }
    ]
}
```

## Statistics

### Total 24h Statistic
```
  `GET` /insight-api/statistics/total
```
This would return:
```
    {
        n_blocks_mined: 1268,
        time_between_blocks: 86301,
        mined_currency_amount: 1268000000000000,
        transaction_fees: 633992859997,
        number_of_transactions: 2547,
        outputs_volume: 46920965480,
        difficulty: 981808167.7687966,
        stake: 0.17279912782774598
    }
```
### Transactions Statistic
```
  `GET` /insight-api/statistics/transactions?days=14
```
This would return:
```
[
    {
        date: "2018-08-10",
        transaction_count: 1087,
        block_count: 541
    },
    ...
]
```

### Fees Statistic
```
  `GET` /insight-api/statistics/fees?days=14
```
This would return:
```
[
   {
       date: "2018-08-10",
       fee: 500000000
   },
   ...
]
```
### Outputs Statistic
```
  `GET` /insight-api/statistics/outputs?days=14
```
This would return:
```
[
   {
       date: "2018-08-10",
       sum: 0
   },
   ...
]
```
### Difficulty Statistic
```
  `GET` /insight-api/statistics/difficulty?days=14
```
This would return:
```
[
    {
        date: "2018-08-10",
        sum: 0
    },
    ...
]
```
### Stake Statistic
```
  `GET` /insight-api/statistics/stake?days=14
```
This would return:
```
[
    {
        date: "2018-08-10",
        sum: 0
    },
    ...
]
```

### Total Supply Statistic

```
  `GET` /insight-api/circulating-supply
```
or
```
  `GET` /insight-api/circulating-supply/?format=object
```
This would return:
```
5100000
```
or
```
{
"circulatingSupply": "5100000"
}
```


```
  `GET` /insight-api/supply
```
or
```
  `GET` /insight-api/supply?format=object
```
This would return:
```
6000000
```
or
```
{
    "supply": "6000000"
}
```

### Block
```
  /insight-api/block/[:hash]
  /insight-api/block/e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a
```

### Block Index
Get block hash by height
```
  /insight-api/block-index/[:height]
  /insight-api/block-index/0
```
This would return:
```
{
  "blockHash":"e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a"
}
```
which is the hash of the Genesis block (0 height)


### Raw Block
```
  /insight-api/rawblock/[:blockHash]
  /insight-api/rawblock/[:blockHeight]
```

This would return:
```
{
  "rawblock":"blockhexstring..."
}
```

### Block Summaries

Get block summaries by date:
```
  /insight-api/blocks?limit=3&blockDate=2018-08-10
```

Example response:
```
{
  "blocks": [
    {
      "height": 408495,
      "size": 355214,
      "hash": "e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a",
      "time": 1533829436,
      "txlength": 1695,
      "poolInfo": {
        "poolName": "N/A",
        "url": "N/A"
      }
    }
  ],
  "length": 1,
  "pagination": {
    "next": "2018-08-10",
    "prev": "2018-08-10",
    "currentTs": 1533829436,
    "current": "2018-08-10",
    "isToday": true,
    "more": true,
    "moreTs": 1533829436
  }
}
```

### Transaction
```
  /insight-api/tx/[:txid]
  /insight-api/tx/e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a
  /insight-api/rawtx/[:rawid]
  /insight-api/rawtx/e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a
```

### Address
```
  /insight-api/addr/[:addr][?noTxList=1][&from=&to=]
  /insight-api/addr/LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm?noTxList=1
  /insight-api/addr/LVfn9yjwarsjLfgEbB9tCwNzuaLtwD3dUm?from=1000&to=2000
```

### Address Properties
```
  /insight-api/addr/[:addr]/balance
  /insight-api/addr/[:addr]/totalReceived
  /insight-api/addr/[:addr]/totalSent
  /insight-api/addr/[:addr]/unconfirmedBalance
```
The response contains the value in Satoshis.

### Unspent Outputs
```
  /insight-api/addr/[:addr]/utxo
```

### Historic Blockchain Data Sync Status
```
  /insight-api/sync
```

### Live Network P2P Data Sync Status
```
  /insight-api/peer
```

### Status of the LUX Network
```
  /insight-api/status?q=xxx
```

### DGP info
```
  /insight-api/dgpinfo
```

Where "xxx" can be:

 * getInfo
 * getDifficulty
 * getBestBlockHash
 * getLastBlockHash


### Utility Methods
```
  /insight-api/utils/estimatefee[?nbBlocks=2]
```

### Min Estimate Fee Per KB
```
  /insight-api/utils/minestimatefee[?nbBlocks=2]
```

resp:

```
{
    fee_per_kb: 0.00001
}
```

### QRC20 info
```
  /insight-api/erc20/:contractAddress
  > DEPRECATED
```
    
```
  /insight-api/qrc20/:contractAddress
```

### QRC20 transfers
```
  /insight-api/erc20/:contractAddress/transfers
```

### QRC20 balances
```
  /insight-api/erc20/:contractAddress/balances
```

### Call Contract
```
/insight-api/contracts/:contractaddress/hash/:contracthash/call
```

## Web Socket API

The web socket API is served using [socket.io](http://socket.io).

The following are the events published by insight:

`status`: every 1% increment on the sync task, this event will be triggered. This event is published in the `sync` room.

Sample output:
```
{
  blocksToSync: 355214,
  syncedBlocks: 475,
  upToExisting: true,
  scanningBackward: true,
  isEndGenesis: true,
  end: "e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a",
  isStartGenesis: false,
  start: "e5e9dd2677ef7e0904efb9bbe7e9af8d7753dc5cf869ef0c7aadcc4876f8d88a"
}
```

### Example Usage

The following html page connects to the socket.io insight API and listens for new transactions.

html
```
<html>
<body>
  <script src="http://<insight-server>:<port>/socket.io/socket.io.js"></script>
  <script>
    eventToListenTo = 'tx'
    room = 'inv'

    var socket = io("http://<insight-server>:<port>/");
    socket.on('connect', function() {
      // Join the room.
      socket.emit('subscribe', room);
    })
    socket.on(eventToListenTo, function(data) {
      console.log("New transaction received: " + data.txid)
    })
  </script>
</body>
</html>
```

## License
(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
