---
title: "Paying transaction fee on behalf of someone else!"
date: 2022-04-13T12:30:05+05:30
draft: false
categories: ["cosmos", "blockchain"]
tags: ["blockchain", "feegrant", "cosmos",]
banner: "/images/authz-feegrant/cover.png"
authors:
  - arnab
---

Building a UX where a User feels comfortable in interacting with the blockchain is much needed. In this blog, we will be looking at how the `authz` and `feegrant` Cosmos SDK modules aim to build a similar User Experience.

The `authz` module grants and revokes permissions to accounts for performation certain transactions. This could be token trasnfer, staking. The generic authorization is where we can explicitly pass the TypeURL of the transaction, which is going to be our focus for this blog.

The `feegrant` module is responsible for granting and revoking Fee allowances. Here, we specify the limit that an account has to spend on fees. Once this limit exhausts, User won't be able to perform the authorized transactions.

### Pre-requisite for this article

The reader must be familiar with the concepts of Decentralised Identity (DID) and Self-Sovereign Identity (SSI), and have basic working knowledge of Cosmos SDK to understand the technical section of blog.

### The Problem

Say a User wants to create a DID on a Decentralized Registry. Since every transaction incur fees, they have to ensure their wallet is loaded with some Gas tokens. This becomes a friction for user, as they first have to trade fiat with crypto on a Centralized exchange, and then transfer them to a Non-Custodial Wallet like Metamask. This becomes a major hassle for dApps when it comes to onboarding their users.

Let's look at how we can solve this problem with [Hypersign Identity Network](https://github.com/hypersign-protocol/hid-node), which is built with Cosmos SDK

### The Approach

The below sequence diagrams lays a high level approach:

![Figure](/images/authz-feegrant/sequence-diagram.png)

We will now look at implementing the sequence with the help of Hypersign Identity Network's CLI. We will assume that Identity Provider's (IdP) account is already created.

IdP address: `hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj`

1. User's account is created

```
$ hid-noded keys add user

- name: user
  type: local
  address: hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"A4+zpdVic3RpnJvATZBuhFsH9lPGy/q0+uGRqSici3yN"}'
  mnemonic: ""
```

2. IdP authorizing User to perform DID Creation Transaction. We have to specify the TypeURL of the transaction being authorized.

```
hid-noded tx authz grant hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk generic --from hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj --msg-type "/hypersignprotocol.hidnode.ssi.MsgCreateDID"
```

We can list the authorizations being granted to User by IdP:

```
$ hid-noded q authz grants hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk

grants:
- authorization:
    '@type': /cosmos.authz.v1beta1.GenericAuthorization
    msg: /hypersignprotocol.hidnode.ssi.MsgCreateDID
  expiration: "2023-04-14T00:35:04Z"
pagination:
  next_key: null
  total: "0"
```

3. IdP Grants User a spend limit of `1000uhid` for Transaction Fee.

```
hid-noded tx feegrant grant hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk --spend-limit 1000uhid
```

We can check the allowance that was given to User:

```
$ hid-noded q feegrant grants hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk

allowances:
- allowance:
    '@type': /cosmos.feegrant.v1beta1.BasicAllowance
    expiration: null
    spend_limit:
    - amount: "1000"
      denom: uhid
  grantee: hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk
  granter: hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj
pagination:
  next_key: null
  total: "0"
```

4. User generates DID Document from the SDK, signs it with their pivate keys and sends it to Idp. Since, we are focused only on CLI, we will be executing the similar SDK process with its help.

```
hid-noded tx ssi create-did '{
"context": [
"https://www.w3.org/ns/did/v1",
"https://w3id.org/security/v1",
"https://schema.org"
],
"id": "did:hs:0f49341a-20ef-43d1-bc93-de30993e6c51",
"controller": ["did:hs:0f49341a-20ef-43d1-bc93-de30993e6c51"],
"verificationMethod": [
{
"id": "did:hs:0f49341a-20ef-43d1-bc93-de30993e6c51#zEYJrMxWigf9boyeJMTRN4Ern8DJMoCXaLK77pzQmxVjf",
"type": "Ed25519VerificationKey2020",
"controller": "did:hs:0f49341a-20ef-43d1-bc93-de30993e6c51",
"publicKeyMultibase": "zEYJrMxWigf9boyeJMTRN4Ern8DJMoCXaLK77pzQmxVjf"
}
],
"authentication": [
"did:hs:0f49341a-20ef-43d1-bc93-de30993e6c51#zEYJrMxWigf9boyeJMTRN4Ern8DJMoCXaLK77pzQmxVjf"
]
}' did:hs:0f49341a-20ef-43d1-bc93-de30993e6c51#zEYJrMxWigf9boyeJMTRN4Ern8DJMoCXaLK77pzQmxVjf --ver-key oVtY1xceDZQjkfwlbCEC2vgeADcxpgd27vtYasBhcM/JLR6PnPoD9jvjSJrMsMJwS7faPy5OlFCdj/kgLVZMEg== --from hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk --generate-only > tx.json
```

5. User submits the transaction with IdP acting as the fee Payer.

**Balances before the `authz` transaction:**

```
// IdP
$ hid-noded query bank balances hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj

balances:
- amount: "9000000000"
  denom: uhid
pagination:
  next_key: null
  total: "0"

// User
$ hid-noded query bank balances hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk

balances: []
pagination:
  next_key: null
  total: "0"

```

**Executing the `authz` transaction:**

```
hid-noded tx authz exec tx.json --from hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk --fee-account hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj --fees 50uhid
```

**Balances after the `authz` transaction:**

```
$ hid-noded query bank balances hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj

balances:
- amount: "8999999950"
  denom: uhid
pagination:
  next_key: null
  total: "0"


$ hid-noded query bank balances hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk

balances: []
pagination:
  next_key: null
  total: "0"
```
We can see that the fees `50uhid` has been deducted from IdP's account.The spend limit balance also went from `1000uhid` to `950uhid`:

```
hid-noded q feegrant grants hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk  

allowances:
- allowance:
    '@type': /cosmos.feegrant.v1beta1.BasicAllowance
    expiration: null
    spend_limit:
    - amount: "950"
      denom: uhid
  grantee: hid1enragujgzhex39tt34ljtqmwgljn0kr685f3nk
  granter: hid1qemjjsrsjwyuucv0zr370kj8k9dakvp9907zzj
pagination:
  next_key: null
  total: "0"
```

## Conclusion

In this blog, we saw how we can use `authz` and `feegrant` module to allow a User perform the DID creation transaction, while someone else handled the fee paying part. This also can be done for other DID related transactions.

## About Hypersign

The Hypersign Identity Network is a permissionless blockchain network to manage digital identity and access rights. It aims to empower humans to gain control of their data and access on the internet by providing scalable, interoperable and secure verifiable data registry (VDR) to implement use cases on Self Sovereign Identity (SSI) principles. The Hypersign Identity Network is built using Cosmos-SDK and is fully compatible with W3C DID specifications.

Github: https://github.com/hypersign-protocol/hid-node