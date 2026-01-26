# Running Sequencer, Batch Poster and Validators

1. build the node

```sh
docker build -t capu-nitro:v3.9.5 --target nitro-node .
```

2. create keystores for validators, batch posters and batch poster managers. owner not needed it just for contracts and control

`cast wallet import validator --private-key <priv_key> --keystore-dir ./data/keystore --unsafe-password "pass"`

Nitro will search for other fields that are not included in the command, so add them:

```json
{
  "address": "without 0x",
}
```

3. get a ETH RPC url. either from a provider or run your own eth node. And a beacon client url.

4. based on `nitro-testnode/scripts/config.ts` create config files for nodes. `sequencer-config.ts`, `validator-config.ts`, `batchposter-config.ts`, `batchpostermanager-config.ts`

```json
// sequencer_config.json
{
    "parent-chain": {
       "connection": {
         "url": "rcp"
       },
       "blob-client": {
         "beacon-url": "beacon client"
       }
    },
    "chain": {
      "id": 662201,
      "info-files": ["/config/l2_chain_info.json"]
    },
    "node": {
      "sequencer": true,
      "dangerous": {
        "no-sequencer-coordinator": true
      },
      "delayed-sequencer": {
        "enable": true
      },
      "batch-poster": {
        "enable": true,
        "max-delay": "30m",
        "parent-chain-wallet": {
          "pathname": "path-to-keystore-folder",
          "account": "0x",
          "password": "pass"
        }
      },
      "staker": {
        "enable": false
      }
    },
    "execution": {
      "sequencer": {
        "enable": true,
        "max-block-speed": "2500ms"
      }
    },
    "http": {
      "addr": "0.0.0.0",
      "port": 8547,
      "api": ["net", "web3", "eth", "txpool", "debug"]
    },
    "ws": {
      "addr": "0.0.0.0",
      "port": 8548
    }
 }
```

5. docker compose file to run rpc, sequencer and batch poster

```yaml
services:
  sequencer:
    image: capu-nitro:v3.9.5
    ports:
      - "8547:8547"    # RPC
      - "8548:8548"    # WebSocket
    volumes:
      - ./config:/config
      - ./keystores:/keystore
      - sequencer-data:/home/user/.capu
    command:
      - --conf.file=/config/sequencer_config.json
    restart: unless-stopped

volumes:
  sequencer-data:
```

the roles are defined in `sequencer_config.json` under `node` section

Since sequencer do not sing anything it just order l1 txs we dont need keysotre
Bacther poster need a keystore with enough eth to pay l1 gas fees


6. `docker compose up`


# Debug

## Basic RPC checks

```sh
RPC_L1="RPC-ETH"
RPC_L2="http://localhost:8547"

# get chain id
curl -s -X POST $RPC_L2 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'

# result in hexadecimal format
printf "%d\n" $(curl -s -X POST $RPC_L2 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' | jq -r '.result')

# get block number
curl -s -X POST $RPC_L2 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

printf "%d\n" $(curl -s -X POST $RPC_L2 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq -r '.result')

```

## check contracts

```sh
# Delayed messages en queue (Bridge)
printf "%d\n" $(cast call 0x47a3c5ea5de9aa34d6c22c18e59c927aaa49ee93 "delayedMessageCount()" --rpc-url $RPC_L1)

# Batches posteados (SequencerInbox)
cast call 0x1D83D4C0eC88a593E935AEeC52Da8E96332c986a "batchCount()" --rpc-url $RPC_L1
```

7. Bridge eth to l2.

```sh
cast send --rpc-url $RPC --private-key $OWNER_KEY \
    $INBOX "depositEth()" --value 0.3ether
```

Wait `max-delay` configured in the batch poster config file to see the eth in l2

8. Now we can send txs in l2

9. Set up a validator
