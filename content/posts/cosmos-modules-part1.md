---
title: "Cosmos Modules, Part I : staking, distribution and slashing"
date: 2022-04-13T12:30:05+05:30
draft: false
categories: ["cosmos", "blockchain"]
tags: ["blockchain", "feegrant", "cosmos",]
banner: "/images/cosmos-modules-1/cover.png"
authors:
  - arnab
---

# Staking

The `x/staking` module handles operations related to staking. Let's undestand Staking modules with an account being made as a validator and then perform operations related to modules.

### Creating a validator
```
hid-noded tx staking create-validator --amount=5000000uhid --from=node1 --pubkey=$(hid-noded tendermint show-validator --home=$HOME/.hid-node) --moniker="node1" --chain-id="hidnode" --commission-rate="0.1" --commission-max-rate="0.2" --commission-max-change-rate="0.05" --min-self-delegation="5000" --keyring-backend=test --home=$HOME/.hid-node --yes
```
Here, we have user with account name `node1`. The user becomes the validator by staking `5000000uhid` in the chain.

We can the `node1` being created as a validator

```
$ hid-noded query staking validators

pagination:
  next_key: null
  total: "0"
validators:
- commission:
    commission_rates:
      max_change_rate: "0.010000000000000000"
      max_rate: "0.200000000000000000"
      rate: "0.100000000000000000"
    update_time: "2022-04-25T02:27:31.631898868Z"
  consensus_pubkey:
    '@type': /cosmos.crypto.ed25519.PubKey
    key: tfcs7VoyvRwrCZ/kzgM95jc+4xuiLBUzFQgxqVZzSsA=
  delegator_shares: "500000000.000000000000000000"
  description:
    details: ""
    identity: ""
    moniker: node1
    security_contact: ""
    website: ""
  jailed: false
  min_self_delegation: "1"
  operator_address: hidvaloper19jcfvf940pr7q3yelxa5xcd0kla68praamclx5
  status: BOND_STATUS_BONDED
  tokens: "500000000"
  unbonding_height: "0"
  unbonding_time: "1970-01-01T00:00:00Z"
```

### Editing details of a validator

```
hid-noded tx staking edit-validator --details "A sample validator" --website "example.com" --from node1
```

Here, we updated the `details` and `website` field of validator. Upon querying the list of validators, we see these values being updated:

```
$ hid-noded query staking validators

pagination:
  next_key: null
  total: "0"
validators:
- commission:
    commission_rates:
      max_change_rate: "0.010000000000000000"
      max_rate: "0.200000000000000000"
      rate: "0.100000000000000000"
    update_time: "2022-04-25T02:27:31.631898868Z"
  consensus_pubkey:
    '@type': /cosmos.crypto.ed25519.PubKey
    key: tfcs7VoyvRwrCZ/kzgM95jc+4xuiLBUzFQgxqVZzSsA=
  delegator_shares: "500000000.000000000000000000"
  description:
    details: A sample validator
    identity: ""
    moniker: node1
    security_contact: ""
    website: example.com
  jailed: false
  min_self_delegation: "1"
  operator_address: hidvaloper19jcfvf940pr7q3yelxa5xcd0kla68praamclx5
  status: BOND_STATUS_BONDED
  tokens: "500000000"
  unbonding_height: "0"
  unbonding_time: "1970-01-01T00:00:00Z"
``` 

### Delegating tokens to validator by a user

Say we have a user, `alice`, who has balance of `100000uhid` and wants to delegate `1000uhid` to the validator `node1`.

```
hid-noded tx staking delegate hidvaloper19jcfvf940pr7q3yelxa5xcd0kla68praamclx5 1000uhid --from alice
```
Here `hidvaloper19jcfvf940pr7q3yelxa5xcd0kla68praamclx5` is the validator address of `node1`

Upon query the list of validators, we can see that `node1` now has `500001000.000000000000000000` shares staked, where 1000 shares are being delegated to it by `alice`:

