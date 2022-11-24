# Nibiru
Nibiru Node Installation Instructions

## General information

### Official documentation:
```
https://docs.nibiru.fi/
```
### System requirements:
2 Cpu, 
4 Ram, 
80Gb ssd,
Ubuntu 20.04

# Installing the Nibiru Node
### 1. System update, environment installation:
```
sudo  apt update && apt upgrade -y
```
```
sudo apt install git build-essential ufw curl jq snapd -y
```
### 2. Installing the node:
```
wget -q -O nibiru.sh https://api.nodes.guru/nibiru.sh && chmod +x nibiru.sh && sudo /bin/bash nibiru.sh
```
```
source $HOME/.bash_profile
```
### 3. Throwing addrbook:
```
wget -O $HOME/.nibid/config/addrbook.json https://api.nodes.guru/nibiru_addrbook.json
```
### 4. Restart the service:
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable nibid
```
```
sudo systemctl restart nibid
```
### 5. Checking the synchronization (you need to wait for False):
Synchronization can be accelerated, see section "Speed Up Sync"
```
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```
### 6. Create a wallet:
Be sure to save the wallet data and do not forget about the seed phrase below, it will come in handy to restore the wallet.
```
nibid keys add wallet
```
### 7. In the discord from the faucet channel, we request coins by resetting the wallet address with the $request command
```
https://discord.com/channels/947911971515293759/984840062871175219
```
### 8. Checking the balance:
We replace "your wallet address" in the command with the wallet address received when creating the wallet.
```
nibid q bank balances "your wallet address"
```
### 9. Create a validator:
Copy everything at once and paste it into the terminal.
```
nibid tx staking create-validator \
--amount=1000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker="$NIBIRU_NODENAME" \
--chain-id=nibiru-testnet-1 \
--commission-rate="0.01" \
--commission-max-rate="0.10" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1000000" \
--fees=50000unibi \
--from=wallet \
-y
```
### 10. Find out valoper address:
```
 nibid keys show wallet --bech val -a
```
### 11. Delegate tokens to increase staking:
Don't forget to replace YOUR_VALOPER_ADDRESS with the address obtained in the previous step.
```
nibid tx staking delegate YOUR_VALOPER_ADDRESS 10000000unibi --from wallet --chain-id nibiru-testnet-1 --fees 5000unibi
```
### 12. The success of the delegation, the status and rating of the validator can be viewed on the website:
```
https://nibiru.explorers.guru/
```
# Speed Up Sync:
```
sudo systemctl stop nibid
```
```
cp $HOME/.nibid/data/priv_validator_state.json  $HOME/.nibid/priv_validator_state.json.backup
```
```
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
```
```
SNAP_RPC="https://nibiru-testnet.nodejumper.io:443"
```
```
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq - r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq - r .result.block_id.hash)
```
```
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
peers="b32bb87364a52df3efcbe9eacc178c96b35c823a@nibiru- testnet.nodejumper.io:27656"
```
```
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|'  $HOME/.nibid/config/config.toml
```
```
sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|"  $HOME/.nibid/config/config.toml
```
```
mv $HOME/.nibid/priv_validator_state.json.backup  $HOME/.nibid/data/priv_validator_state.json
```
```
sudo systemctl restart nibid
```
# Additional commands
### View logs
```
journalctl -u nibid -f
```
### Node restart:
```
systemctl restart nibid
```
### Check node performance:
```
curl localhost:26657/status
```
### List of active validators:
```
nibid query staking validators --limit 2000 -o json | jq -r '.validators[] | select(.status=="BOND_STATUS_BONDED") | [.operator_address, .status, (.tokens|tonumber / pow(10; 6)), .description.moniker] | @csv' | column -t -s"," | sort -k3 -n -r
```
### List of inactive validators:
```
nibid query staking validators --limit 2000 -o json | jq -r '.validators[] | select(.status=="BOND_STATUS_UNBONDED") | [.operator_address, .status, (.tokens|tonumber / pow(10; 6)), .description.moniker] | @csv' | column -t -s"," | sort -k3 -n -r
```
### Restore wallet:
Requires moniker and seed phrase
```
nibid keys add wallet --recover
```
### Delete node:
```
systemctl stop nibid
```
```
systemctl disable nibid
```
```
rm -rf $(which nibid) ~/.nibid ~/nibiru
```
