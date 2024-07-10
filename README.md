# Private Ethereum Network using Hyperledger BESU

In this tutorial I run an Ethereum Private network with PoA as a consensus algorithm by Hyperledger Besu Framework on Ubuntu 22.04

## Prerequisite:
* install Java 21 or later
```bash
sudo apt install openjdk-21-jdk
```
* install BesuCli
```bash
wget https://github.com/hyperledger/besu/releases/<latest Version>
tar xvf besu-24.6.0.tar.gz
vi ~/.bashrc
     export PATH=/..../besu-24.6.0/bin:$PATH
```

## create corresponding Directories for 3 node as follow
```bash
	mkdir -p Node-1/data
	mkdir -p Node-2/data
	mkdir -p Node-3/data
```

## Get the address for Node-1
```bash
cd Node-1
besu --data-path=data public-key export-address --to=data/node1Address
	
masoud@LAPTOP-U7TNPO3A:Node-1$ ll data
drwxrwxrwx 1 masoud masoud 4096 Jul 10 17:45 ./
drwxrwxrwx 1 masoud masoud 4096 Jul 10 14:20 ../
-rwxrwxrwx 1 masoud masoud   66 Jul 10 14:22 key*
-rwxrwxrwx 1 masoud masoud   42 Jul 10 14:22 node1Address*
```
__you need to use node1Address content in following steps__

## Create the genesis file
The genesis file defines the genesis block of the blockchain (that is, the start of the blockchain).\
The Clique genesis file includes the address of `Node-1` as the initial signer in the extraData field. All nodes in a network must use the same genesis file.\
Copy the following genesis definition to a file called `cliqueGenesis.json` and save it in the `Clique-Network` directory:
```bash
{
  "config": {
    "chainId": 1337,
    "berlinBlock": 0,
    "clique": {
      "blockperiodseconds": 15,
      "epochlength": 30000
    }
  },
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x1",

  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000<Node 1 Address>0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0xa00000",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "nonce": "0x0",
  "timestamp": "0x5c51a607",
  "alloc": {
    "fe3b557e8fb62b89f4916b721be55ceb828dbd73": {
      "privateKey": "8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63",
      "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
      "balance": "0xad78ebc5ac6200000"
    },
    "627306090abaB3A6e1400e9345bC60c78a8BEf57": {
      "privateKey": "c87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3",
      "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
      "balance": "90000000000000000000000"
    },
    "f17f52151EbEF6C7334FAD080c5704D77216b732": {
      "privateKey": "ae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f",
      "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
      "balance": "90000000000000000000000"
    }
  }
}
```
__note: all alloc part will be ignored.__

The properties specific to Clique are:
* blockperiodseconds - The block time, in seconds.
* epochlength - The number of blocks after which to reset all votes.
* createemptyblocks - Set to false to skip creating empty blocks.
* extraData - Extra data including the initial signers.

### Skip empty blocks:
By default, Clique creates empty blocks. For large private networks using Clique, skipping empty blocks can reduce the storage needed.

To skip creating empty blocks, set createemptyblocks to false in the genesis file:	

```bash
{
  "config": {
    "londonBlock": 0,
    "clique": {
      "blockperiodseconds": 10,
      "epochlength": 30000,
      "createemptyblocks": false
    }
  },
...
}
```
### Extra data:
The extraData property consists of:
* 0x prefix.
* 32 bytes of vanity data.
* A list of initial signer addresses (at least one initial signer is required). 20 bytes for each signer.
* 65 bytes for the proposer signature. In the genesis block there is no initial proposer, so the proposer signature is all zeros.

