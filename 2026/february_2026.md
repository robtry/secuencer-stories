# 12 - 02 - 2026

I want to see if a wallet like metamask is able to handle my chain and custom tokens.

Yup its very easy to add a custom chain an also there custom tokens, I did test sending fUSDT from one wallet to another and it worked, also I can see the transactions in the explorer and of course the rpc.
You only need to have the rpc url, chain id, and of course the required addresses for the tokens and wallets.

Also since the wasm module works its a good time to test validation, until now I wasnt running a validator, hence I can not make withdraws.

I remember that in the config file when I deploy the chain I set the token to stake to be weth. So I going to need weth for my validator nodes, also I only whitelisted one validator,but it might be a good idea to whitelist more than one, just in case.

So step 1. send eth to the validartos wallets
step 2: wrap some of the eth to weth, its pretty simple also a send:

```sh
cast send 0xfff9976782d46cc05630d1f6ebab18b2324d6b14 "deposit()" --value 0.05ether \
    --rpc-url <RPC> --private-key <KEY>
```

3. check that the validator is whielisted in the chain:

```sh
 cast call 0x77E2752FBCfb3e131B4bc3a5A19C83438387eFEa \ # RollupProxy
    "isValidator(address)(bool)" \
    0xff09f8f611E17b2ba3Dc6ad6aa85ebCa80bdcb36 \ # validator address
    --rpc-url http://100.64.0.2:8545
```

If not whitelisted we can add it:

```sh
CALLDATA=$(cast calldata "setValidator(address[],bool[])" \
  "[<VALIDATOR_ADDRESS>]" "[true]") # this could be false

cast send 0x211Cf91bAE977914eAE40D1E4e755C44d76Dc9De \# UpgradeExecutor
  "executeCall(address,bytes)" \
  0x77E2752FBCfb3e131B4bc3a5A19C83438387eFEa \
  "$CALLDATA" \
  --rpc-url http://100.64.0.2:8545 \
  --private-key <OWNER_PRIVATE_KEY>
```

And also verify with the previos command. Owner has to call upgrade executor, I can not call rollup directly.

Without validato we can not move from l2 to l1. And for this there are 5 strategies, I am sure I already write them before, but I am going to repeat them here:

- Watchtower: do nothing only logs bad assertions.
- Defensive: stakes if detect a bad assertion (reactive)
- StakeLatest: keeps staked on the latest assertion, disputes if detect a bad assertio
- ResolveNodes: StakeLatest + confirms assertions that already passed the challenge period.
- MakeNodes: the more active, creates asertions and disputes them. This is needed also for a operative chain.


Looks like there is no incentive to be a validator, also its kind of closed system to be a validator because you need to be whitelisted. Also having no validator helps to avoid withdraws but not avoid to publish to l1. Well first: in order to make validation of the chain more suitable:

1. I can disable whitelist for validators.
2. If chain id changes, imediatly whitelist gets disabled.
3. If there are no active validators in a defined period of time (I set it to 3 months) anyone can call the whitelist function.

4. So in ortder to add a validator I have to update the config of my sequencer:

```json
 "staker": {
    "enable": true,
    "strategy": "MakeNodes",
}
```

Now I can see:

```logs
BoLD protocol is active, initializing BoLD staker
running as validator  txSender=0xff09...  strategy=makenodes
BlockValidator initialized  current=8a7513..56a499
Starting BOLD staker
Started challenge manager  stakerAddress=0xff09...
```


# 11 - 02 - 2026 

Well I got another error about the wasm module, the thing is `on-chain WASM module root did not match` this happened because when Io built and launched the chain using another has in my config file result of `make build-replay-env` and that was the one in the chain.
But when I build the image my custom nitro image using `docker build` it generates a new hash for the wasm module, and then when I launch the chain with that image it doesn't match with the one in the config file.
So I have to update the value in the chain, even if I am the owner I have to upgrade it via the `UpgradeExecutor`.

