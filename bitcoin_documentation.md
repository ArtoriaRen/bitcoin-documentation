# Bitcoin Documentation

## Table of  Contents
1. [Compile Bitcoin Core](#compile)
2. [Setup a Regtest Network on Local Machine](#Setup a Regtest Network on Local Machine)

## Compile Bitcoin Core <a name="compile"></a>
```bash
$ cd bitcoin
$ git checkout v0.16.0
$ ./autogen.sh
$ ./configure CPPFLAGS="-I${BDB_PREFIX}/include/ -O2" LDFLAGS="-L${BDB_PREFIX}/lib/" --with-gui
$ make
```
see [Rich Apodaca's post](https://bitzuma.com/posts/compile-bitcoin-core-from-source-on-ubuntu/).

## Setup a Regtest Network on Local Machine <a name="Setup a Regtest Network on Local Machine" ></a>
### 1. Edit bitcoin.conf

Creater two folders (`datadir0`, `datadir1`), and include a `bitcoin.conf` file in each of the two folders.
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
Open two terminal windows, and type in each window one following line:
```bash
$bitcoind -datadir=./datadir0 -debug=1
$bitcoind -datadir=./datadir1 -debug=1
```
`-debug=1` enables debug info to be logged. To check latest debug info:
```bash
$tail ./datadir0/regtest/debug.log
```

see [tdrusk's post](https://www.yours.org/content/connecting-multiple-bitcoin-core-nodes-in-regtest-5fdc9c47528b).

## Bitcoin Core Source Code
### 1. `net_processing.cpp`
#### `ProcessMessage()`
Log network message (type, sender) received from peers.

### 2. `net.h`
#### `class CNode`
Information about a peer.
```cpp
    public: const CAddress addr;
```

### 3. `netaddress.h`
#### `class CNetAddr`
IP address (IPv6, or IPv4 using mapped IPv6 range (::FFFF:0:0/96))
```cpp
    protected: unsigned char ip[16]; // in network byte order
```

#### `class CService : public CNetAddr`
A combination of a network address (CNetAddr) and a (TCP) port.
```cpp
    protected: unsigned short port;
```
### 4. `protocol.h`
#### `class CAddress : public CService`
A CService with information about it as peer.

### 5. `validation.cpp`
#### `UpdateTip()`
Check warning conditions and do some notifications on new chain tip set.