One initial signer:
![One initial signer](https://github.com/h-tabatabaei/Private_Ethereum_BESU/blob/main/images/CliqueOneIntialSigner.png)
Two initial signers
![Two initial signers](https://github.com/h-tabatabaei/Private_Ethereum_BESU/blob/main/images/CliqueTwoIntialSigners.png)

`in my Case: `
```bash
masoud@LAPTOP-U7TNPO3A:Node-1$ cat data/node1Address
0xfe93094bd7975ba4a788753c2e4bc7398a699719

masoud@LAPTOP-U7TNPO3A:Node-1$ cat ../cliqueGenesis.json | head -13
{
  "config": {
    "chainId": 1337,
    "berlinBlock": 0,
    "clique": {
      "blockperiodseconds": 15,
      "epochlength": 30000
    }
  },
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x1",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000fe93094bd7975ba4a788753c2e4bc7398a6997190000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
.
.

```
for more information about genesis file configuration visit: [Genesis File](https://besu.hyperledger.org/private-networks/how-to/configure/consensus/clique#genesis-file)

## Start the first node as the bootnode:
```bash
masoud@LAPTOP-U7TNPO3A:Node-1$ pwd
/..../priv-besu/Clique-Network/Node-1

besu --data-path=data --genesis-file=../cliqueGenesis.json --network-id 123 --rpc-http-enabled --rpc-http-api=ETH,NET,CLIQUE --host-allowlist="*" --rpc-http-cors-origins="all"
```
When the node starts, the enode URL displays. Copy the enode URL to specify Node-1 as the bootnode in the following steps.
```bash
	2024-07-10 17:31:42.917+03:30 | main | INFO  | DefaultP2PNetwork | Enode URL eno de://ec75f5c12c45a6e7f19ad38a9d8e0ff16ee22707bbad9803ada0fa61f5ed97fbe523aae646e7ec57fddcb7940742c6c0be34b157c87ef3a3ea04dc3b7da1f7e8@127.0.0.1:30303
```
you can find this address by sending curl request:
```bash
masoud@LAPTOP-U7TNPO3A:Node-1$ curl -X POST --data '{"jsonrpc":"2.0","method":"net_enode","params":[],"id":1}' http://127.0.0.1:8545
{"jsonrpc":"2.0","id":1,"result":"enode://ec75f5c12c45a6e7f19ad38a9d8e0ff16ee22707bbad9803ada0fa61f5ed97fbe523aae646e7ec57fddcb7940742c6c0be34b157c87ef3a3ea04dc3b7da1f7e8@127.0.0.1:30303"}
```
for more information about how to find enode address visit: [net_enode](https://besu.hyperledger.org/public-networks/reference/api#net_enode)
		
##  Start Node-2
```bash
masoud@LAPTOP-U7TNPO3A:Node-2$ pwd
/...../priv-besu/Clique-Network/Node-2
	
besu --data-path=data --genesis-file=../cliqueGenesis.json --bootnodes=<Node-1 Enode URL> --network-id 123 --p2p-port=30304 --rpc-http-enabled --rpc-http-api=ETH,NET,CLIQUE --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-port=8546
```
The command line specifies:
* A different port to Node-1 for P2P discovery using the --p2p-port option.
* A different port to Node-1 for HTTP JSON-RPC using the --rpc-http-port option.
* The enode URL of Node-1 using the --bootnodes option.
* The data directory for Node-2 using the --data-path option.
* Other options as for Node-1.

## Start Node-3
```bash
masoud@LAPTOP-U7TNPO3A:Node-3$ pwd
/...../priv-besu/Clique-Network/Node-3
	
besu --data-path=data --genesis-file=../cliqueGenesis.json --bootnodes=<Node-1 Enode URL> --network-id 123 --p2p-port=30305 --rpc-http-enabled --rpc-http-api=ETH,NET,CLIQUE --host-allowlist="*" --rpc-http-cors-origins="all" --rpc-http-port=8547
```
The command line specifies:

* A different port to Node-1 and Node-2 for P2P discovery using the --p2p-port option.
* A different port to Node-1 and Node-2 for HTTP JSON-RPC using the --rpc-http-port option.
* The data directory for Node-3 using the --data-path option.
* The bootnode as for Node-2.
* Other options as for Node-1.

## Confirm the private network is working
```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' localhost:8545

{"jsonrpc":"2.0","id":1,"result":"0x2"}
```
__The result confirms Node-1 has two peers (Node-2 and Node-3)__

## Stop the nodes
When finished using the private network, stop all nodes using `ctrl+c` in each terminal window.

	
For more information about installation steps visit: [Besu-clique](https://besu.hyperledger.org/private-networks/tutorials/clique)
		
 