```sh
CALLDATA=$(cast calldata "setWasmModuleRoot(bytes32)" 0x8a7513bf7bb3e3db04b0d982d0e973bcf57bf8b88aef7c6d03dba3a81a56a499)

# call UpgradeExecutor
cast send 0x211Cf91bAE977914eAE40D1E4e755C44d76Dc9De \
  "executeCall(address,bytes)" \
  0x77E2752FBCfb3e131B4bc3a5A19C83438387eFEa \
  "$CALLDATA" \
  --rpc-url <url> \
  --private-key <priv>

# 0x211Cf91bAE977914eAE40D1E4e755C44d76Dc9De upgrade excecutor
# 0x77E2752FBCfb3e131B4bc3a5A19C83438387eFEa # rollyupproxy

# verify
cast call 0x77E2752FBCfb3e131B4bc3a5A19C83438387eFEa \
  "wasmModuleRoot()(bytes32)" \
  --rpc-url <url>
```

Then we are ready to deploy the new docker image:

```sh
# build the image
docker build -t capu-nitro:v3.9.5 . 

# verify the new hash
docker run --rm --entrypoint cat capu-nitro:v3.9.5 \
    /home/user/target/machines/latest/module-root.txt

# tag the image
docker tag capu-nitro:v3.9.5 100.64.0.2:5000/capu-nitro:latest
docker push 100.64.0.2:5000/capu-nitro:latest
```

# 10 - 02 - 2026

I notice some error in the explorer and the reason why it wasn't working:

```sh
2026-02-10T00:50:19.584 application=indexer fetcher=coin_balance count=496 error_count=496 [error] failed to fetch: 0xa4b000000000000000000073657175656e636572@: (-32000) missing trie node
    251e62db614a22ec8c5f80bef521117086a8403c2814f84afbeb2b5f1007be93 (path ) state 0x251e62db614a22ec8c5f80bef521117086a8403c2814f84afbeb2b5f1007be93 is not available, not found
```

Missing trie node ... which means my node is not archive, and make sense because when I migrate it, I didn't move the data just started a new docker compose and transactions worked. I though the state was build by default but looks like in order to run it archive I have to enable a flag:

```json
 "execution": {
    "caching": {
      "archive": true
    },
    ...
  }
```

For now its pretty fast there not many blocks, but I think I have to find a elegant way to make backups.

Also used this image which had support for arb, my custom didnt work `ghcr.io/blockscout/blockscout-arbitrum:latest`

This blockscout proyect set ups a los of services i am going to list them here:

- nginx: 80
- fronted: 3002 (which we need to change)
- backend 4000
- smart contract verifier: 8150
- sig provider: 8151
- visualizer: 8152
- stats: 8153
- postgeres dbs (two instances), redis db.

those are not accible by default the only one exposed is 80 with ngnix, it uses the other internally

