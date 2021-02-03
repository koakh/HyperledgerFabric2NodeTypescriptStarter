# NOTES DevMode / Debug Hyperledger Fabric 2.2 Chaincode with JavaScript and Typescript

## TODO

- [-] todo join devMode and project in same dir and push to github
- [-] create a npm scripts or screen session to launch

  1. teardown
  2. upDevMode (orderer and peer devMode)
  3. create/join channel
  4. chaicode liveCycle
  5. launch chainCode with F5 (not needed)
  6. init chaincode

## TOC

- [NOTES DevMode / Debug Hyperledger Fabric 2.2 Chaincode with JavaScript and Typescript](#notes-devmode--debug-hyperledger-fabric-22-chaincode-with-javascript-and-typescript)
  - [TODO](#todo)
  - [TOC](#toc)
  - [Links](#links)
    - [Oficial](#oficial)
    - [Fix Error chaincode has not been initialized for this version, must call as init first](#fix-error-chaincode-has-not-been-initialized-for-this-version-must-call-as-init-first)
    - [Fix Error: failed to initialize operations subsystem: listen tcp 127.0.0.1:9443: bind: address already in use](#fix-error-failed-to-initialize-operations-subsystem-listen-tcp-1270019443-bind-address-already-in-use)
  - [Lift Dev-Mode](#lift-dev-mode)
    - [Pre Reqs](#pre-reqs)
    - [All Terminals](#all-terminals)
      - [TearDown](#teardown)
      - [Terminal 1 : Start the orderer](#terminal-1--start-the-orderer)
      - [Terminal 2 : Start the peer in dev mode](#terminal-2--start-the-peer-in-dev-mode)
      - [Terminal 3 : Create channels, build chaincode](#terminal-3--create-channels-build-chaincode)
      - [Terminal 4 : Approve and commit the chaincode definition](#terminal-4--approve-and-commit-the-chaincode-definition)
      - [Terminal 4 : Next steps : invoke and query the chaincode](#terminal-4--next-steps--invoke-and-query-the-chaincode)
    - [Try change chaincode and build and check changes](#try-change-chaincode-and-build-and-check-changes)
  - [Deplopy myAsset chaincode on DevMode network](#deplopy-myasset-chaincode-on-devmode-network)
    - [Invoke and query the chaincode as needed to verify your smart contract logic](#invoke-and-query-the-chaincode-as-needed-to-verify-your-smart-contract-logic)
  - [Debug node Chaincode on Vscode and dev-mode peer](#debug-node-chaincode-on-vscode-and-dev-mode-peer)
    - [Now try run node chainCode in peer dev-mode like ou do with go chaincode](#now-try-run-node-chaincode-in-peer-dev-mode-like-ou-do-with-go-chaincode)
      - [Terminal 1 : Start the orderer](#terminal-1--start-the-orderer-1)
      - [Terminal 2 : Start the peer in dev mode](#terminal-2--start-the-peer-in-dev-mode-1)
    - [Terminal 3 : Create channels, build chaincode](#terminal-3--create-channels-build-chaincode-1)
      - [Terminal 3 : Start debug node chaincode](#terminal-3--start-debug-node-chaincode)
      - [ChainCode LiveCycle](#chaincode-livecycle)
      - [Inspect ChainCode](#inspect-chaincode)
      - [Invoke/ Query Chaincode](#invoke-query-chaincode)
      - [Start Vscode debug with JavaScript](#start-vscode-debug-with-javascript)
      - [Start Vscode debug with JavaScript](#start-vscode-debug-with-javascript-1)
  - [Bellow are Other non working try and fail notes](#bellow-are-other-non-working-try-and-fail-notes)
    - [Try use '1 Org Local Fabric' with VsCode Debugger (WIP, not working)](#try-use-1-org-local-fabric-with-vscode-debugger-wip-not-working)
    - [Another Try](#another-try)
    - [MiniFabric Debug](#minifabric-debug)

## Links

### Oficial

- [Running chaincode in development mode &mdash; hyperledger-fabricdocs master documentation](https://hyperledger-fabric.readthedocs.io/en/release-2.3/peer-chaincode-devmode.html)

- [peer chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html)
- [peer lifecycle chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html)

- [Deploying a smart contract to a channel &mdash; hyperledger-fabricdocs master documentation](https://hyperledger-fabric.readthedocs.io/en/latest/deploy_chaincode.html#)

### Fix Error chaincode has not been initialized for this version, must call as init first

- [Error in committed chaincode invoke/query with hyperledger fabric 2.0](https://stackoverflow.com/questions/60391417/error-in-committed-chaincode-invoke-query-with-hyperledger-fabric-2-0)

I would think that you have used the peer chaincode lifecycle `approveformyorg` command with the flag `--init-required`. This means that the first invoke of the new chaincode must use the `--isInit` flag e.g. `peer chaincode invoke -C tradechannel -n trade --isInit` .... After this first "special" invoke, other invokes and querys will be fine.

### Fix Error: failed to initialize operations subsystem: listen tcp 127.0.0.1:9443: bind: address already in use

occur when we start peer

- [Port error when setting up Dev mode of Hyperledger Fabric](https://stackoverflow.com/questions/65387254/port-error-when-setting-up-dev-mode-of-hyperledger-fabric)

```shell
# fix
$ sudo nano sampleconfig/core.yaml
```

```yaml
operations:
    # host and port for the operations server
    # listenAddress: 127.0.0.1:9443
    listenAddress: 127.0.0.1:10443
```

## Lift Dev-Mode

### Pre Reqs

```shell
$ sudo zypper in go1.15
go version
go version go1.15.6 linux/amd64
```

```shell
# clone repo
$ cd ~/Development/HyperLedger/
$ git clone https://github.com/hyperledger/fabric.git fabricDevMode
# set up environment
$ make orderer peer configtxgen

```

### All Terminals

```shell
# open four terminals and launch in both
$ cd ~/Development/HyperLedger/fabricDevMode && export PATH=$(pwd)/build/bin:$PATH && export FABRIC_CFG_PATH=$(pwd)/sampleconfig
```

#### TearDown

always tear down, stop orderer and peer and

```shell
$ rm /var/hyperledger/production -r
```

#### Terminal 1 : Start the orderer

```shell
# generate the genesis block for the ordering service
$ configtxgen -profile SampleDevModeSolo -channelID syschannel -outputBlock genesisblock -configPath $FABRIC_CFG_PATH -outputBlock $(pwd)/sampleconfig/genesisblock
...
2021-01-23 17:55:56.479 WET [common.tools.configtxgen] doOutputBlock -> INFO 006 Writing genesis block

# start the orderer
$ ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer
...
2021-01-23 17:38:38.189 WET [orderer.common.server] Main -> INFO 00b Beginning to serve requests
```

#### Terminal 2 : Start the peer in dev mode

> don't forget to change `listenAddress: 127.0.0.1:9443` to `listenAddress: 127.0.0.1:10443`

Open another terminal window and set the required environment variables to override the peer configuration **and start the peer node**. Starting the peer with the `--peer-chaincodedev=true` flag puts the peer into **DevMode**.

```shell
# start the peer in dev-mode
$ FABRIC_LOGGING_SPEC=chaincode=debug CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 peer node start --peer-chaincodedev=true
...
2021-01-23 17:39:01.069 WET [chaincode] Launch -> DEBU 049 launch complete
```

> reminder: When running in **DevMode**, TLS cannot be enabled.

#### Terminal 3 : Create channels, build chaincode

```shell
# create channel and join peer
$ configtxgen -channelID ch1 -outputCreateChannelTx ch1.tx -profile SampleSingleMSPChannel -configPath $FABRIC_CFG_PATH
...
2021-01-23 18:09:56.537 WET [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
2021-01-23 18:09:56.538 WET [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx

# create channel from ch1.tx
$ peer channel create -o 127.0.0.1:7050 -c ch1 -f ch1.tx

...
2021-01-23 18:10:17.764 WET [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-01-23 18:10:17.777 WET [cli.common] readBlock -> INFO 002 Received block: 0

# now join the peer to the channel by running the following command:
$ peer channel join -b ch1.block
...
2021-01-23 18:11:21.568 WET [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-01-23 18:11:21.582 WET [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel

# the peer has now joined channels ch1

# check chaincode location
$ tree ./integration/chaincode/simple/cmd
./integration/chaincode/simple/cmd
└── main.go
# build the chaincode
$ go build -o simpleChaincode ./integration/chaincode/simple/cmd

# start the chaincode
# when DevMode is enabled on the peer, the CORE_CHAINCODE_ID_NAME environment variable must be set to <CHAINCODE_NAME>:<CHAINCODE_VERSION> otherwise, the peer is unable to find the chaincode. For this example, we set it to mycc:1.0. Run the following command to start the chaincode and connect it to the peer:
$ CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052

# because we set debug logging on the peer when we started it, you can confirm that the chaincode registration is successful. In your peer logs, you should see results similar to:

# we see this in terminal #2 peer
2021-01-23 18:12:25.888 WET [chaincode] sendReady -> DEBU 052 sending READY for chaincode mycc:1.0
2021-01-23 18:12:25.888 WET [chaincode] sendReady -> DEBU 053 Changed to state ready for chaincode mycc:1.0

# terminal #3 keeps running `chaincode` until we quit with ctrl-c
```

#### Terminal 4 : Approve and commit the chaincode definition

> now you need to **run the following Fabric chaincode lifecycle commands** to **approve** and **commit** the chaincode definition to the channel:

```shell
# approveformyorg
$ peer lifecycle chaincode approveformyorg  -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --package-id mycc:1.0
...
2021-01-23 18:14:16.218 WET [chaincodeCmd] ClientWait -> INFO 001 txid [3502b0fa6cd67c9d5a2d0f4f1725801503f42253d482a8eaf9743b15f6adfa0f] committed with status (VALID) at 0.0.0.0:7051

# checkcommitreadiness
$ peer lifecycle chaincode checkcommitreadiness -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')"
...
Chaincode definition for chaincode 'mycc', version '1.0', sequence '1' on channel 'ch1' approval status by org:
SampleOrg: true

# commit
$ peer lifecycle chaincode commit -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --peerAddresses 127.0.0.1:7051
...
2021-01-23 18:14:47.860 WET [chaincodeCmd] ClientWait -> INFO 001 txid [d33531196c09363bb418b09129b9380ce70107a95b3c897ed6edf199f235fcb3] committed with status (VALID) at 127.0.0.1:7051
```

#### Terminal 4 : Next steps : invoke and query the chaincode

```shell
# init: first invoke requires --isInit
$ CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["init","a","100","b","200"]}' --isInit
...
2021-01-23 18:16:23.441 WET [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200

# invoke
$ CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["invoke","a","b","10"]}'
...
2021-01-23 18:16:39.926 WET [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200

# query
$ CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["query","a"]}'
...
2021-01-23 18:16:51.037 WET [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200 payload:"90"
# returns payload:"90"
```

The benefit of running the peer in **DevMode** is that **you can now iteratively make updates to your smart contract, save your changes, build the chaincode, and then start it again using the steps above**. You do not need to run the peer lifecycle commands to update the chaincode every time you make a change.

### Try change chaincode and build and check changes

in terminal #3 the one that is running chaincode, quit with ctrl+c

main files 
- `~/Development/HyperLedger/fabricDevMode/integration/chaincode/simple/cmd/main.go`
- `~/Development/HyperLedger/fabricDevMode/integration/chaincode/simple/chaincode.go`

edit change `chaincode.go` result to test deployment without chaincode liveCycle

```go
// query callback representing the query of a chaincode
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
  ...
	// fmt.Printf("Query Response:%s\n", jsonResp)
	fmt.Printf("Changed Query Response:%s\n", jsonResp)
	return shim.Success(Avalbytes)
}
```

```shell
# rebuild chaincode
$ go build -o simpleChaincode ./integration/chaincode/simple/cmd
# run chaincode in dev-mode
$ CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052
# query and check log i terminal #3 chaicode logs
$ CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["query","a"]}'
# terminal 3 output
Changed Query Response:{"Name":"a","Amount":"90"}
```

done we can see the changed `Changed Query Response` string

revert changes in chaicode

## Deplopy myAsset chaincode on DevMode network

in terminal #3 stop go chaincode if is running

```shell
# package myasset and bring it
$ cp ~/.fabric-vscode/v2/packages/myasset@0.0.1.tar.gz .

# install chaincode
$ peer lifecycle chaincode install myasset@0.0.1.tar.gz
2021-01-23 18:20:23.445 WET [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nNmyasset_0.0.1:300cf381df4cb772d95381b7e5b27db6a85b6b5418afa3a8e510fb70c1f338e7\022\rmyasset_0.0.1" >
2021-01-23 18:20:23.445 WET [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: myasset_0.0.1:300cf381df4cb772d95381b7e5b27db6a85b6b5418afa3a8e510fb70c1f338e7

# queryinstalled
$ peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: myasset_0.0.1:300cf381df4cb772d95381b7e5b27db6a85b6b5418afa3a8e510fb70c1f338e7, Label: myasset_0.0.1

we are going to use the **package ID when we approve the chaincode**, so let’s go ahead and save it as an environment variable. Paste the package ID returned by peer lifecycle chaincode queryinstalled into the command below

# the chaincode id never changes if myasset@0.0.1.tar.gz is the same
$ export CC_PACKAGE_ID=myasset_0.0.1:300cf381df4cb772d95381b7e5b27db6a85b6b5418afa3a8e510fb70c1f338e7

# approve and commit the chaincode definition
# now you need to run the following Fabric chaincode lifecycle commands to approve and commit the chaincode definition to the channel:

$ CHAINCODE=myasset
# must match package filename and version
$ VERSION=0.0.1
# override above
# export CC_PACKAGE_ID=${CHAINCODE}:${VERSION}

# approveformyorg with --init-required
$ peer lifecycle chaincode approveformyorg  -o 127.0.0.1:7050 --channelID ch1 --name ${CHAINCODE} --version ${VERSION} --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --package-id ${CC_PACKAGE_ID}
...
2021-01-23 18:23:41.122 WET [chaincodeCmd] ClientWait -> INFO 001 txid [bd1706ca4850d18784a93cb4fe9a7b9e8a9f05aff79f412fb432056e47268cbb] committed with status (VALID) at 0.0.0.0:7051

# checkcommitreadiness
$ peer lifecycle chaincode checkcommitreadiness -o 127.0.0.1:7050 --channelID ch1 --name ${CHAINCODE} --version ${VERSION} --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')"
Chaincode definition for chaincode 'myasset', version '0.0.1', sequence '1' on channel 'ch1' approval status by org:
SampleOrg: true

# commit and initialize
$ peer lifecycle chaincode commit -o 127.0.0.1:7050 --channelID ch1 --name ${CHAINCODE} --version ${VERSION} --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --peerAddresses 127.0.0.1:7051
...
2021-01-23 18:24:30.101 WET [chaincodeCmd] ClientWait -> INFO 001 txid [b651be623312bf89629e525877692977eb2106c80e577458c724bb4309d60c11] committed with status (VALID) at 127.0.0.1:7051
```

### Invoke and query the chaincode as needed to verify your smart contract logic

```shell
$ CORE_PEER_ADDRESS=127.0.0.1:7051
$ CHAINCODE=myasset
$ BASE_INVOKE_CMD="peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n ${CHAINCODE}"
# require to pass --isInit to initialize chaincode
$ ${BASE_INVOKE_CMD} -c '{"Args":["createMyAsset", "001", "Leon"]}' --isInit
$ ${BASE_INVOKE_CMD} -c '{"Args":["myAssetExists", "001"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["readMyAsset", "001"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["updateMyAsset", "001","BMW"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["readMyAsset", "001"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["deleteMyAsset", "001"]}'
```

## Debug node Chaincode on Vscode and dev-mode peer 

### Now try run node chainCode in peer dev-mode like ou do with go chaincode

lift devMode network following bellow steps

#### [Terminal 1 : Start the orderer](#terminal-1--start-the-orderer)

```shell
# open two terminals and launch in both
$ cd ~/Development/HyperLedger/fabricDevMode && export PATH=$(pwd)/build/bin:$PATH && export FABRIC_CFG_PATH=$(pwd)/sampleconfig

# tear down
$ rm /var/hyperledger/production -r

# generate the genesis block for the ordering service
$ configtxgen -profile SampleDevModeSolo -channelID syschannel -outputBlock genesisblock -configPath $FABRIC_CFG_PATH -outputBlock $(pwd)/sampleconfig/genesisblock
# start the orderer
$ ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer
```

#### [Terminal 2 : Start the peer in dev mode](#terminal-2--start-the-peer-in-dev-mode)

> TLDR version

```shell
# start the peer in dev-mode
$ FABRIC_LOGGING_SPEC=chaincode=debug CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 peer node start --peer-chaincodedev=true
```

### [Terminal 3 : Create channels, build chaincode](#terminal-3--create-channels-build-chaincode)

> TLDR version

```shell
# create/join channels
$ configtxgen -channelID ch1 -outputCreateChannelTx ch1.tx -profile
SampleSingleMSPChannel -configPath $FABRIC_CFG_PATH
$ peer channel create -o 127.0.0.1:7050 -c ch1 -f ch1.tx
$ peer channel join -b ch1.block
```

#### [Terminal 3 : Start debug node chaincode](#)

> TLDR version

```shell
$ cd ~/Development/HyperLedger/ibmBlockChainExtension/tutorials/22x/01_introduction
$ npm run build
# go run command: use go chaincode as inspiration
$ CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052
2021-02-02 23:29:52.683 WET [chaincode] sendReady -> DEBU 19e sending READY for chaincode mycc:1.0
2021-02-02 23:29:52.683 WET [chaincode] sendReady -> DEBU 19f Changed to state ready for chaincode mycc:1.0
# in go after we launch chaincode we see in peer / terminal 2 the lines 
# DEBU 19f Changed to state ready for chaincode mycc:1.0

# node run command : try 1 seems work/ to soon
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=myasset:0.0.1 nodemon --inspect=9928 dist/index.js -peer.address 127.0.0.1:7052
[nodemon] clean exit - waiting for changes before restart
```

> TIP: in fabcar javascript we get this from `package.json`

```shell
fabric-chaincode-node start
integration/chaincode/fabcar/javascript/node_modules/.bin/fabric-chaincode-node
```

> the debug that works with javascript

```shell
# start in myasset vscode ibmBlockChainExtension sample project
$ cd ~/Development/HyperLedger/ibmBlockChainExtension/tutorials/22x/01_introduction

# first command that work and `established communication with peer node`
$ CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=myasset:0.0.1 node_modules/.bin/fabric-chaincode-node start /home/mario/Development/HyperLedger/ibmBlockChainExtension/tutorials/22x/01_introduction/dist/my-asset-contract.js --peer.address 127.0.0.1:7052
...
2021-02-03T00:18:38.878Z info [c-api:lib/handler.js]                              Successfully registered with peer node. State transferred to "established"  
2021-02-03T00:18:38.879Z info [c-api:lib/handler.js]                              Successfully established communication with peer node. State transferred to "ready" 
# now appears in terminal 2 
# Changed to state ready for chaincode myasset:0.0.1

# the trick is using `fabric-chaincode-node start`

# finnaly after getting code from dev-mode tutorial it start to work, and it detects chaincode
$ CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n myasset -c '{"Args":["createMyAsset", "001", "Leon"]}' --isInit
2021-02-03 00:27:55.876 WET [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
# now we can jump to peer lifecycle chaincode in step `approveformyorg`
```

#### ChainCode LiveCycle

```shell
# approveformyorg
$ peer lifecycle chaincode approveformyorg  -o 127.0.0.1:7050 --channelID ch1 --name myasset --version 0.0.1 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --package-id myasset:0.0.1
# works
2021-02-02 22:39:43.194 WET [chaincodeCmd] ClientWait -> INFO 001 txid [3bcdb73500cefee4ea2784330902d1ad771d28b8b807e57c45c32f74436ab9ca] committed with status (VALID) at 0.0.0.0:7051

# checkcommitreadiness
$ peer lifecycle chaincode checkcommitreadiness -o 127.0.0.1:7050 --channelID ch1 --name myasset --version 0.0.1 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')"
# works
Chaincode definition for chaincode 'myasset', version '0.0.1', sequence '1' on channel 'ch1' approval status by org:
SampleOrg: true

# commit
$ peer lifecycle chaincode commit -o 127.0.0.1:7050 --channelID ch1 --name myasset --version 0.0.1 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --peerAddresses 127.0.0.1:7051
# works
2021-02-02 22:41:05.490 WET [chaincodeCmd] ClientWait -> INFO 001 txid [90d9314ce2f8263da594e6c5164473d25ab0057d6907f6541474a101c13954c6] committed with status (VALID) at 127.0.0.1:7051
```

#### Inspect ChainCode

```shell
# queryinstalled
$ peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:

# querycommitted
$ peer lifecycle chaincode querycommitted --channelID ch1
Committed chaincode definitions on channel 'ch1':
Name: myasset, Version: 0.0.1, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc

# queryapproved
$ peer lifecycle chaincode queryapproved --channelID ch1 -n myasset
Approved chaincode definition for chaincode 'myasset' on channel 'ch1':
sequence: 1, version: 0.0.1, init-required: true, package-id: myasset:0.0.1, endorsement plugin: escc, validation plugin: vscc
```

#### Invoke/ Query Chaincode

```shell
$ CORE_PEER_ADDRESS=127.0.0.1:7051
# dont add :0.0.1 version
$ CHAINCODE=myasset
$ BASE_INVOKE_CMD="peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n ${CHAINCODE}"

# init chaincode, require to pass --isInit to initialize chaincode
$ ${BASE_INVOKE_CMD} -c '{"Args":["createMyAsset", "001", "Leon"]}' --isInit
# error catched after we found that we don't need the version
Error: endorsement failure during invoke. response: status:500 message:"make sure the chaincode myasset:0.0.1 has been successfully defined on channel ch1 and try again: chaincode myasset:0.0.1 not found"
# above error is because we add version, ex myasset:0.0.1, chaincode dont have :version

$ ${BASE_INVOKE_CMD} -c '{"Args":["myAssetExists", "001"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["readMyAsset", "001"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["updateMyAsset", "001","BMW"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["readMyAsset", "001"]}'
$ ${BASE_INVOKE_CMD} -c '{"Args":["deleteMyAsset", "001"]}'
# start again create others
$ ${BASE_INVOKE_CMD} -c '{"Args":["createMyAsset", "002", "Golf"]}'
```

#### Start Vscode debug with JavaScript

after we get the chaincode registered in peer with devMode with `Successfully established communication with peer node. State transferred to "ready"`, we can start to debug chaincode in `javascript`

launch 

```shell
# first launch our magic comand to run chaincode on devMode peer
$ CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=myasset:0.0.1 node_modules/.bin/fabric-chaincode-node start /home/mario/Development/HyperLedger/ibmBlockChainExtension/tutorials/22x/01_introduction/dist/my-asset-contract.js --peer.address 127.0.0.1:7052
```

now create a `launch.json` debug config to `Attach by Process ID` (`fabric-chaincode-node start`)

```json
{
  "name": "Attach by Process ID",
  "processId": "${command:PickProcess}",
  "request": "attach",
  "skipFiles": [
    "<node_internals>/**"
  ],
  "type": "pwa-node"
}
```

now press F5 and choose `fabric-chaincode-node start` processId

```shell
# run watch to build when we work
$ npm run build:watch

# we can see that debugger attached to peer chaincode
For help, see: https://nodejs.org/en/docs/inspector
Debugger attached.
```

`ctrl-c` to quit running chaindoe, add a breakpoint `debugger;` in

```javascript
async createMyAsset(ctx, myAssetId, value) {
  debugger;
```

> NOTE run bellow after start orderer, start peer dev-mode, create/join channel, chaicode liveCycle, init chaincode, and test invokes

launch debuger with F5, choose processId, and invoke `createMyAsset` 

done, we have debug is working in javascript at last

#### Start Vscode debug with JavaScript

to use typescript we must have a magic configuration, to run the magic command `fabric-chaincode-node start`, and the soureMaps configured

typescript working to with this magic config

`launch.json`

```json
{
  "name": "Debug Chaincode",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/node_modules/.bin/fabric-chaincode-node",
  "args": [
    "start",
    "dist/my-asset-contract.js",
    "--peer.address",
    "127.0.0.1:7052",
  ],
  "env": {
    "CORE_CHAINCODE_LOGLEVEL": "debug",
    "CORE_PEER_TLS_ENABLED": "false",
    "CORE_CHAINCODE_ID_NAME": "myasset:0.0.1"
  },
  "cwd": "${workspaceFolder}",
  "protocol": "auto",
  "disableOptimisticBPs": true,
  "sourceMaps": true,
  "smartStep": true,
  "outFiles": [
    "${workspaceFolder}/dist/**/*.js"
  ],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen",
},
```

add a `debugger;` to `src/my-asset-contract.ts`

```typescript
@Transaction()
public async createMyAsset(ctx: Context, myAssetId: string, value: string): Promise<void> {
  debugger;
  const exists: boolean = await this.myAssetExists(ctx, myAssetId);
  if (exists) {
    throw new Error(`The my asset ${myAssetId} already exists`);
  }
```

```shell
# run watch to build when we work
$ npm run build:watch
```

processed with folowing actions

1. start orderer
2. start peer dev-mode
3. create/join channel
4. chaicode liveCycle
5. launch chainCode with F5, acts as install
6. init chaincode (on init we land in typescript breakpoint)
7. and test invokes

> the debug configuration launch `fabric-chaincode-node` and **attach to process**, and works with typescript sourceMaps, pretty awesome


- [Hyperledger Fabric  How to run node js chaincode in dev mode](https://www.edureka.co/community/29595/hyperledger-fabric-how-to-run-node-js-chaincode-in-dev-mode)
> I used the following command and it worked like a charm: `$(npm bin)/fabric-chaincode-node start --peer.address localhost:7052 --chaincode-id-name "mycontract:v0"`

- [Attach to process: cannot put process &#39;18&#39; in debug mode. · Issue #1834 · microsoft/vscode-remote-release](https://github.com/microsoft/vscode-remote-release/issues/1834)

- [Hyperledger Fabric: ERROR [lib/handler.js] Chat stream with peer - on error: &quot;Error: 14 UNAVAILABLE: EOF\n at createStatusError](https://stackoverflow.com/questions/53991656/hyperledger-fabric-error-lib-handler-js-chat-stream-with-peer-on-error-er)

- [7 Performing Transactions on the Network · Programming Hyperledger Fabric: Creating Permissioned Blockchains MEAP V03](https://livebook.manning.com/book/programming-hyperledger-fabric/chapter-7/v-3/121)

## Bellow are Other non working try and fail notes

### Try use '1 Org Local Fabric' with VsCode Debugger (WIP, not working)

```shell
# base command
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052
inside container ibm stack `/opt/fabric/config/core.yaml`

# change start.sh script
$ cd ~/.fabric-vscode/v2/environments/1\ Org\ Local\ Fabric
# bring file to edit
$ docker cp ed6c0edc13e2:/opt/fabric/config/core.yaml /tmp
# uncomment
chaincodeListenAddress: 0.0.0.0:7052
# change to
mode: dev

# flag puts the peer into DevMode.
FABRIC_LOGGING_SPEC=chaincode=debug CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
# change docker run line to
$ nano /home/mario/.fabric-vscode/v2/environments/1\ Org\ Local\ Fabric/start.sh
# replace line
docker run --mount type=bind,source=/tmp/core.yaml,target=/opt/fabric/config/core.yaml -e CORE_PEER_TLS_ENABLED=false -e FABRIC_LOGGING_SPEC=chaincode=debug -e CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 -e MICROFAB_CONFIG --label fabric-environment-name="1 Org Local Fabric Microfab" -d -p 8080:8080 ibmcom/ibp-microfab:0.0.8

# try launch lines manually without -d 

now in output 1 Org Local Folder appears `fabric-chaincode-node start "--peer.address" "192.168.1.32:7052"`
```

stop here it won't work

### Another Try

```shell
# command that appears when we launch debugger
/home/mario/.nvh/bin/node ./node_modules/.bin/fabric-chaincode-node start --peer.address org1peer-chaincode.127-0-0-1.nip.io:8080
```

add to `launch.json`

```json
"program": "${workspaceFolder}/node_modules/.bin/fabric-chaincode-node",
"args": [
  "--peer-chaincodedev=true",
  "--chaincode-id-name=myasset:0.0.1"
],
```

now when run bellow command appears `--peer-chaincodedev=true`

```shell
/home/mario/.nvh/bin/node ./node_modules/.bin/fabric-chaincode-node --peer-chaincodedev=true start --peer.address org1peer-chaincode.127-0-0-1.nip.io:8080
```

with line and Package open project, sucefully invoke a `createMyAsset` with `Debug command list`

```shell
# launch stack by hand
$ docker run --mount type=bind,source=$(pwd)/core.yaml,target=/opt/fabric/config/core.yaml -e CORE_PEER_TLS_ENABLED=false -e FABRIC_LOGGING_SPEC=chaincode=debug -e CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 -e MICROFAB_CONFIG --label fabric-environment-name="1 Org Local Fabric Microfab" -p 8080:8080 ibmcom/ibp-microfab:0.0.8

# in other window try running debug chaincode
$ nvh i 12.8.1
$ npm run build
$ /home/mario/.nvh/bin/node ./node_modules/.bin/fabric-chaincode-node --peer-chaincodedev=true chaincode-id-name=myasset:0.0.1 start --peer.address org1peer-chaincode.127-0-0-1.nip.io:8080
# apears 
2021/01/23 19:39:21 http: proxy error: dial tcp 127.0.0.1:2005: connect: connection refused
2021/01/23 19:39:36 http: proxy error: dial tcp 127.0.0.1:2005: connect: connection refused
on stack logs
# inside stack
[ibp-user@a04cfbfec056 /]$ ls -la /opt/microfab/data/peer-org1/
# kill running pear and try launch manually with --peer-chaincodedev=true
$ peer node start --peer-chaincodedev=true start
2021-01-23 19:55:40.140 UTC [main] InitCmd -> ERRO 001 Cannot run peer because cannot init crypto, specified path "/opt/fabric/config/msp" does not exist or cannot be accessed: stat /opt/fabric/config/msp: no such file or directory
```

### MiniFabric Debug

- [Hyperledger fabric: use dev mode to debug chaincode (chaincode) - Programmer Sought](https://www.programmersought.com/article/68805506373/)

- [https://github.com/hyperledger-labs/minifabric/blob/main/playbooks/ops/netup/apply.yaml](https://github.com/hyperledger-labs/minifabric/blob/main/playbooks/ops/netup/apply.yaml)

```shell
hyperledger/fabric-peer:{{ fabric.release }} peer node start
# change to
hyperledger/fabric-peer:{{ fabric.release }} peer node start --peer-chaincodedev=true
``` 

```shell
$ docker build . -t minifabpatched
path script
$ code /usr/local/bin/minifab
```

```shell
#!/bin/bash
[ ! -d "$(pwd)/vars" ] && mkdir vars
ADDRS=$(ifconfig|grep 'inet '|grep -v '\.1 '|tr -s ' '|awk '{$1=$1};1'|cut -d ' ' -f 2|cut -d '/' -f 1|paste -sd "," -|sed s/addr://g)
if [ -f "$(pwd)/spec.yaml" ]; then
  echo "Using spec file: $(pwd)/spec.yaml"
  # docker run --rm --name minifab -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd)/vars:/home/vars \
  # -v $(pwd)/spec.yaml:/home/spec.yaml -e "ADDRS=$ADDRS" hyperledgerlabs/minifab:latest /home/main.sh "$@"
  docker run --rm --name minifab -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd)/vars:/home/vars \
  -v $(pwd)/spec.yaml:/home/spec.yaml -e "ADDRS=$ADDRS" minifabpatched:latest /home/main.sh "$@"
else
  echo "Using default spec file"
  # docker run --rm --name minifab -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd)/vars:/home/vars \
  # -e "ADDRS=$ADDRS" hyperledgerlabs/minifab:latest /home/main.sh "$@"
  docker run --rm --name minifab -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd)/vars:/home/vars \
  -e "ADDRS=$ADDRS" minifabpatched:latest /home/main.sh "$@"
fi
```
