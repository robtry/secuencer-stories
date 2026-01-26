In order to run a l2 we need to deploy many contracts:

0. fund owner and batch poster accounts with ETH.

1. Set up .env file with the required variables:

```env
CHILD_CHAIN_NAME=capu-stagenet
DEPLOYER_PRIVKEY=priv_key_without_0x_prefix
MAINNET_PRIVKEY=priv_key_without_0x_prefix

# Sepolia
PARENT_CHAIN_RPC=https://ethereum-sepolia-rpc.publicnode.com
PARENT_CHAIN_ID=11155111
STAKE_TOKEN_ADDRESS=0xfff9976782d46cc05630d1f6ebab18b2324d6b14 # weth sepolia

#Mainnet
#PARENT_CHAIN_RPC=rpc eth
#PARENT_CHAIN_ID=1
#STAKE_TOKEN_ADDRESS=0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 # weth mainnet

ETHERSCAN_API_KEY=<api_key>
IS_DEV_DEPLOYMENT=false # custom var, need a modification in the deploy scripts

```

2. Create custom `contracts/scripts/config.ts` the most important stuff to change are the chainId and addresses (whitelist)

3. Apply to fixes:

```diff
// scripts/rollupCreation.ts : 129
const createRollupTx = await rollupCreator.createRollup(deployParams, {
   value: feeCost,
+  gasLimit: 7750000,
})
```

```diff
// scripts/local-deployment/deployCreatorAndCreateRollup.ts : 90
   const result = await createRollup(
     deployerWallet,
-    true,
+    process.env.IS_DEV_DEPLOYMENT !== 'false',
     contracts.rollupCreator.address,
     feeToken,
     feeTokenPricer,
```

4. Deploy it:

```sh
cd contracts
npx hardhat run scripts/local-deployment/deployCreatorAndCreateRollup.ts --network sepolia # or --network mainnet
```

In sepolia cost around 0.33 ETH.

5. Outputs: 

It will create `deploy.json` and `l2_chain_info.json` in the `contracts` folder.

Those file are needed to configure the sequencer, validators and other nodes.
