# Jalingpp/TestGeth192
A tutorial on "Building and Maintaining Private Blockchains with go-ethereum-1.9.2".

## 1. Download Go-ethereum-1.9.2
The installation package is `go-ethereum-1.9.2.zip`, or download other releases from `https://github.com/ethereum/go-ethereum/releases`.

Building geth requires both a Go (version 1.22 or later) and a C compiler. You can install them using your favourite package manager. Once the dependencies are installed, run `make geth`.

You can change the source codes and rebuild.

## 2. Build Private Blockchain
Step 1. Configure the execution path of geth: `cd go-ethereum-1.9.2` and then `cp ./build/bin/geth /usr/local/bin/`. Find the geth version with `geth --version`.

Step 2. New a folder for nodes beside go-ethereum-1.9.2: `mkdir ethnodes`.

Step 3. Generate `genesis.json` File in `ethnodes`. Copy the following content into the file.

```json:genesis.json
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
```

Step 4. Initialize a node of the private blockchain: `cd ethnodes` and then `geth --datadir ./ init ./genesis.json`.

Step 5. Start Node and go into the console: `cd ethnodes` and then `geth --networkid 200 --datadir "./" --nodiscover --rpcapi personal console`. Note that the networkid is the same as the chainId in `genesis.json`. `--rpcapi personal` allows usage of rpc methods of personal module.

Step 6. New accounts and test transfer in the console: view all accounts by `eth.accounts`, new account by `personal.newAccount("123456")` where `"123456"` is the passwords. After that we can see a file under `ethnodes/keystore/`.

Step 7. Set some balance to the new account by mining: view the address of coinbase by `eth.coinbase`, set coinbase by `miner.setEtherbase(eth.accounts[0])`, start mining by `miner.start()`, stop miner by `miner.stop()`, view the balance by `eth.getBalance(eth.accounts[0])`.

Step 8. New another account and make a transfer: new account by `personal.newAccount("123456")`, unlock the sender account by `personal.unlockAccount(eth.accounts[0],"123456")`, send trasaction by `eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:web3.toWei(3,"ether")})`, view balances by `eth.getBalance(eth.accounts[0])` and `eth.getBalance(eth.accounts[1])`.

Step 9. You can write multiple console commands into JavaScript and execute them all at once in the console.

```JavaScript:sendTx.js
var fromAddress = web3.eth.accounts[0];
var toAddress = web3.eth.accounts[1];
var v = web3.toWei(1, 'ether');
//unlock the sender account
web3.personal.unlockAccount(web3.eth.accounts[0],"123456")
//send transaction
web3.eth.sendTransaction({from:fromAddress,to:toAddress,value:v})
```

Then execute js file in console by `loadScript('/root/ethnodes/sendTx.js')`. View balances by `eth.getBalance(eth.accounts[0])` and `eth.getBalance(eth.accounts[1])`.

Step 10. Exit console by `ctrl+D`.

## 3. Explor Curl RPC Methods
Step 1. Start node to listen for RPC requests: `cd ethnodes` and then `geth --datadir ./ --rpc --rpcaddr "0.0.0.0" --rpcport "8545" --rpcapi "eth,net,web3,personal" --rpccorsdomain "*" --allow-insecure-unlock`.

Step 2. Create another terminal and try some curl commands.

view blockNumber: `curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' -H "Content-Type: application/json" localhost:8545`.

view chainId: `curl -X POST --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' -H "Content-T
ype: application/json" localhost:8545`.

view accounts: `curl -X POST --data '{"jsonrpc":"2.0","method":"eth_accounts","params":[],"id":1}' -H "Content-
Type: application/json" localhost:8545`.

unlock account: `curl -X POST --data '{"jsonrpc":"2.0","method":"personal_unlockAccount","params":["0xf1e20a2db759693c3b19eda5154eb13d0ff14472","123456"],"id":1}' -H "Content-Type: application/json" localhost:8545`.

send transaction: `curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xf1e20a2db759693c3b19eda5154eb13d0ff14472","to":"0x1fe1a8a7417cd979810c704be6c49affe0b8f7d5","value":"0x100000000000"}],"id":1}' -H "Content-Type: application/json" localhost:8545`.

......

Step 3. Listening to RPC while still being able to connect to the console: `geth attach http://localhost:8545`.

Step 4. Other methods (see in `geth-rpc-methods.txt`) found by printing services in func `ServeCodec` in `./go-ethereum-1.9.2/rpc/server.go`.

```go:server.go
func (s *Server) ServeCodec(codec ServerCodec, options CodecOption) {
	defer codec.Close()

	// Don't serve if server is stopped.
	if atomic.LoadInt32(&s.run) == 0 {
		return
	}

	// Add the codec to the set so it can be closed by Stop.
	s.codecs.Add(codec)
	defer s.codecs.Remove(codec)
	fmt.Println(s.services.services)
	c := initClient(codec, s.idgen, &s.services)
	<-codec.Closed()
	c.Close()
}
```
7:^