```
$ hid-noded query staking validators
pagination:
  next_key: null
  total: "0"
validators:
- commission:
    commission_rates:
      max_change_rate: "0.010000000000000000"
      max_rate: "0.200000000000000000"
      rate: "0.100000000000000000"
    update_time: "2022-04-25T02:27:31.631898868Z"
  consensus_pubkey:
    '@type': /cosmos.crypto.ed25519.PubKey
    key: tfcs7VoyvRwrCZ/kzgM95jc+4xuiLBUzFQgxqVZzSsA=
  delegator_shares: "500001000.000000000000000000"
  description:
    details: A sample validator
    identity: ""
    moniker: node1
    security_contact: ""
    website: example.com
  jailed: false
  min_self_delegation: "1"
  operator_address: hidvaloper19jcfvf940pr7q3yelxa5xcd0kla68praamclx5
  status: BOND_STATUS_BONDED
  tokens: "500001000"
  unbonding_height: "0"
  unbonding_time: "1970-01-01T00:00:00Z"
```

### Unbonding tokens staked by user

`alice` has `1000uhid` stake in the system. However, they now want to unstake or unbond `500uhid`.

```
hid-noded tx staking unbond hidvaloper19jcfvf940pr7q3yelxa5xcd0kla68praamclx5 500uhid --from alice
```
The `500uhid` won't be credited to `alice` immediately, but after a specified unbond time that defined in the `genesis.json` file:

```json
"staking": {
      "params": {
        "unbonding_time": "1814400s",
        "max_validators": 100,
        "max_entries": 7,
        "historical_entries": 10000,
        "bond_denom": "uhid"
      },
      "last_total_power": "0",
      "last_validator_powers": [],
      "validators": [],
      "delegations": [],
      "unbonding_delegations": [],
      "redelegations": [],
      "exported": false
  }
```
The value of `unbonding_time` is 1814400 seconds or 21 days, after the requested amount will be creidted to `alice`'s wallet. The unbonded tokens become part of the unbonded tokens pool. We can query the pool by following command:

```
$ hid-noded query staking pool

bonded_tokens: "500000500"
not_bonded_tokens: "500"
```
The `500uhid` requested by `alice` is present in the `not_bonded_tokens` section. Till the unbonding time period is over, `alice` will continue to have the stake in HID-Network

### Redelgate from one validator to another

`alice` can also redelegate some tokens (say `500uhid`) from `node1` to some other validator

```
hid-noded tx staking redelegate <node1-validator-address> <other-validator-address> 500uhid --from alice
```

Unlike unbonding, this will have instantaneously once the block is committed.

# Distribution

The `x/distribution` module handles the reward distribution in the chain.

- Collected rewards are pooled together in a **community pool**
  - They are divided out to validators and delegators
  - Validators charge a commission rate to delegators, that takes some cut from delegator’s reward
  - While withdrawing, one must withdraw all the rewards they are entitled to
  - A full withdrawal of rewards must occur when bonding, re-delegate or unbonding of token happens
 
- Once a validator receives their portion of the reward, it distributes them to its delegators before charging a commission rate (defined by `commission_rate` param in `genesis.json`)
- Understanding reward distribution with the help of an example:
  - Lets assume we have 10 validators with equal weightage to stake.
  - The Block Reward is `1000 HID` per block. Hence, each validator will receive `100 HID`.
  - Shifting our focus to a particular validator now. This validator just got `100 HID`. Let's say, it has 20% tokens the participants of that validator delegate self-delegated and the reset, and the commission that validator charges to its delegators is 1%.
    - Commission: (`100 HID`) * (80% part of delegators) * (1% commission) = `0.8 HID`
    - Validator received: (20% of `100 HID` reward) + Commission = `20.8 HID`
    - Delegators: (80% of `100 HID` reward) - Commission = `79.2 HID`
      - `79.2 HID` is then shared among the delegators proportional to their stake

# Slashing

- The `x/slashing` module handles punishing the validators for not being adhered to rules. The reasons could be:

  - Being for offline for a said number of blocks
  - Double Signing

- Following are how validators are being punished:

  - Some of their stake is burned
  - Removing their ability to vote on blocks for a particular time period

- A `ValidatorSigningInfo` record is present that contains information partaining to validator's liveliness.

- Tombstone Caps
  - It's allows a validator to be slashed only once for double signing
  - This is quite expensive
  - They help in mitigating any economic impact of such infraction.
  - This concept is only applied to Double signing, because of the delay between the infraction committed and the evidence being uneartherd. (This is also one of the reasons for `unbonding_period` to exist)

- Once the `Jail Period` is over, the validator can send an unjail transaction, and join the validator set. While if they are in Tombstone state (infinite jail period), they are essentially kicked out of the validator set. The delegator tokens remain bonded
