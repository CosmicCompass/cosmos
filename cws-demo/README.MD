## Demo Setup

### Install Starport
```
go get -u github.com/CosmicCompass/starport
```
- Install
```bash=
cd $GOPATH/src/github.com/CosmicCompass/starport
git checkout cws-demo
make install
```

### Setup of two chains

- #### FreeFlix-Media-Hub

- Setup chain
```bash=
starport app github.com/<Your UserName>/ff  -a freeflix -d ffmt
```
- Adding nfts Modules
```bash=
cd github.com/<Your UserName>/ff
starport add nfts
```

- Install 
```bash=
make install
```

- Creating Account
> Note: store the seed to use in relayer
```bash=
ffcli keys add relayer 
```

- Init Chain & Add Config
```bash=
bash entry-point.sh freeflix-media-hub $(ffcli keys show relayer -a)
```

- Start the FreeFlix App
```bash=
ffd start --pruning nothing
```
- #### CoCo-Post-Chain
```bash=
starport app github.com/<Your UserName>/coco -a coco -d coco
```

- Adding nfts Modules
```bash=
starport add nfts
```
- Install  App
```bash=
make install
```
- Creating Account
 > Note: store the seed to use in relayer
```
cococli keys add relayer 
```

- Init Chains & Add Config
```bash=
bash entry-point.sh coco-post-chain $(cococli keys show relayer -a)
```
- Start CoCo chain
```bash=
cocod start  --rpc.laddr tcp://0.0.0.0:26557 --p2p.laddr tcp://0.0.0.0:26558 --address tcp://0.0.0.0:26556 --pruning nothing
```

- ### Relayer Config
- Install relayer
```bash=
go get -u github.com/iqlusioninc/relayer

git check goz-phase-3
make install
```

- Config init
```bash=
rly config init
```
- Adding chains config
> Note: [relayer config](https://github.com/CosmicCompass/cosmos/tree/develop/cws-demo/config)
```bash=
rly chains list 

rly chains add -f coco.json 

rly chains add -f freeflix.json

```
- Restore keys
```bash=
rly keys restore freeflix-media-hub testkey "<seed>"

rly keys restore coco-post-chain testkey "<seed>"
```
- Init light clients
```bash=
rly lite init coco-post-chain -f
rly lite init freeflix-media-hub -f
```
- Generate paths b/w two chains
```bash=
rly paths list
rly paths gen -f freeflix-media-hub xnfts coco-post-chain xnfts fftococo
```
- Create connection b/w two chains 
```bash=
rly tx link fftococo
```
- Show path
```bash=
rly paths show fftococo
```
- Start relayer
```bash=
rly start fftococo
```

### Txs 

#### FreeFlix Txs
- Query Account Balance
```bash=
ffcli q bank balances $(ffcli keys show relayer -a) -o json --trust-node | jq
```

- Query PrimaryNFTs Associated to Address
```bash=
ffcli q nfts nfts $(ffcli keys show relayer -a) -o json  --trust-node | jq
```

- Tx: Create PrimaryNFT in FreeFlix Chain
```bash=
ffcli tx nfts mint-nft --from relayer --asset-id asset1 --handle sai -b block --chain-id freeflix-media-hub  --licence true --licence-fee 100coco --revenue-share 0.2
```
#### CoCo Txs
- Query Account Balance
```bash=
cococli q bank balances $(cococli keys show relayer -a) -o json --trust-node --node tcp://0.0.0.0:26557 | jq
```
- Query SecondaryNFTs Associated to Address
```bash=
cococli q nfts nfts $(cococli keys show relayer -a) -o json  --trust-node --node tcp://0.0.0.0:26557 | jq
```
- Tx: Pay LicensingFee for Asset
```bash=
cococli tx xnfts pay-licensing-fee xnfts fqqfvsslkj 1000 100coco $(ffcli keys show relayer -a)  ffmttweetnft0 --from relayer  -b block --chain-id coco-post-chain --node tcp://0.0.0.0:26557
```
