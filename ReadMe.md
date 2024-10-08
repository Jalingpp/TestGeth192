# Jalingpp/TestGeth192
A tutorial on "Building and Maintaining Private Blockchains with go-ethereum-1.9.2".

## 1 Download Go-ethereum-1.9.2
The installation package is `go-ethereum-1.9.2.zip`, or download other releases from `https://github.com/ethereum/go-ethereum/releases`.

Building geth requires both a Go (version 1.22 or later) and a C compiler. You can install them using your favourite package manager. Once the dependencies are installed, run `make geth`.

You can change the source codes and rebuild.

## 2 Build a Private Blockchains
Step 1. Configure the execution path of geth: `cd go-ethereum-1.9.2` and then `cp ./build/bin/geth /usr/local/bin/`. Find the geth version with `geth --version`.

Step 2. New a folder for nodes beside go-ethereum-1.9.2: `mkdir ethnodes`.

Step 3. Generate `genesis.json` File in `ethnodes`. Copy the following content into the file.

```genesis.json
{
    "config": {
      "chainId": 200,
      "homesteadBlock": 0,
      "eip155Block": 0,
      "eip158Block": 0
    },
    "alloc": {},
    "coinbase": "0x0000000000000000000000000000000000000000",
    "difficulty": "0x20000",
    "extraData": "",
    "gasLimit": "0x2fefd8",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "nonce": "0x0000000000000042",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp": "0x00"
}
^return

Step 4. 
