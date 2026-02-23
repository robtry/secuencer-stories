# 20 - 02 - 2026

This was the fisrt time I consume all the tokens available in my claude plan, I had to wait around 2 hours to continue, it could mean one of three things:
1. I am finally paralelizing the work and consuming more tokens, which is good.
2. The task I asking for help are more complex and require more tokens, which is also good because I am asking for more complex things. And more important doing real things instead of simple code.
3. None of the above and this new model version takes more tokens to process the same things.

Today the team has an important showcase. So I couldnt show them my poc.
It worked and it is fun to be backtracing all the txs and see how the flow works.
 
# 19 - 02 - 2026

So far my conclusions are defintly for running the test I need at least 64 gbs of ram.
I see that nitro in the CI/CD use:

```sh
export GOMEMLIMIT=6GiB
export GOGC=80
make test-go
```

With that config I was able to reduce 20gb of ram thus making it possible to run the test with 40 gbs. I am going to phisically reduce the ram and test again using 44 gb to see if is persistent.

- Yup persistent, now I am going to test reducing the cpus to 5.

Today also I am going to debug the vibecoded fronted.

I noticed that those contracts are not verfied in the explorer, but it has a lot of options about hw to verify them many tools that I havent hear before. I am going to use the well know `hardhat`. The contracts are in the aa repo and the other one its the token.

```sh
npm install --save-dev @nomicfoundation/hardhat-verify@^2 --legacy-peer-deps
```

in `hardhat.config.ts`

```ts
import '@nomicfoundation/hardhat-verify'

// config object
etherscan: {
  apiKey: {
    capuchain: 'empty'   // Blockscout do not need it
  },
  customChains: [
    {
      network: 'capuchain',
      chainId: 662201,
      urls: {
        apiURL: 'http://100.64.0.5/api',
        browserURL: 'http://100.64.0.5'
      }
    }
  ]
}
```

The sintax is `npx hardhat verify --network capuchain <ADDRESS> [constructor args...]`

```sh
cd repo_code
# EntryPoint
npx hardhat verify --network capuchain 0x433709009B8330FDa32311DF1C2AFA402eD8D009

# SimpleAccountFactory (1 arg: entryPoint)
npx hardhat verify --network capuchain \
  0xAD07bbb7bEA77E323C838481F668d22864e9F66E \
  "0x433709009B8330FDa32311DF1C2AFA402eD8D009"

# Simple7702Account (1 arg: entryPoint)
  "0x433709009B8330FDa32311DF1C2AFA402eD8D009" \
  "0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7" \
  "5000000" \
  "10000000"

# CapuPaymasterSponsor (4 args: entryPoint, owner, minBalance, maxSponsored)

npx hardhat verify --network capuchain \
  0x5112FD5fd80455ed7Dd3c5ee67119D1473159E78 \
  "0x433709009B8330FDa32311DF1C2AFA402eD8D009" \
  "0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7" \
  "5000000" \
  "10000000"

# FakeUSDT (2 args: recipient, initialOwner)
cd repo_code_token
npx hardhat verify --network capuchain \
  0xbaCa5E7C88D887B61a25b8Bd507Eab9E8e348ee9 \
  "0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7" \
  "0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7"
```

About args they need to be same that when they were deployed, they are stored here `deployments/capuchain/*.json`

I noticed that my owner address its not the owner of the EntryPoint (0x433709009B8330FDa32311DF1C2AFA402eD8D009) it is `0x4e59b44847b379578588920cA78FbF26c0B4956C` what is that?
Well when the hardhat config has `deterministicDeployment: true` it deploys the well known Nick's Factory: https://github.com/Arachnid/deterministic-deployment-proxy

This is something I didnt know the trick for this proxy that works on all the evms and can be deployed with the same address is that the tx is presigned so the address has no txs in the chain hence nonce = 0 and the address is deterministic. So hardhat send some eth to that address to deploy the proxy.
The address who fund the presinged address is `0x2aE3a2085c469A91A8AceECd388e30168b95FddA` and this addres came since the block 1 so this might be part of arbirum. Its a `DeployHelper`

This a rabit hole of traces tomorrow I am going to make it easier to explain

# 18 - 02 - 2026

Oh no my services are down, my hdd 1tb drive its not enough for eth sepolia eth daemon. Since the sequencer depends on the ethe daemons its not working fine, and also the blockscout for some reason finished redis db and the proxy.
Well I have no other disk to move the data and I discovered that quicknode also exposes the beacon url. So I am going to use it.

