# 23 - 01 - 2026

Well I am still pending to test the validator role and make a false tx to be challenged. I just wanted to live that process haha.
Also test the withdraw process from l2 to l1.

Now, I am facing some problems, the objective of this proyect is to offer a l2 with low gas fees, even to point that under certain conditions gas fees are 0 (okay it could be). And another feature that I need to implement is to be able to pay gas fees with another tokens, not only eth. A bunch of tokens. (at least 3 plus eth).

The second one looks harder than the first one. But looks like solving that one I will be able to check or partially solve the first one.

Lets explain what I have found:

- It is posible to pay gas with a custom token. ONLY 1 token by default. Has to be a erc20 token under certain conditions.
- However the gas fee is calculated in a exchange rate l1 terms. This make sense since the base is l1 gas fees, there is where the settlement is done. In other words "fees must be converted to the parent chain's token for data availability costs"
- This make me question: what if the custom token has no value?:

Arb has documentation about this [choose custom gas token](https://docs.arbitrum.io/launch-arbitrum-chain/features/common/gas-and-fees/choose-custom-gas-token). And the advert that "Significant prices declines, could lead to operational deficits"

- Its really hard to change the token once its deployed.
- Also existing smart contracts that asume eth will need to be adapted, definitly we need to accept eth as gas fee token too.
- The sequencer do no swaps itself, so if the custom token has no value, the sequencer will not be able to pay l1 gas fees. So someone need to swap those tokens to eth and fund the sequencer address with eth.

Here I found some options (not all of them are useful for me or my case but I want to compare them):

- **op 1**: Subsidize eth settlement gas, however the user still needs to have some capu token in order to make txs. And I am sure that subsidizing eth is not going to be sustainable or somehow could be a problem.
- **op 2**: Use a token with value, for example usdc.
- **op 3**: Avoid the posting fee usign anytrust instead of rollup but this option make the protocol less secure.
- **op 4**: EIP-4337 account abstraction, this is a new concept that I need to explore more in depth. Also looks like this need anothe type of wallet (supported by this nitro version). And looks like I have to check the EIP-7702
- **op 5**: I can set `setL1BaseFeeEstimate(0)` thus the user will only pay for l2 execution gas fees. I will need to pay eth settlemet.
- **op 6**: Create a custom mecanism to swap capu tokens to eth in order to fund the sequencer address. This could be done using a DEX or a CEX. However this mecanism could be complex and could introduce new risks.

Found the code part where I could change the transaction conditions `arbos/extra_transaction_checks.go`

- https://docs.arbitrum.io/launch-arbitrum-chain/configure-your-chain/common/gas/use-a-custom-gas-token-rollup
- https://docs.arbitrum.io/launch-arbitrum-chain/configure-your-chain/common/gas/gas-optimization-tools

# 22 - 01 - 2026

Continuig with the deploy of nodes, I keep updating in the guides folder, a lot of new commands that I've discovered while configuring the sequencer.

And also time to explore the new concept of `bridges`:

- Brige is a mechanisms to move actives between l1 and l2.
- You lock eth on l1 and use in l2, when you burn it in l2 it can be freed in l1.
- User sends eth to a l1 bridge contract. Then l1 contract creates a message to l2 bridge contract.
- Sequencer reads the message and credits the user in l2. But the real eth is locked in l1 bridge contract.
- For withdraws the user creates outbox message to l1 bridge contract.
- Waits for challenge period.
- After challenge period the user can call l1 bridge contract to free the eth.

This is different from stake token.

Okay so I send a txs to the inbox contract to bridge some eth but I found some interesting things:

1. batch poster config

```json
  "batch-poster": {
    "max-delay": "30m",
    "max-size": 100000,
    "compression-level": 11
  }
```

- `max-delay`: maximum time to wait before posting a batch, even if not full. When its full automatically posts.
- `max-size`: maximum size in bytes of a batch. When reached automatically posts. So it do not waits `max-delay`. The higher less batches but more delay, the lower more batches but less delay.
- `compression-level`: brotli compression level, from 0 to 11. Higher is better compression but slower. This is the max level. There is no reason to use lower levels unless you want faster compression with less efficiency.

`max-size` and `compression-level` where not configured the use the default values.

Also useful commands to debug contracts. I am going to put them in the guides. In cast.

About those configurations are  differets from the ones in `contracts/scripts/config.ts` those are the ones that cant be changed after deployment (later). Are publci because the other ones are privates and in this way is the anti-censorship mecanism. Those are the security rules.

About contracts and roles, now is more clear for me:

**L1 Contarcts**:

- Inbox: recibe deposits from users.
- Bridge: locks funds and creates messages to l2.
- Outbox: processes messages from l2 to l1.
- Sequencer Inbox: Recibe batches
- Rollup Proxy: status rollup

**L2 Nodes**:

- Sequencer: get txs, order them, create batches.
- Batch poster: read l2 blocks, compres and post batches to l1 sequencer inbox.
- Delayed sequencer: read l1 inbox messages, process them in l2.
- Validator: needs stake token and post assertions to rollup proxy


Perfect so for now everyone can bridge eth which is the gas fee token. And we should be able to use my l2. I officially were able to bridge capu.eth to l2.

# 21 - 01 - 2026

Okay it worked and cost 0.33 eth in sepolia, give a output `deploy.json` and `l2_chain_info.json` in the `contracts` folder.
Those files are not commited, however they have no sensitive information.

I have to check a place to run my tests and also try to configure the sequencer.

I am going to be writing the steps in ![./guides/05-sequencer.md](./guides/05-sequencer.md) so I dont repeat them here.

Pretty obvios but I will need a eth daemon rcp. Three main reasons:

1. batch poster: to publish batches
2. sequencer: read l1 direct deposits
3. validators: to read l1 state and compare

# 20 - 01 - 2026

Today I feel more energized, keep continue yesteraday was hard hahaha.

1. First issue: The deploy ignores my max gas limit, this is because the deployt script creates its own provider:

```ts
// scripts/local-deployment/deployCreatorAndCreateRollup.ts
const deployerWallet = new ethers.Wallet(
    deployerPrivKey,
    new ethers.providers.JsonRpcProvider(parentChainRpc)
  )
```

Not yet sure why they create a new provider insted of using the one from hardhat config that already has the gas limits.

I am going to change this limits too in the deploy script:

Option 1:
```ts
const provider = new ethers.providers.JsonRpcProvider(parentChainRpc, {
    maxPriorityFeePerGas: ethers.utils.parseUnits('0.1', 'gwei'),
    maxFeePerGas: ethers.utils.parseUnits('0.6', 'gwei'),
  })
```

However looks like there are more places where I need to set gas limits.

But first may be interesting to deploy to some testnet or mocknet to see how much gas it uses. For future reference.

Back here again, I tried to run without setting gas limits and no matter how much eth I put in the deployer address it always run out of gas but only for the rollupCreator. (You will see my attemps in step 3).

Option 2: for now I am going to set gas limits directly in the createRollup function:

```ts
// /scripts/rollupCreation.ts
const createRollupTx = await rollupCreator.createRollup(deployParams, {
       value: feeCost,
       gasLimit: 7750000,
     })
const createRollupReceipt = await createRollupTx.wait()
```

2. I also didn't change this variable `isDevDeployment -> false` for the script in `scripts/local-deployment/deployCreatorAndCreateRollup.ts` in order to make it read my config.ts file. But if I change this variable the testnode may break since it deploy contracts too.

Lets see for the test-node in `nitro-testnode/test-node.bash` the docker-compose command injects `-e CHILD_CHAIN_CONFIG_PATH="/config/l2_chain_config.json" -e CHAIN_DEPLOYMENT_INFO="/config/deployment.json" -e CHILD_CHAIN_INFO="/config/deployed_chain_info.json" rollupcreator create-rollup-testnode`

I dont want to mess whith other repo so a simple change for my deployment:

```ts
// scripts/local-deployment/deployCreatorAndCreateRollup.ts : 92

process.env.IS_DEV_DEPLOYMENT !== 'false'

```

```env
IS_DEV_DEPLOYMENT=false
```

`false` is not different from `false` so the value sent is going to be `false` (its not dev deployment). And for testnode `undefined` is not equal to `false` so the value is going to be `true` (its dev deployment). Perfect.


3. Not an issue but a fact is that I will not test directly in mainnet, actually I didnt want but others ecouraged me to do it hahaha.
And I dicoveres a feutre of hardhat that I didnt know before, the forking feature. So I can test in mainnet fork first.
This is different from testnet or mocknet since I will be using real mainnet state. I think I am going to be using this feature a a lot.

```sh
cd contracts
npx hardhat node --fork <RPC-URL> --no-deploy

# in another terminal
curl -X POST http://localhost:8545 -H "Content-Type: application/json" \
    -d '{"jsonrpc":"2.0","method":"hardhat_setBalance","params":["OWNER_ADDRESS","0x56BC75E2D63100000"],"id":1}' 
# 0x56BC75E2D63100000 is 100 eth in hex

PARENT_CHAIN_RPC=http://localhost:8545 npx hardhat run scripts/local-deployment/deployCreatorAndCreateRollup.ts --network localhost
```

Damm error again:

```log
eth_sendRawTransaction
  Contract call:       RollupCreator#createRollup
  Transaction:         0xeba2073907f43e14ba79dbb68e9b8650b486cda561c07d8662d6871cfaee7289
  From:                0x8b36f5a6f4e88c3a98598b92b28178b772c2d2a7
  To:                  0xe9e70a82e1bca739a8f1abbbfc96ae0f3b709151
  Value:               0.13 ETH
  Gas used:            7687989 of 7687989
  Block #24278540:     0xac318059db535a0068d66e32154cbbd6ed54eb605983a9bf5881c5c86cee8cc0

  TransactionExecutionError: Transaction ran out of gas
```

And it is `Transaction ran out of gas` again. Hahahahah this is personal now, even fake eth in forked mainnet is not enough for me... this should be a joke.
Read that hardhat fork has do no calculate gas instead sets high ammounts by default.

Lets add 100M fake eth to the deployer address:
`curl -X POST http://localhost:8545 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"hardhat_setBalance","params":["ADD","0x52B7D2DCC80CD2E4000000"],"id":1}'`

And again ... same error. Its not about how much eth I put there its about gas limits.

```ts
// scripts/rollupCreation.ts
const createRollupTx = await rollupCreator.createRollup(deployParams, {
       value: feeCost,
       gasLimit: 7750000,
     })
     const createRollupReceipt = await createRollupTx.wait()
```

and finally got:

```log
Congratulations! 🎉🎉🎉 All DONE! Here's your addresses:
```

With that gas limit it costed `0.368` fake eth in mainnet fork.

Well this make me think that probably is the same stuff in mainnet its about gas limits. That I looks like I still do not know, how does gas works.

Lets try for sepolia with no changes

```env
PARENT_CHAIN_RPC=<>
PARENT_CHAIN_ID=11155111
STAKE_TOKEN_ADDRESS=0xfff9976782d46cc05630d1f6ebab18b2324d6b14 # WETH sepolia
```

```sh
cd contracts
npx hardhat run scripts/local-deployment/deployCreatorAndCreateRollup.ts --network sepolia
```

Lets wait to get enough eth in sepolia faucet.


# 19 - 01 - 2026

Dam its really hard for me to get the same entusiasm on mondays. It is suppoused that today its the clasic "blue monday", for me every monday is blue hahaha.

Okay I couldn't finish the deployment because hardhat set a high ammount of gas for the deployment for smart contracts, so as a permanent advice:

    always remember to set manually a maximun gas fee limit when deploying to mainnet.

I looked for some test eth, looks like some internet places give them for free (a few) and others sell them.

Now I have a quicknode paid rpc to avoid some weird limits.

If this deployment fails to work, I will try to deploy in a eth testnet first. I dont want to burn more real eth. `.env` file for stagenet:

```env
CHILD_CHAIN_NAME=capu-stagenet
DEPLOYER_PRIVKEY=# privkey remove 0x, signs tx in script
MAINNET_PRIVKEY=# privkey remove 0x, signs tx in hardhat
PARENT_CHAIN_RPC=# l1 rpc script
MAINNET_RPC_URL=# l1 rpc hardhat
PARENT_CHAIN_ID=1
STAKE_TOKEN_ADDRESS=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 # WETH
ETHERSCAN_API_KEY=# hardhat verify contracts
```

and the most importan config in `hardhhat.config.ts` is:

```ts
{
maxPriorityFeePerGas: 100000000,  // 0.1 gwei
maxFeePerGas: 600000000, // 0.6 gwei
}
```

Finally within `contracts` folder run:

```sh
npx hardhat run scripts/local-deployment/deployCreatorAndCreateRollup.ts --network mainnet
```

Daaam ... something went wrong again... The deployment failed because of insufficient funds. I think I need to fund more the deployer address. But I already put more than 0.1 eth... Looks like the deployment ignores my gas limits.
Well I have to learn somehow that mainnets are painfull to work with hahaha. So my next try is going to be in testnet first, I also saw another errors. Which of course could be preferable to test in testnet first.



# 16 - 01 - 2026

I haven't used foundry before and looks like a great tool for blockchain development. Continuing with the deployment process:

Well for the deployment they use hardhat instead, I've used hardhat before so no problem there.

1. create a set of 4 walleets
1. fund de owner address
1. `make build-reply-env` this will create the `machine.wavm.br` and `module-root.txt` in `target/machines/latest/machine.wavm.br`: the execution machine of L2 compiled to wasm and compressed with brotli.
1. in  `contracts` run `pnpm i` I prefer pnpm over npm or yarn.
1. in `contracts` repo `cp scripts/config.example.ts scripts/config.stagenet.ts` and modify the `config.stagenet.ts` with the new chainid, adresses and new to changevalues. There are many configs there and it looks like there something called proxy to update those values in the future without redeploying everything. But not sure at all (later).

```json
baseStake: 0.001,
wasmModuleRoot: 'hash'
owner: '0x...',
loserStakeEscrow: '0x...',
chainId: 662201,
InitialChainOwner: '0x...',
validatorAfkBlocks: * 3,
miniStakeValues: [
      0,
      ethers.utils.parseEther('0.0005'),
      ethers.utils.parseEther('0.00025'),
    ],
validators: [
      '0x...'
    ],
batchPosters: [
      '0x...'
    ],
batchPosterManager: '0x...',
```

Make sure to run `cp config.stagenet.ts config.ts` before deploying. Because the deploy script uses `config.ts`.

Deploy command:

```sh
npx hardhat run scripts/local-deployment/deployCreatorAndCreateRollup.ts --network mainnet
```

As usaual the env variables requiered for deployment are in `hardhat.config.ts` so we need a `.env` file in `contracts` folder like this:

```env
CHILD_CHAIN_NAME=capu-stagenet
DEPLOYER_PRIVKEY=#privkey remove 0x
PARENT_CHAIN_RPC= # rpc
PARENT_CHAIN_ID=1
STAKE_TOKEN_ADDRESS=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2
```

For RPC I am going to use a free one. For STAKE_TOKEN_ADDRESS I am going to use WETH address in mainnet. I think I am going to need to redeploy soon if I want to use another token for staking. Probaly this is part of the changes that do require a full redeploy.
For now will chose WETH.

Well for hardat deployments its usefull to have an account a apikey in [etherscan](https://etherscan.io/) in order to verify the contracts after deployment.

Woops something went wrong... The contracts werent deployed (not all of them).


# 15 - 01 - 2026

Well looks like each GO PROC takes around 24gb of ram, so with 32gb ram computer only 1 proc can run.

I am going to add more swap:

```sh
sudo fallocate -l 16G /swapfile2
sudo chmod 600 /swapfile2
sudo mkswap /swapfile2
sudo swapon /swapfile2
# verify
free -h
```

Okay I am going to try with `GOMAXPROCS=2 make -j1 test-go`

- Today I found the notice that even linus torvalds is using AI haha, he was reluctant at the beginning but finally accepted it. To be honest this make feel better about using AI tools for coding.

- Talking about test it worked with `GOMAXPROCS=2 make -j1 test-go` but pretty slow, I ncrease even more the swap and try with `GOMAXPROCS=4 make -j2 test-go` and lets see how it goes, specially in this RAM crisis period.

Dam even `GOMAXPROCS=2 make -j1 test-go` make my computer unresponsive. Well run this test is going to be painfull until I get a better computer.

Lets continue with smart contracts and necesary configs for rebrand. For now I will not use eth testnet, I am going to go directly to eth mainnet, but it is going to be a test so a testnet on mainnet which I am going to call it a stagenet. And later on I will make teset on testnet.

About the chainid, looks like there are 3 main sources of chain id list:

- [chainlist.org](https://chainlist.org/)
- [ethereum-lists/chains]( https://github.com/ethereum-lists/chains)
- [chainid.network](https://chainid.network/)

This will be important when I deploy to mainnet, in order to make everyone know about the new chain (later).

- Chain id for stagenet: `662201`


# 13 - 01 - 2026

I am still not used to work using submodules, so in order to update a submodule to a new commit, I have to go to the submodule folder, checkout the desired commit, and then go back to the main repo and commit the change.

```sh
cd nitro-testnode
git fetch origin
git checkout <new-commit-or-branch>
cd ..
git add nitro-testnode
git commit -m "Update contracts submodule to new commit"
git push
```

Just to make sure run again `make build` and `docker build -t capu-nitro:latest .` to verify that everything works fine.

It does. Now lets try:

```sh
cd nitro-testnode
./test-node.bash --init --dev
```

And it works fine too. 


```log
INFO [01-14|18:27:34.097] created jwt file                         filename=/home/user/.arbitrum/jwtsecret
INFO [01-14|18:27:34.097] Running Capu nitro node                  revision=development vcs.time=development
```

But looks like it is still using `.arbtrum` in the folder. So lets `rg '\.arbitrum'` to see where this name is used. And try again.

```sh
cd nitro-testnode
./test-node.bash --init --dev --build # to test everything (build included)
```

It takes sometime and trigger fans, but finally in the `nitro-testnode-sequencer` I see the logs:

```log
INFO [01-14|19:38:08.511] created jwt file                         filename=/home/user/.capu/jwtsecret
INFO [01-14|19:38:08.511] Running Capu nitro node                  revision=development vcs.time=development
```

Finally just to make sure everything is working fine, lets run the test suite, again good code organization, everything is in `Makefile`. But first you need to install a lot of dependencies, check [!./guides/01-setup-env.md](./guides/01-setup-env.md)

```sh
make test-all # this will trigger your fans :) and take some time
```

Woops looks like some test break my terminal hahah and I cant not see where does it fails. Lets run one by one to find the problem. Dam I cant belive I am going to need more resources to develop in this project.

`-j <N>` flag is to limit the number of parallel jobs. Lets see if this maybe helpfull.

```sh
make clean 
make -j6 build-node-deps # ok
make -j6 test-go-deps # ok
make -j6 test-go # failed
make -j6 test-rust # pending
make -j4 test-go-challenge
make -j4 test-go-stylus
make -j4 test-gen-proof
```


# 12 - 01 - 2026

- Well today I found that the repo `safe-smart-account` moved from `https://github.com/safe-global/safe-smart-account/` to `https://github.com/safe-fndn/safe-smart-account/`, looks like they moved some repos to a new fundation `https://safe.global/` -> `https://safefoundation.org/` and they referenced also the x account `https://x.com/safefndn`.
- Nvm git does automatic redirects when a repo is moved, so no problem there.
- I didn't explain next steps for rebrand so here it goes:

After the mirror for nitro and nitro-contracts, udpate `.gitmodules` to point to the new repos in gitea.

```sh
git submodule sync
git submodule update --init --recursive
make build # this will trigger your fans :)
```

We can talk about `Makefile` now, around 600 lines, the `build` command triggers a build chain of `yarn`, compile contracts, compile go bindings, brotli, cbingind, rust, jit, go binaries.

I had to make a fix, not sure how do this works for them, I see there prs about `lint_on_build = false` for `foundry.toml` in contacts repo [here](https://github.com/OffchainLabs/nitro-contracts/pull/408/files#diff-bac743fe3d899998e32393af53af8cc7d28c89c3d7fabfa48b01361cf8d42d9b) so eventually it will get merged.

So applied the fix to `contracts`, `contracts-legacy` and `contracts-local`.

Make `git clone` for first time in another computer, and found some problems with submodules, I think I am not being fan of submodules. Well here are the steps:

```sh
git clone --recursive <>
cd nitro
git checkout develop
git submodule sync --recursive
git submodule update --recursive
make build
```

Finally when the build succeeds we can find the binaries in `target/bin/` folder. The principal of course is `./target/bin/nitro --help`.
But there are other useful binaries like: `nitro-val`, `relay`, `deploy`, `da-server`, `el-proxy`, etc.

Before configuring `arbitrum_chain_info.json` for real mainnet o testnet deployment, I am going to test it locally. Lets build now using docker.

```sh
docker build -t capu-nitro:latest .
```

Now its time to modify `nitro-testnode` repo, basically the same mirror commands, and point to my custom nitro.

Change my custom contract version

```sh
# test-node.bash
DEFAULT_NITRO_CONTRACTS_VERSION="custom-branch-or-commit"
```

Just face the problem that docker clones public repo, but for now my fork the repos are private, so I am going to need to copy code instead of cloning.

But `nitro-testnode` its a submodule and it should work alone. Since docker is its own enviroment I need to bridge my ssh key or somehow give access to the repo.

The other option is that we are going to have the limitation that you can only run `nitro-testnode` only from nitro and not in the submodule.
I think I am going to go with this last option for now, since I am going to be the only one making changes to the codebase.

```yml
# nitro-testnode/docker-compose.yml
rollupcreator:
  context: ../  # instead of cloning, use the parent folder
  dockerfile: nitro-testnode/roullupcreator/Dockerfile
```

```Dockerfile
# roullupcreator/Dockerfile

# RUN git clone --no-checkout contracts
# git checkout 
# git submodule update --init --recursive 
COPY contracts ./
```


# 09 - 01 - 2026

- Okay I am going to mirror offchainlabs repos and keep track of the main branches so:

contracts:

```sh
git clone --mirror git@github.com:OffchainLabs/nitro-contracts.git
cd nitro-contracts.git
git remote add bk <>
git push --mirror bk

# to update origin in mirror mode
git fetch origins
git push --mirror bk

# for gittea, since gittea dont like pull requests refs
git remote add gittea <>
git for-each-ref --format='delete %(refname)' refs/pull | git update-ref --stdin
git push gittea --mirror
```

Okay so mirror brings the git objects but not the working tree, fortunately gittea know how to handle bare repos. And this was only needed beacause of those random referenced commits, in my workflow I will only be watching for those main branches. Something like:

```sh
git clone gittea
git remote add arb <> # offchainlabs repo
# for updating
git fetch arb
git checkout main
git merge arb/main # or git rebase arb/main
```

Now I can delete mirror repo `cd .. && rm -rf nitro-contracts.git`. And finally I have all the commits I need. Hmmm not sure if this was an overkill but now I have all the history.

- However I am going to point to another commit, since I have to modify a simple file. But well at least use the same version as offchainlabs.

- Wait a minute, just today the fixed the commit reference to the latest tag in legacy contract hahaha. Okay I need to use nitro in the correct version.

Another useful command is `git tag --sort=-creatordate -n`

```sh
git checkout v3.9.5
git submodule status # to see the commit references
```

Then created my custom branches for those submodules, since they do no have tags in the commits they are using. And actually I am worried about future updates, but well since the change y this repo is minimal (just one file) for now. A cherry pick is going to be enough.

- Apply the same mirror process to `nitro`. And lets continue.


# 08 - 01 - 2026

- Definitly faster to make this soft-rebrand changing only the superficial names, and leaving the internal logic as it is. This way we can mantain the codebase more easily with future updates from nitro.
- This soft rebrand was made in `3.9.4` intentionally, arbitrum nitro is now in `3.9.5` so we can test the update process too.
- Perfect, basically a `git rebase v3.9.5`, 0 conflicts. This was an easy one.
- The final list to keep track of custom submodules is going to be: `contracts`, `contracts-legacy` and of course `nitro`. Still thinking if it is work to track the 14 other repos that are referenced in the main nitro repo. 
- Lets make this progressive so for now only 3 repos are going to be tracked as submodules.
- Since I am selfhosted proficient now I am going to have antoher backup, not using gittea, instead bare git repos in a vps that I own. Just in case. But as in the cloud I am going to be backing them up as soon as I need it.
- Something I didn't know is that when you are working with submodules, you have to ways to bring changes or update files: 1. In the repo itself and the pull changes to the main repo. 2. In the main repo, and then go to the submodule folder and push or pull changes from there. Both ways work. Of course you need permissions to to that.

```sh
git clone https://github.com/OffchainLabs/nitro-contracts
cd nitro-contracts
git rename origin arb
git remote add origin <>
git push -u origin main
```

- I think I am going to keep track of offchainlabs repos in `arb` branch pointing to their main branch (later).
- Looks like the commits they are using are not in the main branch at least for legacy contracts. For legacy contracts since march 2025 they do not change the reference. But the commit [5f61a6b0e9799d7ccc0b3811f3893c68a14e43bf](https://github.com/OffchainLabs/nitro-contracts/tree/5f61a6b0e9799d7ccc0b3811f3893c68a14e43bf) has not tag or anything else. There are 13 commits ahead of the last tag and many line changes.


# 07 - 01 - 2026

- Today something came into my mind, and it is, looking in all the files that the AI change and recompile with a panic error given by chaging the name in the smart contracts.
- Probably insted of changing all the files files, I just need to change the basic names in order that the final user feels the change but the internal logic can remain the same, in this way its going to be easier to mantain the code with future updates from nitro. And as far as I dominated the codebase and we implement custom features, we can change the internal logic later.
- Lets make a plan for this. And see how many changes are needed to make the rebrand only superficial and upgradeable friendly.
- I wanted to point out [headscale](https://headscale.net/) as a selfhosted alternative to tailscale, I have it running and finally can say that I overpassed NAT problems,also dicovered that DERP servers exist and tailscale provides free ones.

# 06 - 01 - 2026

- We have a custom gitea repo [git moca](https://git.moca.app/Moca) I am going to migrate the nitro structure there with the submodules approach too.
- I have to learn about `git submodules` but with differente origins
- Noticed that id `42069` and `69420` are alredy taken, checked in [chainlist.org](https://chainlist.org/). No problem we are going to use `6622`. But for testing I am going to be using `69420` in testnet and see how complicated is to send upgrades.
- Second try with the new instructions to replicate the rebranding process. And compare with another agent to see differences.
- While claude works I am going to keep reading docs I think that this is how developers will need to work in the near future.


# 05 - 01 - 2026

Hi everyone, I am back after a long break, the holidays were great! I hadn't took a break in a while so it was much needed. Now I am ready to continue with the Arbitrum project.

- I've seen a lot of updates about Claude: [Claude Continues v2](https://github.com/parcadei/Continuous-Claude-v2) and one of my predictions came true, someone already released [Claude Code Workflow Studio](https://github.com/breaking-brake/cc-wf-studio/) and [Claude How To](https://github.com/luongnv89/claude-howto) the most introductory repo. Another one that I've installed is [Claude HUD](https://github.com/jarrodwatts/claude-hud)
- Time to deep dive into the Arbitrum codebase again. Before this project turns to be "vibecoding arb fork"
- Looks like my claude actually worked better than expected, I will keep using it to help me with the project.
- Today I also create in this repository a CRON git action to check for updates in the arbitrum nitro repo twice a day.
- Selfhosted my first rss feed in order to check arbitrum updates (I really wanted to use rss but didn't find a real use case until now).
- Also requested calude to create a manual for future merges, probably will be nitro updates harder to integrate, beyond the rebrand. Especially if there are changes in the sequencer logic. (later).
- Requested to create a guide for rebranding arb, and lets apply it on my own.

