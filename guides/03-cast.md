
# create wallets

```sh
cast wallet new-mnemonic -w 24 --json
cast balance <addr> --rpc-url <YOUR_RPC_URL>
```

# create keystores

```sh
mkdir -p ./data/keystore
cast wallet import <name> \
    --private-key <privatekey> \
    --keystore-dir ./data/keystore \
    --unsafe-password "pass"

```

# send eths

```sh
cast send --rpc-url $RPC --private-key $PRIV_KEY \
    0xDESTINATION \
    --value 0.5ether


# Call function depositEth() Inbox contract
cast send --rpc-url $RPC_L1 --private-key $KEY $INBOX "depositEth()" --value 0.3ether

```

# check balance

```sh
cast balance <ADDRESS> --rpc-url $RPC_L1 --ether
```

# call functions

```sh
# Delayed messages en queue (Bridge)
cast call 0x47a3c5ea5de9aa34d6c22c18e59c927aaa49ee93 "delayedMessageCount()" --rpc-url $RPC_L1
```

