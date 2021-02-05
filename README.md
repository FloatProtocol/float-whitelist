# Democratic Launch

[Why a Democratic Launch](https://docs-float.gitbook.io/docs/democratic-launch/the-little-guy)

## Governance Sources
```sh
# Timestamp
timestamp=$(date +%s)
mkdir "$timestamp"

# Snapshot
curl https://hub.snapshot.page/api/voters > "$timestamp/snapshot.json"
cat "$timestamp/snapshot.json" | jq '.[].address' | egrep -o '0x[a-Z0-9]{40}' > "$timestamp/snapshot.txt"

# MakerDAO
curl 'https://api.thegraph.com/subgraphs/name/protofire/makerdao-governance' \
  -H 'authority: api.thegraph.com' \
  -H 'content-type: application/json' \
  -H 'accept: */*' \
  -H 'origin: https://thegraph.com' \
  -H 'referer: https://thegraph.com/' \
  --data-raw '{"query":"{\n  voterRegistries(first: 1000) {\n    coldAddress\n    hotAddress\n  }\n  addressVoters(first: 1000){\n    id\n  }\n}","variables":null}' \
  --compressed > "$timestamp/makerdao.json"

cat "$timestamp/makerdao.json" | jq '.data.addressVoters | .[].id' | egrep -o '0x[a-Z0-9]{40}' > "$timestamp/hot-and-cold-makerdao.txt"
cat "$timestamp/makerdao.json" | jq '.data.voterRegistries | .[].hotAddress' | egrep -o '0x[a-Z0-9]{40}' >> "$timestamp/hot-and-cold-makerdao.txt"
cat "$timestamp/makerdao.json" | jq '.data.voterRegistries | .[].coldAddress' | egrep -o '0x[a-Z0-9]{40}' >> "$timestamp/hot-and-cold-makerdao.txt"
cat "$timestamp/hot-and-cold-makerdao.txt" | sort | uniq | tee "$timestamp/makerdao.txt"

# Compound
curl https://api.compound.finance/api/v2/governance/accounts?page_size=2000&page_number=1&with_history=false&network=mainnet > "$timestamp/compound.json"
cat "$timestamp/compound.json" | jq '.accounts | .[] | select(.votes > "0") | .address' | egrep -o '0x[a-Z0-9]{40}' > "$timestamp/compound.txt"

# Moloch DAOs
cat dao/* >> "$timestamp/daos.txt"

# The Whitelist
cat "$timestamp/snapshot.txt" >> "$timestamp/all.txt"
cat "$timestamp/compound.txt" >> "$timestamp/all.txt"
cat "$timestamp/makerdao.txt" >> "$timestamp/all.txt"
cat "$timestamp/daos.txt" >> "$timestamp/all.txt"
cat "$timestamp/all.txt" | tr '[:upper:]' '[:lower:]' | sort | uniq | tee whitelist.txt | wc -l
# Convert to json
python -c "import json; addresses = [ a.strip() for a in open('whitelist.txt').readlines() ]; print(json.dumps(addresses));" > whitelist.json
```


### Snapshot

```sh
curl https://hub.snapshot.page/api/voters > "snapshot.json"
cat "snapshot.json" | jq '.[].address' | egrep -o '0x[a-Z0-9]{40}' > "snapshot.txt"
```

### MakerDAO

https://api.thegraph.com/subgraphs/name/protofire/makerdao-governance

```graphql
{
  voterRegistries(first: 1000) {
    coldAddress
    hotAddress
  }
  addressVoters(first: 1000){
    id
  }
}
```

```sh
curl 'https://api.thegraph.com/subgraphs/name/protofire/makerdao-governance' \
  -H 'authority: api.thegraph.com' \
  -H 'content-type: application/json' \
  -H 'accept: */*' \
  -H 'origin: https://thegraph.com' \
  -H 'referer: https://thegraph.com/' \
  --data-raw '{"query":"{\n  voterRegistries(first: 1000) {\n    coldAddress\n    hotAddress\n  }\n  addressVoters(first: 1000){\n    id\n  }\n}","variables":null}' \
  --compressed > "makerdao.json"

cat "makerdao.json" | jq '.data.addressVoters | .[].id' | egrep -o '0x[a-Z0-9]{40}' > "all-makerdao.txt"
cat "makerdao.json" | jq '.data.voterRegistries | .[].hotAddress' | egrep -o '0x[a-Z0-9]{40}' >> "all-makerdao.txt"
cat "makerdao.json" | jq '.data.voterRegistries | .[].coldAddress' | egrep -o '0x[a-Z0-9]{40}' >> "all-makerdao.txt"
cat "all-makerdao.txt" | sort | uniq | tee "makerdao.txt"
```


### Compound

```sh
# Assumes no. voters < 2000, can easily verify by checking number of addresses < 2000

curl https://api.compound.finance/api/v2/governance/accounts?page_size=2000&page_number=1&with_history=false&network=mainnet > "compound.json"
cat "compound.json" | jq '.accounts | .[] | select(.votes > "0") | .address' | egrep -o '0x[a-Z0-9]{40}' > "compound.txt"
```

### Top 10 DAOs by Bank Value
1. MetaCartel Ventures
1. The LAO
1. Moloch DAO
1. MetaCartel v2
1. Raid Guild War Chest
1. Machi X DAO
1. TrojanDAO
1. Rocket Bank LP Dao
1. yEarn DAO
1. Meta Gamma Delta (v2)