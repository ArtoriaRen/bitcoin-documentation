# Bitcoin Documentation

## Table of  Contents
1. [Compile Bitcoin Core](#compile)
2. [Setup a Regtest Network on Local Machine](#setup-regtest-network)
3. [Bitcoin Core Source Code Walkthrough](#code-walkthrough)
4. [Add Miner to a Regtest Network](#miner)
## Compile Bitcoin Core <a name="compile"></a>
```bash
$ cd bitcoin
$ git checkout v0.16.0
$ ./autogen.sh
$ ./configure CPPFLAGS="-I${BDB_PREFIX}/include/ -O2" LDFLAGS="-L${BDB_PREFIX}/lib/" --with-gui
$ make
$ make install
```
If GUI is not needed, add `--without-gui` to the `configure` command.

see [Rich Apodaca's post](https://bitzuma.com/posts/compile-bitcoin-core-from-source-on-ubuntu/).

The final install is necessary if you want to start bitcoind regardless of directory. Otherwise, you can change path to bitcoin/src and start bitcoind therei (or give a full path when you are at another directory).

## Setup a Regtest Network on Local Machine <a name="setup-regtest-network" ></a>
### 1. Edit bitcoin.conf

Creater a folder `datadirConfig` under the bitcoin source code root folder to store blockchain info for nodes. Create two subfolders (`datadirConfig/datadir0`, `datadirConfi/gdatadir1`), and include a `bitcoin.conf` file in each of the two subfolders.
Add to `datadir0/bitcoin.conf`:

<!-- TODO: add a link to sample bitcoin.conf on my github. make sure the bitcoin.conf do not include sensetive info.
-->

    regtest=1
    port = 8330
    rpcport=8331

Add to `datadir1/bitcoin.conf`:

    regtest=1
    port=8332
    rpcport=8333
    connect=localhost:8330


### 2. Start a Two-node Network
Open two terminal windows, redirect to the folder `datadirConfig` and type in each window one following line:
```bash
$../src/bitcoind -datadir=./datadir0 -debug=1
$../src/bitcoind -datadir=./datadir1 -debug=1
```
`-debug=1` enables debug info to be logged. To check latest debug info:
```bash
$tail ./datadir0/regtest/debug.log
```

see [tdrusk's post](https://www.yours.org/content/connecting-multiple-bitcoin-core-nodes-in-regtest-5fdc9c47528b).

## Bitcoin Core Source Code <a name="code-walkthrough"></a>
### `block.h`
#### `class CBlock : public CBlockHeader`
Has the following public field to access transactions.
```cpp
public:
    std::vector<CTransactionRef> vtx;
```

### `chain.h`
####`class CBlockIndex`
Can access block header info, but not directly to transactions.

#### `class CService : public CNetAddr`
A combination of a network address (CNetAddr) and a (TCP) port.
```cpp
    protected: unsigned short port;
```

###  `net.h`
#### `class CNode`
Information about a peer.
```cpp
    public: const CAddress addr;
```

###  `netaddress.h`
#### `class CNetAddr`
IP address (IPv6, or IPv4 using mapped IPv6 range (::FFFF:0:0/96))
```cpp
    protected: unsigned char ip[16]; // in network byte order
```

###  `net_processing.cpp`
#### `ProcessMessage()`
Log network message (type, sender) received from peers.

### `primitives/transaction.h`
#### `class CTxOut`
Has two public field:
```cpp
    CAmount nValue;
    CScript scriptPubKey;
```
#### `class CTxIn`

###  `protocol.h`
#### `class CAddress : public CService`
A CService with information about it as peer.

### `rpc/blockchain.cpp`
Has functions to handle blockchain-related cli commands, e.g., getblockcount.

`getblock` function can return tx info of this block.
```cpp
	UniValue getblock(const JSONRPCRequest& request)
```


### `rpc/client.cpp`
Has a list of most cli commands.

### `rpc/mining.h`
Has a function `generateBlocks` and can check PoW target.

### `rpc/net.cpp`
Has functions to handle network-info-related cli commands.

### `rpc/rawtransaction.cpp`
Has functions to handle transaction-related cli commands.

### `rpc/register.h`
Register all cli commands.

### `rpc/util.h`
Has three functions to convert public keys (used by `wallet/rpcwallet.cpp`).
```cpp
    CPubKey HexToPubKey(const std::string& hex_in);
    CPubKey AddrToPubKey(CKeyStore* const keystore, const std::string& addr_in);
    CScript CreateMultisigRedeemscript(const int required, const std::vector<CPubKey>& pubkeys);
```
### `validation.h`
`mapBlockIndex` map block hashes to block index pointers, which has pretty much all block header info.
```cpp
	typedef std::unordered_map<uint256, CBlockIndex*, BlockHasher> BlockMap;
	extern BlockMap& mapBlockIndex;
```

###  `validation.cpp`
#### `UpdateTip()`
#### `ReadBlockFromDisk()`
Given a `CBlockIndex*`, Return corresponding block info (include all tx) by reading from disk and store the result in an `CBlock` object passed in by reference.

Check warning conditions and do some notifications on new chain tip set.

======================
# Hash 
`block.h::CBlockHeader::GetHash()` ---> `hash.h::SerializeHash()` ---> `hash.h::CHashWriter::GetHash()` ---> `hash.h::CHash256::Finalize()`.

## Add Miner to a Regtest Network <a name="miner"></a>
1. download the  Bitcoin CPU miner `https://github.com/pooler/cpuminer`
- Download a release version by clicking the `Downloads` link, which bring you to SOURCEFORGE website. Version 2.5.0 works on Mac OS High Sierra.
- Unzip the file, redirect to the root dir of cup-miner, compile it with commands:
```bash
./nomacro.pl
./configure CFLAGS="-O3"
make
``` 

2. run a bitcoind instance.
3. check mining info with 
```bash
$../src/bitcoin-cli -datadir=datadir0 getmininginfo
```
The output should be like
```json
{
  "blocks": 109,
  "currentblockweight": 4000,
  "currentblocktx": 0,
  "difficulty": 4.656542373906925e-10,
  "networkhashps": 8.41984596316665e-07,
  "pooledtx": 0,
  "chain": "regtest",
  "warnings": ""
}
```
At the beginning, mining difficulty and network hash powe are extremely low.
By default, mining difficulty is set to be not retargetable. To change it to be retargetable, open the file `src/chainparams.cpp` and set the value of the boolean variable `consensus.fPowNoRetargeting` at line 289 to `false`.

Check the balance of an account with 
```bash
../src/bitcoin-cli -datadir=datadir0 listaccounts
```
Check the address of the default with
```bash
../src/bitcoin-cli -datadir=node0 getaccountaddress ""
```
The output of my instance is 
```bash
2Mz36kRLMjVu2VkjdU8mnqAxxLoYuuGr6nF
```
The address will be used to receive payoff during mining.

4. run miner process
Change dir to the root of cpu-miner, then start a  miner process with only one thread:
```bash
./minerd --url=127.0.0.1:8331  --user=l27ren --pass=UofW2016  --debug --coinbase-addr=2Mz36kRLMjVu2VkjdU8mnqAxxLoYuuGr6nF -a sha256d -t 1
```
Note that `cpuminer` require the `bitcoind` to connect to at least one peer. Otherwise `cpuminer`  will report http error 500, and `bitcoind debug.log` will report `tor: Error connecting to Tor contril socket; tor: Not connected to Tor control port 127.0.0.1:9501, ...` 
Check the balance again, and you should see an increase. 
Check mining info again, and you should find the `networkhashps` has increased. 
Reference to [Niraj Blog](http://nirajkr.com/bitcoin/solo-cpu-mining-for-bitcoin-in-regtest-mode/)

