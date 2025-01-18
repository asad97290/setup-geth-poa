# Setup Geth POA


## forked from 
- https://github.com/ethereum/go-ethereum.git
- git checkout tags/v1.13.2
## install go
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.2.linux-amd64.tar.gz

## Building the source

Building geth requires both a Go (version 1.23 or later) and a C compiler. You can install them using your favourite package manager. Once the dependencies are installed, run


```shell
make geth
```

or, to build the full suite of utilities:

```shell
make all
```

### Hardware Requirements

Minimum:

* CPU with 2+ cores
* 4GB RAM
* 1TB free storage space to sync the Mainnet
* 8 MBit/sec download Internet service

Recommended:

* Fast CPU with 4+ cores
* 16GB+ RAM
* High-performance SSD with at least 1TB of free space
* 25+ MBit/sec download Internet service


## generate new account
`./build/bin/geth account new --datadir ./my-bootnode `

` ./build/bin/geth account new --datadir ./node1 `

` ./build/bin/geth account new --datadir ./node2 `


## Defining the genesis state

First, you'll need to create the genesis state of your networks, which all nodes need to be
aware of and agree upon. This consists of a small JSON file (e.g. call it `genesis.json`)

Note:
extraData: This is normally optional, but mandatory with clique! It configures the network and enables signers. It contains three parts, lets break it down:

32 bytes vanity data. That means you can put into the first 32 bytes whatever you want. In our case its all zeros
0x0000000000000000000000000000000000000000000000000000000000000000
The concatenated addresses from the sealers of the clique network. It's written as hex-string without the "0x" at the beginning. It must be a multiple of 20 bytes
0001fcd01073fe6017920a97cc384bee72c98beb
0002f7067c48ef957b21009685ab69ee768e38bd
The last part is a 65 bytes proposer seal, which is 65 bytes long (or 130 characters), in hex notation without the 0x. It's all zeroes in the genesis block because there are no proposers yet.
0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
You can read more about it in EIP-650
All combined is 0x00000000000000000000000000000000000000000000000000000000000000000001fcd01073fe6017920a97cc384bee72c98beb0002f7067c48ef957b21009685ab69ee768e38bd0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

```json
{
    "config": {
        "chainId": 271997,
        "homesteadBlock": 0,
        "eip150Block": 0,
        "eip155Block": 0,
        "eip158Block": 0,
        "byzantiumBlock": 0,
        "constantinopleBlock": 0,
        "petersburgBlock": 0,
        "istanbulBlock": 0,
        "berlinBlock": 0,
        "clique": {
            "period": 6,
            "epoch": 30000
        }
    },
    
    "timestamp": "0x00",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x000000000000000000000000000000000000000000000000000000000000000051e25A3F59Dc7fb1e1eb27250250a9255Cd373F50000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "difficulty":"0x000000",
    "gasLimit": "0x1c9c380",
    "nonce":"0x0000000000000042",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "alloc": {
        "51e25A3F59Dc7fb1e1eb27250250a9255Cd373F5": {
            "balance": "0x3635c9adc5dea00000"
        }
    },
    "feePerTx":1,
    "proposedFee":1,
    "votes":0
}
```

With the genesis state defined in the above JSON file, you'll need to initialize **every**
`geth` node with it prior to starting it up to ensure all blockchain parameters are correctly
set:



HTTP based JSON-RPC API options:

  * `--http` Enable the HTTP-RPC server
  * `--http.addr` HTTP-RPC server listening interface (default: `localhost`)
  * `--http.port` HTTP-RPC server listening port (default: `8545`)
  * `--http.api` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
  * `--http.corsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--ws.addr` WS-RPC server listening interface (default: `localhost`)
  * `--ws.port` WS-RPC server listening port (default: `8546`)
  * `--ws.api` API's offered over the WS-RPC interface (default: `eth,net,web3`)
  * `--ws.origins` Origins from which to accept WebSocket requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,debug,eth,miner,net,personal,txpool,web3`)
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)


## initialize genesis block
` ./build/bin/geth --datadir ./my-bootnode init ./genesis.json   `
` ./build/bin/geth --datadir ./node1 init ./genesis.json   `
` ./build/bin/geth --datadir ./node2 init ./genesis.json   `

## protact key using clef

1. `clef --keystore ./node1/keystore --configdir ./clef-node1 --chainid 271997 --suppress-bootwarn init`

2. `clef --keystore ./node1/keystore --configdir ./clef-node1 --chainid 271997 --suppress-bootwarn setpw 0xB02aDdbbc1fCACd3abB30513A3E552f6469EE7D4   `

3. ``clef --keystore ./node1/keystore --configdir ./clef-node1 --chainid 271997 --suppress-bootwarn  attest  `sha256sum rules.js | cut -f1`
``
4. `clef --keystore ./node1/keystore --configdir ./clef-node1 --chainid 271997 --suppress-bootwarn --rules ./rules.js`




#### Creating the rendezvous point

With all nodes that you want to run initialized to the desired genesis state, you'll need to
start a bootstrap node that others can use to find each other in your network and/or over
the internet. The clean way is to configure and run a dedicated bootnode:

```shell
$ ./build/bin/bootnode --genkey=boot.key
$ ./build/bin/bootnode --nodekey boot.key -addr $(hostname -I | awk '{print $1}' ):30305 --verbosity=3
```

*Note: You could also use a full-fledged `geth` node as a bootnode, but it's the less
recommended way for development deployments. We recommend using a regular node as bootstrap node for production deployments.*

## run bootnode as regular geth node
` ./build/bin/geth --networkid=271997 --datadir ./my-bootnode --port 30305 --authrpc.port 8553 `



## start node1 as miner
 `./build/bin/geth --bootnodes enode://95459190146d262f8d0153e6db52c19e0bc34bab214e9103c869a4620139ea6f349b36eff33b729f9499f92f13e56fe5f3b8c5924c4bfa6245d600d92e5b8e36@172.31.1.190:30305 --ws --ws.addr 0.0.0.0 --ws.port 8545  --ws.api eth,net,web3,personal --http --http.addr 0.0.0.0 --http.port 8545 --http.corsdomain "*" --http.api eth,net,web3,personal --networkid=271997 --datadir ./node1 --password ./password.txt --port 30304  --authrpc.port 8551 --miner.etherbase=0x51e25A3F59Dc7fb1e1eb27250250a9255Cd373F5 --mine --unlock 0x51e25A3F59Dc7fb1e1eb27250250a9255Cd373F5 -allow-insecure-unlock --syncmode full`