- /api/* -> 4000
- /socket/* -> 4000

And its working now but there is a iframe with ads, I think I can remove it, let me see

```
NEXT_PUBLIC_AD_BANNER_PROVIDER=none
NEXT_PUBLIC_AD_TEXT_PROVIDER=none
```

Just needed to add two more env variables in my config

and then: `docker compose -f docker-compose-no-build-frontend.yml up -d --force-recreate frontend `

By default its this banner `slise` there are other options and also I can put my custom ones

At the end changes in `common-blockscout.env`

```init
ETHEREUM_JSONRPC_VARIANT: geth 
ETHEREUM_JSONRPC_HTTP_URL: <ip-vpn>:8547
ETHEREUM_JSONRPC_TRACE_URL: <ip-vpn>:8547
 INDEXER_DISABLE_PENDING_TRANSACTIONS_FETCHER: true # since arb do not have trational mempol
```

`common-frontend.env`

```
NEXT_PUBLIC_AD_BANNER_PROVIDER: none
NEXT_PUBLIC_AD_TEXT_PROVIDER: none
```

`docker-compose-no-build-frontend.yml`

```init
image: ghcr.io/blockscout/blockscout-arbitrum:latest
CHAIN_TYPE: arbitrum 
```

And finally my blockscout its working, and I dont have to build anything.


# 09 - 02 - 2026

Today I am going to prove the validation node. But I shutted down my eth sepolia daemon and now I have a buch of logs errros in the sequencer, I just need to wait.

Meanwhile I am going to launch my blockscout implementation. Arb already has one so I amjust going to modify it to work with my chain.

Hmm seems its not just plug and play also not sure if its ngnix config, there are many services that the docker compose launch.
And actually it make sense because it has to index transactions.

# 06 - 02 - 2026

Continuing with the bundler implementation:

- **beneficiary** : where the over gas fees will go.
- **maxBundleGas**: maximun gas to pay for mempool.
- **minStake**: min stake in factory and paymaster in entrypoint to be used by the bundler. Also mitigation of DOS attacks.
- **autoBundleInterval**: how often the bundler will bundle the transactions in seconds.
- **autoBundleMempoolSize**: how many transactions the bundler will wait before bundling them.
- **gasfactor**: multiplier to the gas price to pay more to miners.
- **whitelist**: paymaster and factory addresses that do not require minStake

I have to mention that this bundler repo also uses `.gitsubmodules` too, after all submodules won.

I discovered that the paymaster also need to stake some eth.. which makes sense.

And finally a bundler is running and all the test passed.

Got another errror in the chain about wasm module. looks like it takes the wasm compilation from another file. TO fix it I have to build my nitro image with `docker build --target nitro-node-dev -t capu-nitro:latest .`


# 05 - 02 - 2026

Okay I haven't deep dive in the changes that I am working on, because its all about testing, also real steps to replicate I am going to write them in the `guides` folder.

So today I am going to move the sequencer to another machine, and keep mine for testing and development.

I have to add that machine in another location to my proxy, also I am going to host a custom docker registry to store the images that takes many resources to build.
I am using headscale and I thougth it was going to be a faster connection, actually ssh behaves really well, but docker pull from that machine is slow. I have to investigate why. Probably because DERP server, but idk. Soon I am going to add my own DERP server with no limits.

The move was succesful, its incredibly that the sequencer can reconstruct the state from the L1 and the rollup chain info only. No need to backup anything. Also its pretty light, the server looks like if its not doing much work. I have to monitor the resources.
The only stuff to move are the config files and of course the most important and critical info its the private keys. From the batchposter.

Those keys are the only stuff that protects anyone else to "fake" my l2, because all the other info in the config files are public.

Now I have another mission today. Whats if for eip7702 we can sponsor gas fees too.. so that the user don't need to have two wallets. Looks like yup its possible  but I need to be more careful with the bundler.

This https://github.com/eth-infinitism/bundler looks like pling and play. I am going to modify to add security to be only used by the ones that I want.
O dam there are many parameters to configure. I need to read more about it. 
There also is a sdk for integration definilty easier to integrate.

# 04 - 02 - 2026

Looks like its going well, I am going to be adding more stuff progressively.

As I mentioned yesterday I just tested with the `TestPaymasterAcceptAll` from the eth-infinitism repo. Its a good start to understand how the flow works.

My next test was about whitelisting ERC20 tokens that the paymaster will accept to pay for gas fees.
It worked.

Now I am looking for writing the last free tx for the user. Looks like the only way to record this information in the smart contract is storing a list inside the contract itself.  It worked.

Now lets add the complex fee logic. It worked.

Finally I want to test the eip-7702. Fourtunately its only a more test that I need to implement, arb already supports it. Hmm the version that I deploy is 32 this eip is supported from version 40. And lates version is 50.

```sh
cast send 0x0000000000000000000000000000000000000070 \
    "scheduleArbOSUpgrade(uint64,uint64)" 50 1 \
    --rpc-url http://localhost:8547 \
    --private-key <chain_owner>
```


# 03 - 02 - 2026

There is a lot of information about erc4337, and I also found many implementations. Idk which one to use for my project. Let's see if I can find a good one.

So far I've seen Alchemy has put a good effort explaining the concept:

- https://www.youtube.com/watch?v=FjK5rYznJjU -> good and simple intro

Okay I think that I am going to implement my own smart contracts based on the ones from infinitsm. https://github.com/eth-infinitism/account-abstraction 

Definitly using this repo as reference its a must. So my first test was:

Deploying the EntryPoint contract.

1. First check with de `TestPaymasterAcceptAll`. And fourtunately it worked in the first try. I create also another 2 wallet to test the UserOperation flow, and yup the user was able to send usdc from one wallet to another without having eth in the wallet. (smart wallet to be precise).
2. Then I think I am going to try `TestPaymasterWithSig` I need to have more control about who can use the paymaster.
