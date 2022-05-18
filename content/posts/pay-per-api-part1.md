---
title: "Pay per API Call Protocol - Part 1"
date: 2022-05-18T18:25:30+05:30
draft: false
categories: ["payment", "blockchain",]
tags: ["payment", "cryptocurrencies", "api", "payment-gateway", "tokens"]
banner: "/images/pay-per-api-part1/HID-layerd-architecutre-Page-9.drawio.png"
authors:
  [vishwas]
---

> NOTE: This is just my imagination without doing any proper research either on the problem statement or on the protocol. I request you to read this blog keeping that in mind. This is NOT full proof solutinon. 

Applications share data with each others via APIs. Sometimes, these APIs need to protected behind paywall by the API providers. This is to ensure APIs consumers do not misuse APIs as well as to incure cost of the API services. Currently the way APIs providers implement this paywall is vis montly or yearly subscriptions; APIs consumers either needs to pre-subscribe for different plans like the way Twitter APIs charges (**pre-subscribe-plan**) or monthly bills are generated based on usage and sent to API consumers (**billed-on-usage-plan**), like the way AWS or other infa providers charges. 

### *For API Providers*

Both of these methods of subscripitions needs books or record keeping of all transactions which have been done or pending and process is required for generating bills and handling settlements. All these maintainances become hectic and costly and needs a lot of efforts, specially for small and medium API service providers. There is also a trust issue for provider in case of **billed-on-usage-plan** plan where he is the mercy of consumer to pay for the already cosumed services. 

### *For API Consumer*

The pre-subscription model is not very effective since API consumer needs to pay upfront even before using the services. It may happen that API consumer do need monthly subscription but just new to make a few API call to the provider. But they will end up spending for the entire month's cost. 

## **Pay Per API Call**

Imagine the API consumer pay for every API call they make to the provider. The API provider verifies the payment and serves data only when payment is done or atleast promise is made for the payment in near future. In either case,  there is no need to maintain books or manage settlement or wait for payment cycle. Payment settlemnt is done with every API request the consumer make. 

We propose a http header called, `X-Payment-Auth`  with http request for the promise that *"API consumer will pay to the provider for this service"* - just like a cheque! 

Its an authorization grant provided by the consumer to deduct cost of the API call by the provider. This grant is cryptographically signed by the cosumer and sent to the provider. 

However there is one problem with this protocol. Usually, payment needs time to get settled down, say for 30s or so, but frequency of API consumption will be much higher like 1k/s in that case there will clearly be latency in the API request fullfillment becuase the providers will have to wait until the payment is done before sending the response.  This will have huge impact on the business. 

The obvious solution to this problem would be to do payment synchronously. And keep pushing the authorization grants in some form of *queue* and then settle those payment in a seperate thread and let the provider serve the request without blocking the call. The issue would what if the queue fails? What if the payment fails? 

### **Protocol**

Before we jump into all those edge case, let's see how the protocol might look like. 

Say there is a service to provide list of on-going events in a city Bangalore. 

```
Path:
/api/v1/events

Method: 
GET

Response: 
[
 {
   "eventname": "Summer Event",
   "startDate": "2022-05-06T05:40:00.000Z",
   "endDate": "2023-05-18T05:40:00.000Z",
   "venue": "ABC Hotel"
 }
]
```

Let's put this API behind the Pay-Per-API paywall. To do that, the consumer will have to send the 	`X-Payment-Auth` header in this http request;

```
Path:
/api/v1/events

Method: 
GET

Headers:
{
  "X-Payment-Auth": "<signed transaction message>"
}

Response: 
[
 {
   "eventname": "Summer Event",
   "startDate": "2022-05-06T05:40:00.000Z",
   "endDate": "2023-05-18T05:40:00.000Z",
   "venue": "ABC Hotel"
 }
]
```


![HID-layerd-architecutre-Page-9.drawio.png](/images/pay-per-api-part1/HID-layerd-architecutre-Page-9.drawio.png)

Upon receiving the request, the API consumer will parse the header and keep the signed transaction message into a queue, say **unconfirmed-payment-queue**, for the seperate process, say **payment-settlement-service**, to process and severs the response with event list data. 

The **payment-settlement-service** picks ups the **signed transaction message** from the queue and proceeds with payment settlements process. 

> **NOTE**: The idea here is to merge  the service request with payment so as to save cost of extra call just for payment. Thats the reason payment is being sent in the header of the same request. 

#### **Payment Authorization Header Formation**

How does the `X-Payment-Auth` formed? 

Assume that the API consumer already know about the price of the API call, say $1,  and it also knows the account Id or wallet address (in case of cryptocurrency transfer), it can generate the tranasaction message, signs it and sends the signature as `X-Payment-Auth` header in the request.

**Sample Tx Message:**

```js
tx_message = {
  "api": "/api/v1/events",
  "amount": "1",
  "denomination": "$",
  "from": "<API consumer wallet address / account Id>",
  "to": "<API provider wallet address / accountId>",
  "timestamp": "5232131321",
  "nounce": "28"
}
```

**Signature:** 

```js
{signature} = web3.eth.sign(tx_message)
```

This signature can be sent as `X-Payment-Auth` header.

#### **Payment Authorization Header Verification** 

API consumer can parse/decode the `X-Payment-Auth` header and does the following verifications :

1. Verifies if the `api` field is correct?
2. Verifies if right `amount` is being sent for that API?
3. Verifies if `denomination` is accecpted by the provider or not?
4. Verifiers if `to` address belongs to API Provider or not? 

Once these verification dones, the API provider can push this tx into the *Unconfirmed-payment-queue*. 

#### **Payment Settlement Procedure**

The payment settlement procedures may have two folds:

Firstly, the **payment-settlement-service** picks up each tx from **unconfirmed-payment-queue**, segregates these txs based on which bank or blockchain they belongs to and then finally submits them into their respective payment systems. 

Secondly, the tx goes through various verfications steps (like nonce, signature) and consensus, depending on the payment network, and finally gets settle down. This whole procedure is out of scope of this docuement and we will let the payment network decide how they want to settle the payment. We only care about the acknolegment and status of that payment. 

### **This System Is Far From Perfect**

This system seems to be far from perfect because it does not answer the following questions:

1. What if the *unconfirmed-payment-queue* goes **down**? 
2. What if the *payment-settlement-service* goes **down**?
3. What if the payment settlement **fails**? 
4. What about the **availbility** or *unconfirmed-payment-queue* and *payment-settlement-service*?
5. Who **owns** the unconfirmed-payment-queue? 
6. Who will **runs** the payment-settlement service? 
7. How can the service provider **trust** on the payment provider? 
8. What if the consumer pays in **different currencies** / tokens? 

---

I will leave it here with food for thoughts. In the next blog we will try to redesign this system and find answers to some of these questions. 
