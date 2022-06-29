---
title: "Technical explanation of Two-Way-Authentication; Linking DID with Website Origin"
date: 2022-06-22T14:27:56+05:30
draft: false
categories: ["did", "blockchain", "ssi"] 
tags: ["did", "verifiableCredential", "vc", "web3", "login", "authentication" "ssi", "blockchain"]
banner: "/images/did-linkage-origin/phising-attack.png"
authors:
  [vishwas]
---

Two features we proudly announce on our website are;  **Two-Way Authentication**  and  **Independent Verification**. We will talk about the latter in some other blog but for this blog, let us just stick to the feature -  **Two-Way Authentication**. The question is what is this feature and how it can be technically implemented and proven that it MAY help to stop *phishing attacks*.

## Two-Way Authentication

Traditionally, when a user tries to log in to a website through a thrid party auth provider (or Identity Provider) like Google Login or Facebook Login, a user is asked to allow access to his/her data by showing him a popup. This is one way for the auth provider to take the user's consent to share his data with the service provider. That is how web2 authentication works.

But web3 authentication flow is different. In web3, the Identity Provider only comes into the picture at the time of issuing a credential, but once the credential has been issued, it has no idea, where the user is using it. Moreover, the verifier also does not connect with IdP to verify the digital signature of that credential.  It can verify the issuer's signature on its own merely by connecting with the blockchain. This authentication flow not only solves scalability problems for IdPs but also solve privacy problem for users. 

In web3, when the user tries to log in a website, the same "consent" is required for the verifier/website to access the user's data, but even before the user's shares data, the user should be able to verify the website/verifier's identity. This is called **Two-Way Authentication**, where not only websites verify user's credential but users too verifies the website before sharing their data. 

This Two-Way Authentication Process is based on the concept of mutual authentication, once the mutual authentication is established, a secure channel is created between both these parties to be able to communicate with each other securly. The process of mutual authentication happens with a tech called **DID Auth** where both parties verify ownership of DID of each other. 

Now for users to verify a website, they not only has to verify whether that website holds a DID or not but also have to verify who is that website? What is its origin? And how to prove that its origin is linked with that DID? 

