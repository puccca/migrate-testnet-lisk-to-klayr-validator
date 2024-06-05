### 10 steps to migrate testnet lisk v4 validator to testnet klayr validator

Initial requirements:
- Validated on lisk testnet network before May 2024
- At least 1000 testnet klayr in the validator's wallet
- [Testnet klayr node installed](https://github.com/puccca/install-testnet-klayr-core)

## 1. Change "allowedMethods": [] to "allowedMethods": ["*"] in config
```shell
nano .klayr/klayr-core/config/config.json
```

## 2. Restart klayr core
```shell
pm2 stop all
pm2 delete klayr-core
pm2 start core-start.json
pm2 logs
```

## 3. Create validator's keys
```shell
klayr-core keys:create --chainid 1 --output config/keys.json --add-legacy
```

## 4. Import keys
```shell
klayr-core keys:import --file-path config/keys.json
```

## 5. Create hashonion for 6+ years
```shell
ADDRESS="<YOUR_TESTNET_KLAYR_ADDRESS>"
HASHCOUNT=200000
JSONFILE="$HOME/hash_onion.json"
klayr-core hash-onion --count "$HASHCOUNT" --output "$JSONFILE"
echo $( cat "$JSONFILE" | jq --arg x $ADDRESS '. + {address: $x}' ) > "$JSONFILE"
klayr-core endpoint:invoke random_setHashOnion --file "$JSONFILE"
klayr-core endpoint:invoke random_getHashOnionSeeds "{\"address\":\"$ADDRESS\"}"
klayr-core endpoint:invoke random_getHashOnionUsage "{\"address\":\"$ADDRESS\"}"
rm -f "$JSONFILE"
```

## 6. Check if node is still syncing - syncing=false means it's synced
```shell
klayr-core system node-info --pretty
```
*If node is synced, proceed to step 7*

## 7. Set status values to 0-0-0 only if your generator didn't generate any block in the current core version release!
```shell
klayr-core endpoint:invoke generator_setStatus '{"address":"<YOUR_TESTNET_KLAYR_ADDRESS>","height":0,"maxHeightGenerated":0,"maxHeightPrevoted":0}' --pretty
```

## 8. Enable generator
```shell
klayr-core generator:enable <YOUR_TESTNET_KLAYR_ADDRESS> --use-status-value
```

## 9. Create self-vote transaction of 1000 testnet KLY
```shell
klayr-core transaction:create pos stake 100000000 --pretty --json --params='{"stakes":[{"validatorAddress":"<YOUR_TESTNET_KLAYR_ADDRESS>","amount":100000000000}]}' --key-derivation-path=legacy
```

## 10. Send self-vote transaction
```shell
klayr-core transaction:send <TRANSACTION_ID_FROM_STEP_9>
```

## Useful commands
*Check validator's infos*
```shell
klayr-core endpoint:invoke pos_getValidator '{ "address":"<YOUR_TESTNET_KLAYR_ADDRESS>"}' --pretty
```

*Check generator's imported keys*
```shell
klayr-core endpoint:invoke generator_getAllKeys --pretty
```

*Check hash onion*
```shell
klayr-core endpoint:invoke random_getHashOnionSeeds '{"address":"<YOUR_TESTNET_KLAYR_ADDRESS>"}' --pretty
```

*Check hash onion usage*
```shell
klayr-core endpoint:invoke random_getHashOnionUsage '{"address":"<YOUR_TESTNET_KLAYR_ADDRESS>"}' --pretty
```

*Check generator status (the magic numbers)*
```shell
klayr-core generator:status --pretty
```

*Restart the node & enable generator*
```shell
pm2 delete all
pm2 start core-start.json
pm2 logs
klayr-core generator:status --pretty
klayr-core generator:enable <YOUR_TESTNET_KLAYR_ADDRESS> --use-status-value
```