## start node2
 `./build/bin/geth --bootnodes enode://672267382b4ae546a471b5cf3984e91e7024b8d46a67788fc3d898bf7528a4b001fa9a5887b32088086d16f74fa4ebc09cb557ac33015efd6a5fed4b2faccc7b@192.168.18.16:30305 --ws --ws.addr 0.0.0.0 --ws.port 8545  --ws.api eth,net,web3,personal --http --http.addr 0.0.0.0 --http.port 8545 --http.corsdomain "*" --http.api eth,net,web3,personal --networkid=271997 --datadir ./node1 --signer ./clef/clef.ipc --port 30306  --authrpc.port 8552 --miner.etherbase=0xeF008F3ECE189110d25cBeAbE3fE7183E767fF80 --mine --unlock 0xeF008F3ECE189110d25cBeAbE3fE7183E767fF80 -allow-insecure-unlock --syncmode full`


## connect to geth JavaScript console
` ./build/bin/geth attach ./node1/geth.ipc   `   


## send tx
`eth.sendTransaction({from:eth.accounts[0],to:"0xFD01A2868caACaceB32636fa8A7391f732689Ef9",value:web3.toWei("50000000000")})`
## check balance
`web3.fromWei(eth.getBalance("0xb420eA2C43AcabbA17881E49957731B53fc2a5fd"))`



## rpc url
`http://127.0.0.1:8545`


## check signer in console
`clique.getSigners()`

## check node info in console
`admin.nodeInfo`

`admin.peers`
## add new sealer/miner/validator
`clique.propose("0x76da2973add8f149c31c571ad379785528c91fb9",true)`


## References

https://ethereum-blockchain-developer.com/123-geth-clique-blockscout/00-overview/



## Test using Curl request
``` 
curl http://127.0.0.1:8545 \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{"method":"eth_getBalance","params":["0x51e25A3F59Dc7fb1e1eb27250250a9255Cd373F5", "latest"],"id":1,"jsonrpc":"2.0"}'



  curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest", true],"id":1}' -H "Content-Type: application/json" http://127.0.0.1:8545       
```





## blockscout backend env

```bash
export DATABASE_URL=postgresql://eth-node-db:1234@localhost:5432/blockscout
export SECRET_KEY_BASE=L4CC+3JMhUaJN2phtsqRwyCeSj8qOoUnagH43DrwCzgX320WtNCWUlnWmqBoZcck
export ETHEREUM_JSONRPC_VARIANT=geth
export ETHEREUM_JSONRPC_HTTP_URL=https://rpc.agensensus.ai
export API_V2_ENABLED=true
export PORT=3001 # set for local API usage
export COIN=AgenSensus
export COIN_NAME=AGS
export DISPLAY_TOKEN_ICONS=true
export MICROSERVICE_SC_VERIFIER_ENABLED=true
export MICROSERVICE_SC_VERIFIER_URL=http://localhost:8082/
export MICROSERVICE_VISUALIZE_SOL2UML_ENABLED=true
export MICROSERVICE_VISUALIZE_SOL2UML_URL=http://localhost:8081/
export MICROSERVICE_SIG_PROVIDER_ENABLED=true
export MICROSERVICE_SIG_PROVIDER_URL=http://localhost:8083/
export METADATA_CONTRACT=all
```


## blockscout frontend env
```bash
NEXT_PUBLIC_API_HOST=192.168.18.16
NEXT_PUBLIC_API_PORT=3001
NEXT_PUBLIC_API_PROTOCOL=http
NEXT_PUBLIC_STATS_API_HOST=http://192.168.18.16:8080
NEXT_PUBLIC_VISUALIZE_API_HOST=http://192.168.18.16:8081
NEXT_PUBLIC_APP_HOST=192.168.18.16
NEXT_PUBLIC_APP_PORT=3000
NEXT_PUBLIC_APP_INSTANCE=192.168.18.16
NEXT_PUBLIC_APP_ENV=development
NEXT_PUBLIC_API_WEBSOCKET_PROTOCOL='ws'
NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID=<your-secret>
NEXT_PUBLIC_NETWORK_CURRENCY_NAME=MyChain 
NEXT_PUBLIC_NETWORK_CURRENCY_SYMBOL=MC
NEXT_PUBLIC_NETWORK_CURRENCY_DECIMALS=18
NEXT_PUBLIC_NETWORK_NAME="My Chain"
NEXT_PUBLIC_NETWORK_ID=271997
NEXT_PUBLIC_IS_TESTNET=true
NEXT_PUBLIC_AD_BANNER_PROVIDER="none"
NEXT_PUBLIC_NETWORK_LOGO=17a7bd3b.svg

NEXT_PUBLIC_NETWORK_LOGO_DARK=17a7bd3b.svg


NEXT_PUBLIC_NETWORK_ICON=K22E.png


NEXT_PUBLIC_NETWORK_ICON_DARK=K22E.png
```
