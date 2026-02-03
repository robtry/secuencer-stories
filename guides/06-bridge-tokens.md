
USDC: Circle USD Coin, USDT: Tether. Both are stablecoins pegged to the US dollar.
They can blacklist addresses, freeze funds, and reverse transactions if needed.


USDC Faucet: https://developer.bitaps.com/faucet 


1. Clone and build the sdk

```sh
git clone git@github.com:OffchainLabs/arbitrum-chain-sdk.git
cd arbitrum-chain-sdk
yarn install && yarn build
```

2. Modify `examples/create-token-bridge/index.ts`

```diff
--import { arbitrumSepolia } from 'views/chain';
++import { sepolia } from 'views/chain';
```
Then on line `41` change the `arbitrumSepolia` to `sepolia`

3. `cp .env.example .env` and fill the variables, correct rpcs and rollup addresses.

```ssh
cd examples/create-token-bridge
yarn dev 2&>1 | tee create-token-bridge.log
```

4. Verifiy contracts:

```sh
CREATOR=0x7edb2dfBeEf9417e0454A80c51EE0C034e45a570
INBOX=0xd25279f1471ebe381E34CC7D0d99EB492074baC5
cast call $CREATOR "inboxToL1Deployment(address)(address,address,address,address,address)" $INBOX --rpc-url http://localhost:8545
cast call $CREATOR "inboxToL2Deployment(address)(address,address,address,address,address,address,address,address,address)" $INBOX --rpc-url http://localhost:8545

```

# NOTE

Looks like the repo tried to verify on blockscout but sepolia is not supported yet. So verify code there is going yo be manual and tedious.

5. Fund some account on L1 with usdc.


```sh
# Approve standard gateway to spend usdc
cast send 0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238 \ # usdc l1 sepolia contract
    "approve(address,uint256)" \
    0xdD00F3F9D13Cf41FcB05773E169FF509a95b1eab \ # standard gateway
    50000000 \
    --rpc-url http://localhost:8545 \
    --private-key $OWNER_KEY

 # enconde _data: (maxSubmissionCost, callHookData empty)
DATA=$(cast abi-encode "f(uint256,bytes)" 100000000000000 0x) # abi encode

# Llamar al Router L1 con outboundTransferCustomRefund
cast send 0x5E213973b01c4849A932122E1085EB02066f9C6c \
"outboundTransferCustomRefund(address,address,address,uint256,uint256,uint256,bytes)" \
    0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238 \  # _token: USDC L1
    0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7 \  # _refundTo: owner (gas refund en L2)
    0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7 \  # _to: recipient en L2
    50000000 \                                       # _amount: 50 USDC
    1000000 \                                         # _maxGas: gas limit from retryable
    200000000 \                                      # _gasPriceBid: 0.2 gwei
    $DATA \                                          # _data: encoded
    --value 0.001ether \                             # ETH to pay retryable gas
    --rpc-url http://localhost:8545 \
    --private-key $OWNER_KEY
```


```sh
# wait ~15-20 min (delayed sequencer waits safe block)
# verify:
cast call 0x6b5d98BC3186B0fB210FAFe59ce44D36D91e6b9d \
    "balanceOf(address)(uint256)" 0x8b36F5A6F4e88c3A98598B92B28178b772C2d2a7 \
    --rpc-url http://localhost:8547

# Redeem retryable manualmente (ticket 7 days)
  cast send 0x000000000000000000000000000000000000006E \
    "redeem(bytes32)" \
    <TICKET_ID> \                                    # hash of TX ticket creation
    --gas-limit 2000000 \
    --rpc-url http://localhost:8547 \
    --private-key $OWNER_KEY
```


6. Bridge another token (USDT) its like any erc20 to l2.

Since my owner account has nonce > 0 I created a script. But its a simple ECR20 with usdt characteristics.

```sh
npx hardhat run scripts/deploy.ts --network capuchain
```