Also see that the blocksout repo has no `restart: unless-stopped` in the docker compose for redis and the proxy. Not sure why, I need to add it.

About running the test I discovered interesting things (pasting this table if more confortable for me):

 | Config                  | Tiempo | Pico RAM | Swap  | Failures | Resultado |
  |-------------------------|--------|----------|-------|----------|-----------|
  | 6 cores / 64GB (cache)  | 39min  | 36.9GB   | 0     | 4 flaky  | Pasa      |
  | 5 cores / 40GB          | 34min* | 39.9GB   | 27MB  | 60       | Falla     |
  | 5 cores / 44GB          | 2h*    | 43.9GB   | 4GB   | 79       | Falla     |
  | 6 cores / 44GB          | 27min  | 44GB     | 8GB   | 71       | Falla     |
  | 5 cores / 49GB          | 46min  | 49.9GB   | 3.5GB | 17       | Falla     |
  | 6 cores / 49GB          | 33min  | 50.1GB   | 8GB   | 4        | ~Límite   |
  | 6 cores / 63GB (cache)  | 35min  | 58.4GB   | 0     | 4 flaky  | Pasa      |
  | 6 cores / 64GB (limpio) | 34min  | 62.2GB   | 0     | 2 flaky  | Pasa      |

That was the reason it ran succesfully with 64 gbs of ram for the first time. And it behaves persistent now, I need 64 ram just for this test.

For the blockscout I see that I can enable 4337 support under `/ops` using `ghcr.io/blockscout/user-ops-indexer:latest`

```sh
docker compose up -d user-ops-indexer
docker compose up -d --force-recreate backend
docker compose up -d --force-recreate frontend
docker compose up -d --force-recreate proxy
```

Today I also fullyvibecode a frontend poc, at the beginning the idea was to modify a exisiting open source wallet repo.
But it was kind of overkill, I just need to demonstrate in a UI the POC of the eip 4337 and eip-7702.
And I think its also clearer to me. The implementation of the bundler and the sings.

# 17 - 02 - 2026

Now with the previos values I am able to identify clearely the resoucer I am going to need.
Monitor the test with less resources to avoid surprises.

- GOMAXPROCS=1 took 2 hours and 39.7 gbs peak
- using 6 cores took 39 minutes and 36.9 gbs peak

Make sense to use less ram using more cores, because it can finish tests faster and free resources, so why I cant run my test with 32 gbs of ram and 16 threads?.

So far this are my conclusions:

| service | cpu | ram | storage |
| eth sepolia | 2 | 8 | 3 |
| sequencer, validator and batchposter | 1 | 3 | 0.5 |
| blockscout | 1 | 2 | 0.1 |
| bundler | 1 | 1 | 0.1 |
| test/cicd | 6 | 42 | 1 |

This gives a total of 11 cpus, 56 gbs of ram and 4.7 tbs of storage. Which are reachablevalues for a single machine. Also I am thinking in runnning this with proxmox so pretty good to round up to 16 cpus, 64 gbs of ram and 5 tbs of storage.

Now I am going to check which alternatives I have, for example a tower with single cpu, or a tower with 2 cpus, or multiple minipcs, or something in the cloud.

Well it took a lot of time being navigating on the stores and looking for prices and pieces. Half of the budget goes to ram and storage.

Now with a more clear about requirements, I can present this to the team and ask for the budget to buy the machine.

So now good time to test the frontend proof of concept.

# 16 - 02 - 2026

Well I still have pending to test all the assertions methods.

I am stil waiting for the first assertion, test because it has 6.4 days to be executed.

For this week I can make a test in the fronted side using the bundler, it might be a good test for the integration flow.
Also its a good time to retry to run the test, I got access to a more powerful machine, so I can run the tests in a more comfortable way and make a list to request for the pc that I am going to need to run all the test and the services.

I am also very happy to took my time implement headscale, now its pretty simple to add more server and handle the permissions, specially for moments like this.

On the other hand I can put in practice one of my guides to install all the depedencies and tools that I need to run the tests.

So It worked very well using 64 gb of ram and 6 cores. The commands to run the tests are in the guides, the most expensive are the go tests.
I ran the whole suite tests and it worked, I am not sure but I didnt see the ram go beyond 18 gbs, which no make sense.

Then I didnt remember if I was able to run the test with the go flags to use less jobs, So I tried again in the 32 gbs machine and it didnt work. Definitely I need more than 32 gbs to run the tests. Now I want to make sure how much is the less ram I can use.

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
