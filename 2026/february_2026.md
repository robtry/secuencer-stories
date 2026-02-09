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