I hope you have basic knowledge of DID and Verifiable Credentials, if not, I request you to read [this](https://labs.hypersign.id/posts/ssi-detail/) blog post before proceeding.

--- 

> **How to proof an entity controlling the [origin](https://identity.foundation/.well-known/resources/did-configuration/#Origin) and the [Controller](https://identity.foundation/.well-known/resources/did-configuration/#Controller) of the [DID](https://identity.foundation/.well-known/resources/did-configuration/#DID) to be the same entity?**

Any service wants to prove that they OWN a certain domain and a DID. How can it prove that? 

> **How can hypersign.id can proof that it owns an origin[hypersign.id](https://hypersign.id) and DID `did:hs:fb7f70e4-40a6-4e8b-a1ac-51221b2bbb5f`?**

So here are two things that need to be proven

- The service which owns origin hypersign.id also owns a DID. 
- Looking at DID, one should be able to identify association of an origin



## Concept of `/.well-known/` URI

Before we go into those, let's just first understand the concept of `/.well-known/` URI. It will help us to find answers to those questions. 

A well-known URI is a URI where the path component begins with `/.well-known` path. Example [https://example.com/**.well-known**/oauth-authorization-server](https://example.com/.well-known/oauth-authorization-server) is a well know URI. Your server needs to have a path called `/.well-known/` where you store certain meta-data which can be used by other systems.  Read [RFC 8615](https://datatracker.ietf.org/doc/html/rfc8615) for more. 

In short:

> Defining well-known locations usurps the origin's control over its own URI space!

## Linking DID with an Origin

Now lets us understand how a well-known URI can be used to prove that a website owns a particular DID. 

> How Well-known URI helps an Origin to prove ownership of a DID?

### Via [DID Configuration](https://identity.foundation/.well-known/resources/did-configuration)

An origin can form a well-known configuration file (`did-configuration.json`) which will contain `DomainLinkageCredential` - a verifiable credential issued to itself, and store it at its `/.well-known/` location or directory. 

**For example:** 

If hypersign.id has to prove the ownership of a DID, then it will issue a `DomainLinkageCredential` credential which has two properties `id` and `origin` and store that credential in a `did-configuration.json` file in the `/.well-known/` location.

```
 "credentialSubject": {
        "id": "did:hs:fb7f70e4-40a6-4e8b-a1ac-51221b2bbb5f",
        "origin": "https://identity.foundation"
      },
```
**Sample well-known URI would be**

[https://hypersign.id/.well-known/did-configuration.json](https://hypersign.id/.well-known/did-configuration.json)

**Sample domain linkage credential:**

```json
{
  "@context": "https://identity.foundation/.well-known/did-configuration/v1",
  "linked_dids": [
    {
      "@context": [
        "https://www.w3.org/2018/credentials/v1",
        "https://identity.foundation/.well-known/did-configuration/v1"
      ],
      "issuer": "did:hs:fb7f70e4-40a6-4e8b-a1ac-51221b2bbb5f",
      "issuanceDate": "2020-12-04T14:08:28-06:00",
      "expirationDate": "2025-12-04T14:08:28-06:00",
      "type": [
        "VerifiableCredential",
        "DomainLinkageCredential"
      ],
      "credentialSubject": {
        "id": "did:hs:fb7f70e4-40a6-4e8b-a1ac-51221b2bbb5f",
        "origin": "https://hypersign.id"
      },
      "proof": {
        "type": "Ed25519Signature2018",
        "created": "2020-12-04T20:08:28.540Z",
        "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..D0eDhglCMEjxDV9f_SNxsuU-r3ZB9GR4vaM9TYbyV7yzs1WfdUyYO8rFZdedHbwQafYy8YOpJ1iJlkSmB4JaDQ",
        "proofPurpose": "assertionMethod",
        "verificationMethod": "did:hs:fb7f70e4-40a6-4e8b-a1ac-51221b2bbb5f#z6MkoTHsgNNrby8JzCNQ1iRLyW5QQ6R8Xuu6AA8igGrMVPUM"
      }
    },
    "eyJhbGciOiJFZERTQSJ9.eyJleHAiOjE3NjQ4Nzg5MDgsImlzcyI6ImRpZDprZXk6ejZNa29USHNnTk5yYnk4SnpDTlExaVJMeVc1UVE2UjhYdXU2QUE4aWdHck1WUFVNIiwibmJmIjoxNjA3MTEyNTA4LCJzdWIiOiJkaWQ6a2V5Ono2TWtvVEhzZ05OcmJ5OEp6Q05RMWlSTHlXNVFRNlI4WHV1NkFBOGlnR3JNVlBVTSIsInZjIjp7IkBjb250ZXh0IjpbImh0dHBzOi8vd3d3LnczLm9yZy8yMDE4L2NyZWRlbnRpYWxzL3YxIiwiaHR0cHM6Ly9pZGVudGl0eS5mb3VuZGF0aW9uLy53ZWxsLWtub3duL2RpZC1jb25maWd1cmF0aW9uL3YxIl0sImNyZWRlbnRpYWxTdWJqZWN0Ijp7ImlkIjoiZGlkOmtleTp6Nk1rb1RIc2dOTnJieThKekNOUTFpUkx5VzVRUTZSOFh1dTZBQThpZ0dyTVZQVU0iLCJvcmlnaW4iOiJpZGVudGl0eS5mb3VuZGF0aW9uIn0sImV4cGlyYXRpb25EYXRlIjoiMjAyNS0xMi0wNFQxNDowODoyOC0wNjowMCIsImlzc3VhbmNlRGF0ZSI6IjIwMjAtMTItMDRUMTQ6MDg6MjgtMDY6MDAiLCJpc3N1ZXIiOiJkaWQ6a2V5Ono2TWtvVEhzZ05OcmJ5OEp6Q05RMWlSTHlXNVFRNlI4WHV1NkFBOGlnR3JNVlBVTSIsInR5cGUiOlsiVmVyaWZpYWJsZUNyZWRlbnRpYWwiLCJEb21haW5MaW5rYWdlQ3JlZGVudGlhbCJdfX0.6ovgQ-T_rmYueviySqXhzMzgqJMAizOGUKAObQr2iikoRNsb8DHfna4rh1puwWqYwgT3QJVpzdO_xZARAYM9Dw"
  ]
}
```

## So how does this prove the ownership of a DID by an origin?

Well, access to `/.well-known/` folder can only be with the website owner, he can only be the one who can store the `did-configuration.json` file in that folder.

## Verifying the DomainLinkageCredential 

Now that a well-known URI is formed and a `DomainLinkageCredential` is stored, a verifier (user agent in this case) can verify the fact that the website owns a DID.

> How can someone verify that the DomainLinkageCredential that the website owner stored in that folder is correct?

Steps to verify the Domain Linkage Credential

1. The `credentialSubject.origin` property **MUST** be present, and its value **MUST** match the origin the resource was requested from. 
2. The implementer **MUST** perform [DID resolution](https://www.w3.org/TR/did-core/#resolution) on the DID specified in the [Issuer](https://identity.foundation/.well-known/resources/did-configuration/#Issuer) of the [Domain Linkage Credential](https://identity.foundation/.well-known/resources/did-configuration/#DomainLinkageCredential) to obtain the associated DID document.
3. Using the retrieved DID document, the implementer **MUST** validate the signature of the [Domain Linkage Credential](https://identity.foundation/.well-known/resources/did-configuration/#DomainLinkageCredential) against key material referenced in the [assertionMethod section](https://www.w3.org/TR/did-core/#assertion) of the DID document.

If [Domain Linkage Credential](https://identity.foundation/.well-known/resources/did-configuration/#DomainLinkageCredential) verification is successfull, a [Verifier](https://identity.foundation/.well-known/resources/did-configuration/#Verifier) **SHOULD** consider the entity controlling the [origin](https://identity.foundation/.well-known/resources/did-configuration/#Origin) and the [Controller](https://identity.foundation/.well-known/resources/did-configuration/#Controller) of the [DID](https://identity.foundation/.well-known/resources/did-configuration/#DID) to be the same entity.

Pay attention to the first point very carefully. The well-known URI to which the verifier is accessing must have the same origin as mentioned in the   `credentialSubject.origin` property of the `DomainLinkageCredential`. 


![img](/images/did-linkage-origin/did-origin-link.png)


## Linking an Origin with a DID

Now that we understood how we can link a DID with an Origin,  the next question is,  how can a domain be linked with a DID?  Or in other words, how a DID configuration can be located given a DID?

### Via [Linked Domain Service Endpoint](https://identity.foundation/.well-known/resources/did-configuration/#linked-domain-service-endpoint)

As per spec: 

- There must exist a DID Document mechanism for expressing origins where **DID Configurations MAY be located** that prove a **bidirectional linkage** between the DID and the origins it claims to possess control over.
- LinkedDomains [Service Endpoint](https://w3c.github.io/did-core/#service-endpoints), which allows a DID controller to specify a list of origins over which **they assert control**

Origins asserted within the LinkedDomains endpoint descriptor can then be subsequently crawled by verifying parties to locate and verify any DID Configuration resources that may exist. 

Example DIDDoc with linkedDomain:

```json
{
  "@context": ["https://www.w3.org/ns/did/v1","https://identity.foundation/.well-known/did-configuration/v1"],
  "id": "did:example:123",
  "verificationMethod": [{
    "id": "did:example:123#_Qq0UL2Fq651Q0Fjd6TvnYE-faHiOpRlPVQcY_-tA4A",
    "type": "JsonWebKey2020",
    "controller": "did:example:123",
    "publicKeyJwk": {
      "kty": "OKP",
      "crv": "Ed25519",
      "x": "VCpo2LMLhn6iWku8MKvSLg2ZAoC-nlOyPVQaO3FxVeQ"
    }
  }],
  "service": [
    {
      "id":"did:example:123#foo",
      "type": "LinkedDomains",
      "serviceEndpoint": {
        "origins": ["https://foo.example.com", "https://identity.foundation"]
      }
    }
  ]
}
```

I hope this concept helped you to understand how phishing attacks may be mitigated because the user-agent should be able to authenticate the website before even asking the user to share data. We at Hypersign are determined to bring products which help users to gain control of their data and protect their privacy. In the next blog, we take a deep dive into "Independent Verification". 

Till then Happy Learning!






