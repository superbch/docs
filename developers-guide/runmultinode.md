# Multi-Node Testnet

This document shows how to start a testnet of smartBCH with multiple nodes. 

We suggest to use ubuntu 20.04.

In these nodes, just pick one to generate the genesis file. And all the other nodes do not generate the genesis file, instead, they use the generated genesis file as is.

### 1. On the nodes which do not generate the genesis file

#### Step 0: Prepare

Follow the instructions in [running a single-node testnet](./runsinglenode.md), and Just finish the steps 0, 1, 2, 3 and 4 described in it.



#### Step 1: generate the private keys to operate one validator

```bash
cd ~/smart_bch/smartbch
./smartbchd gen-test-keys -n 1 --show-address
2915b09298d0362df490fe3969b10aece94cd3460fb1ba29184cba31516a9b5f 0xF0c6969C2a554ddae639ba1Aa1d2fA11382CAb2B
```

The above command generates one private key. And the corresponding EOA address is `0xF0c6969C2a554ddae639ba1Aa1d2fA11382CAb2B`.



#### Step 2: generate genesis validator consensus key info

Now we generate the ed25519 private key for consensus engine:

```bash
./smartbchd generate-consensus-key-info
a07871d10858179c1bb3c1f0d7bab31103135b76f96b6bd1f06a5bc0d350f862
```

The output hex string is consensus pubkey which will be used in `generate-genesis-validator` command, and a file containing the consensus public and private key is generated under the current directory, named `priv_validator_key.json`.



#### Step 3: generate genesis validator info using pubkey generated above

```bash
./smartbchd generate-genesis-validator \
	--validator-address=0xF0c6969C2a554ddae639ba1Aa1d2fA11382CAb2B \
	--consensus-pubkey=a07871d10858179c1bb3c1f0d7bab31103135b76f96b6bd1f06a5bc0d350f862 \
	--voting-power=1 \
	--staking-coin=0 \
	--introduction="happyman"
7b2241646472657373223a5b3136362c3234382c3230392c39312c32342c3137382c3138352c36302c3139312c3132372c3137342c32352c33332c3133322c3230342c3231372c3232342c35392c3235302c3234345d2c225075626b6579223a5b3136302c3132302c3131332c3230392c382c38382c32332c3135362c32372c3137392c3139332c3234302c3231352c3138362c3137392c31372c332c31392c39312c3131382c3234392c3130372c3130372c3230392c3234302c3130362c39312c3139322c3231312c38302c3234382c39385d2c22526577617264546f223a5b3136362c3234382c3230392c39312c32342c3137382c3138352c36302c3139312c3132372c3137342c32352c33332c3133322c3230342c3231372c3232342c35392c3235302c3234345d2c22566f74696e67506f776572223a312c22496e74726f64756374696f6e223a2268617070796d616e222c225374616b6564436f696e73223a5b302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c302c305d2c2249735265746972696e67223a66616c73657d
```

The `validator-address` uses the one we get at step 1, and the `consensus-pubkey` is the one we get at step 2.

The output hex string contains the information of a validator. Send the this hex string to the node who is genesis-generator.



#### Step 4: wait for the data directory's tarball and unpack it

The genesis-generator will send you a tarball named dot.smartbchd.tgz, which contains the initial content for the data directory.

```bash
cd ~
tar zxvf dot.smartbchd.tgz
./smartbchd init mynode --chain-id 0x2711 \
  --init-balance=1000000000000000000000000000000 \
  --test-keys="a07871d10858179c1bb3c1f0d7bab31103135b76f96b6bd1f06a5bc0d350f862"
cp -rf dot.smartbchd/* .smartbchd/
```

The balance number and the test-keys are not importance, because these information will be overwritten by the genesis.json file in the `dot.smartbchd` directory.



#### Step 5: Copy private key file to data directory and start node

Copy priv_validator_key.json generated in Step 2, to the data directory:

```bash
cp ./priv_validator_key.json ~/.smartbchd/config/
```

Now start the node:

```
./build/smartbchd start
```



#### Step 6: activate validator

before you activate your genesis validator, try to get testnet BCH in your validator address, it need at least 

```
1000000000000000000000 bch
```

you can connect smartBCH team to get testnet bch, you can check your balance with below command:

replace `your_validator_address` with your genesis validator address

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["your_validator_address","latest"],"id":1}' -H "Content-Type: application/json" http://localhost:8545
```

After you have enough bch, you can follow below:

replace the `--validator-key` with your validator private key

repace the `--staking-coin` with `1000000000000000000000` or even more you have.

⚠️Keep your private key safe, and execute this command on a secure, offline machine

```
./build/smartbchd staking \
--validator-key=07427a59913df1ae8af709f60f536ddba122b0afa8908291471ca58c603a7447 \
--staking-coin=2000000000000000000000000 \
--nonce=0 \
--chain-id=0x2711
```

replace `--your_tx_data` with what hex string get above.

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendRawTransaction","params":["you_tx_data"],"id":1}' -H "Content-Type: application/json" http://localhost:8545
```

now, you had send a editValidator tx to staking contract，check the tx receipt

replace `your_tx_hash` with what the hex string you get above.

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["your_tx_has"],"id":1}' -H "Content-Type: application/json" http://localhost:8545
```

if the `status` displays `0x1`, congratulations, your genesis validator actived now.





### 2. On the node which generates the genesis file

#### Step 0: Prepare

Just finish the steps 0-9 as [running a single-node testnet](./runsinglenode.md).



#### Step 1: add genesis validator info to genesis.json using hex strings sent by other nodes

Using `./build/smartbchd add-genesis-validator` multiple times, one time with one hex string sent by one node. Finally we'll get all the validators' information in the genesis.json file.h

Then, make a copy of the `~/.smartbchd` directory, excluding the `node_key.json` file. 

```bash
cp -r ~/.smartbchd ~/dot.smartbchd
rm ~/dot.smartbchd/config/node_key.json
```



#### Step 2: start the node

Run the step 10 and 11 as  [running a single-node testnet](./runsinglenode.md).



#### Step 4: collect seeds information

Collect the p2p seeds information and modify the `~/dot.smartbchd/config/config.toml` file.

#### 

#### Step 5: send information to all the other nodes

Make a tarball:

```bash
cd ~
tar czvf dot.smartbchd.tgz dot.smartbchd
```

Then send this dot.smartbchd.tgz file to all the other nodes.



#### 