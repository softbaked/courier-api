---
title: Page365 Courier API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby

toc_footers:
  - <a href='https://www.page365.net/'>Developed by Page365</a>
  - <a href='mailto:peace@page365.net'>Technical Contact</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to Page365 Courier API. This standard allows you to become a supported courier, delivering parcels for over 300,000 merchants on the Page365 platform.

The purpose of the Courier API is to expedit the importing/exporting of ecommerce transaction data. Courier is still responsible for collecting shipment fees from the merchant, for handling COD transfers, and for maintaining its own relationship with the merchant. The Courier API will refer to each merchant with a consistent Account ID to ease in this mapping, as well as provide basic information about each merchant such as bank account and pickup address.

You can use the Courier API to register your webhook endpoint, at which Page365 will send new shipment orders. Subsequently, you can make requests according to this document to confirm shipment weight, and update shipment status for your users.

Courier API is implemented in standard HTTP REST protocol, and we do not yet have language libraries available. You can view examples for how to invoke the requests in Shell and Ruby in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# How it works

1. [Register webhook](#register-webhook) - first, make this request to set or update the webhook url that Page365 Courier API will push data to, after confirmed, `secret_key` will be send to you.
2. [Wait for webhook](#shipment-created) - then wait for webhook that will be send, after shipment being create. You will get shipment information from the webhook, such as, id, sender address, and receiver address.
3. [Update tracking code](#update-shipment-tracking-code) - then you will send tracking code that will be on parcel label back to us.
4. Customer bring parcel to your service store
5. [Update shipment weight](#confirm-shipment-weight) - call api update weight, after parcel being weighted.
6. [Keep update shipment status](#update-shipment-status) - customer will be able to track shipment status via the update on this api.

In case of any error on webhook, you can [query list of shipments](#get-shipment-list), or [specific shipment](#get-specific-shipment) by GET API anytime.

# Environment endpoint

for <ENDPOINT> value in each api.

Environnment | URL
------------ | ---
Staging | `https://courier.staging365.net/`
Production | `https://courier.page365.net/`

# Webhook

## Shipment Created

> Example of returns JSON structured, after shipment being created:

```json
{
  "id": 1034234,
  "status": "new",
  "sender": {
    "id": 1123,
    "name": "Miss Pimploen",
    "phone": "086-123-4456",
    "email": "pimploen@page365.net",
    "address": {
      "text": "555 Soi Sukhumvit 63 Khwaeng Khlong Tan Nuea, Khet Watthana, Krung Thep Maha Nakhon 10110",
      "postcode": 10110
    }
  },
  "receiver": {
    "name": "Mr. Drumb",
    "phone": "023456678",
    "email": "d@page365.net",
    "address": {
      "text": "Softbaked Co., Ltd. (HQ) 90 Fifty Fifth Plaza 4/F Unit 4L2 Thong Lo 2, Sukhumvit 55 Rd., Khlong Tan Nuea, Watthana, Bangkok Thailand 10110",
      "postcode": 10110
    },
    "bank_account": {
      "bank": "tmb",
      "number": "756-1-22231-9",
      "name": "Mr. Y"
    }
  },
  "parcel": {
    "is_pickup": 0,
    "is_cod": 1,
    "price": 150.00
  }
}
```

This is the webhook detail that Page365 will send to webhook url, after any shipment being created.

### Validating Payloads

Every webhook will have **SHA1** signature included in the request's `X-Hub-Signature` header, preceded with `sha1=`. You don't have to validate the payload, but you should.

To validate the payload:
- 1. Generate a **SHA1** signature using the payload and your `secret_key`.
- 2. Compare your signature to the signature in the `X-Hub-Signature` header (everything after `sha1=`). If the signatures match, the payload is genuine.

<aside class="notice">
Please note that we generate the signature using an escaped unicode version of the payload, with lowercase hex digits. If you just calculate against the decoded bytes, you will end up with a different signature. For example, the string <i>เทสๆ</i> should be escaped to <i>\u0E40\u0E17\u0E2A\u0E46</i>.
</aside>

### Webhook Body

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | false | Shipment id
status | false | Current status of shipment (Default: `new`)
sender | false | (Account object) Sender detail
receiver | false | (Account object) Receiver detail
parcel | false | (Parcel object) Parcel detail

#### Account object

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | true | Account id (Mandatory on sender detail)
name | false | Account name
phone | true | Account phone, free text
email | true | Account email, allow null
address | false | (Address object) Address detail
bank_account | true | (Bank account object) Bank account detail

#### Address object

Parameter | Allow Null | Description
--------- | ---------- | -----------
text | false | Address detail on text, free text
postcode | false | Postcode

#### Bank account object

Parameter | Allow Null | Description
--------- | ---------- | -----------
bank | false | Name of bank company
number | false | Bank number
name | false | Bank owner's name

#### Parcel object

Parameter | Allow Null | Description
--------- | ---------- | -----------
is_pickup | false | Is this shipment require pickup or not? (Default: 0)
is_cod | false | Is this shipment cod? (Default: 0)
price | true | Shipment price, in case of cod

<aside class="notice">
Optional: webhook normally expect response :ok, but can also receive <a href="#update-shipment-details">shipment object</a> too.
</aside>
<aside class="warning">
Warning: any server error code (5xx), will trigger retry logic, 3 times in total. For any client error (4xx), system will not retry at all.
</aside>

# POST API

## Register Webhook

```ruby
HTTParty.post('https://<ENDPOINT>/providers', body: {
  name: 'SuperFastShipping',
  url: 'https://www.super-fast-shipping.com/webhook'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --request POST \
     -u secret_key \
     --data '{"name":"SuperFastShipping", "url":"https://www.super-fast-shipping.com/webhook"}' \
     https://<ENDPOINT>/providers
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to register new courier and webhook url on Page365 system.

### HTTP Request

`POST https://<ENDPOINT>/providers`

### Request Parameters

Parameter | Description
--------- | -----------
name | Name of the courier company
url | Webhook url that Page365 will be send shipment information to

<aside class="notice">
The register might take a few days, due to verification process before actual save into system is done by men.
</aside>

# PATCH API

## Update Webhook

```ruby
HTTParty.patch('https://<ENDPOINT>/providers', basic_auth: { username: secret_key }, body: {
  name: 'SuperFastShipping',
  url: 'https://www.super-fast-shipping.com/new_webhook'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --request PATCH \
     -u secret_key \
     --data '{"name":"SuperFastShipping", "url":"https://www.super-fast-shipping.com/mew_webhook"}' \
     https://<ENDPOINT>/providers
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to patch updated webhook url on Page365 system.

### HTTP Request

`PATCH https://<ENDPOINT>/providers`

### Request Parameters

Parameter | Description
--------- | -----------
name | Name of the courier company
url | Webhook url that Page365 will be send shipment information to

# PUT API

## Update Shipment Details

```ruby
HTTParty.put('https://<ENDPOINT>/shipments/1034234', basic_auth: { username: secret_key }, body: {
  status: 'shipping',
  tracking_code: "ABQZ1234KL",
  weight: 0.05,
  reference_id: '003-11245',
  note: 'dimention size: 3 * 3 * 3'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --request PUT \
     -u secret_key \
     --data '{"status":"shipping", "tracking_code":"ABQZ1234KL", "weight":0.05, "reference_id": "003-11245", "note": "dimention size: 3 * 3 * 3"}' \
     https://<ENDPOINT>/shipments/1034234
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to update shipment status, weight, ref id, or note.

### HTTP Request

`PUT https://<ENDPOINT>/shipments/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of shipment to update

### Request Parameters

Parameter | Mandatory | Description
--------- | --------- | -----------
status | No | Current status of shipment: <ul><li>`new`: waiting for send / pickup</li><li>`shipping`: in process of shipping</li><li>`completed`: the delivery completed</li><li>`cancelled`: any error that occur and make the shipping incomplete</li><li>`returned`: returned parcel to sender completed</li></ul>
tracking_code | No | Tracking code that will be printed on the parcel
weight | No | Weight in kilogram unit (0.05 = 50 gram)
reference_id | No | Any reference id that will be needed on other end
note | No | Any free text for given shipment
error_message | No | Reason for cancelled status

# GET API

## Get Shipment List

```ruby
HTTParty.get('https://<ENDPOINT>/shipments', basic_auth: { username: secret_key }, body: {
  name: 'SuperFastShipping'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --request GET \
     -u secret_key \
     --data '{"name":"SuperFastShipping"}' \
     https://<ENDPOINT>/shipments
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 13532,
    "status": "shipping"
  },
  {
    "id": 13565,
    "status": "new"
  }
]
```

This endpoint allow you to get list of shipments that register for your company name.

### HTTP Request

`GET https://<ENDPOINT>/shipments`

### Request Parameters

Parameter | Description
--------- | -----------
name | Name of the courier company

### Response Body

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | false | Shipment id
status | false | Current status of shipment (Default: `new`)

## Get Specific Shipment

```ruby
HTTParty.get('https://<ENDPOINT>/shipments/1124232', basic_auth: { username: secret_key })
```

```shell
curl --header "Content-Type: application/json" \
     --request GET \
     -u secret_key \
     https://<ENDPOINT>/shipments/1124232
```

> The above command returns JSON structured like this:

```json
{
  "id": 1124232,
  "status": "completed",
  "sender": {
    "name": "Miss Pimploen",
    "phone": "086-123-4456",
    "email": "pimploen@page365.net",
    "address": {
      "text": "555 Soi Sukhumvit 63 Khwaeng Khlong Tan Nuea, Khet Watthana, Krung Thep Maha Nakhon 10110",
      "postcode": 10110
    }
  },
  "receiver": {
    "name": "Mr. Drumb",
    "phone": "023456678",
    "email": "d@page365.net",
    "address": {
      "text": "Softbaked Co., Ltd. (HQ) 90 Fifty Fifth Plaza 4/F Unit 4L2 Thong Lo 2, Sukhumvit 55 Rd., Khlong Tan Nuea, Watthana, Bangkok Thailand 10110",
      "postcode": 10110
    }
  },
  "parcel": {
    "is_pickup": 0,
    "is_cod": 0
}
```

This endpoint allow you to get details of specific shipment.

### HTTP Request

`GET https://<ENDPOINT>/shipments/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of shipment to update

### Response Body

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | false | Shipment id
status | false | Current status of shipment (Default: `new`)
sender | false | (Account object) Sender detail
receiver | false | (Account object) Receiver detail
parcel | false | (Parcel object) Parcel detail

#### Account object

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | true | Account id (Mandatory on sender detail)
name | false | Account name
phone | true | Account phone, free text
email | true | Account email, allow null
address | false | (Address object) Address detail

#### Address object

Parameter | Allow Null | Description
--------- | ---------- | -----------
text | false | Address detail on text, free text
postcode | false | Postcode

#### Parcel object

Parameter | Allow Null | Description
--------- | ---------- | -----------
is_pickup | false | Is this shipment require pickup or not? (Default: 0)
is_cod | false | Is this shipment cod? (Default: 0)
price | true | Shipment price, in case of cod

## Get Account

```ruby
HTTParty.get('https://<ENDPOINT>/accounts/<ID>', basic_auth: { username: secret_key })
```

```shell
curl --header "Content-Type: application/json" \
     --request GET \
     -u secret_key \
     https://<ENDPOINT>/accounts/<ID>
```

> The above command returns JSON structured like this:

```json
{
  "id": 1123,
  "name": "Miss Pimploen",
  "phone": "086-123-4456",
  "email": "pimploen@page365.net",
  "address": {
    "text": "555 Soi Sukhumvit 63 Khwaeng Khlong Tan Nuea, Khet Watthana, Krung Thep Maha Nakhon 10110",
    "postcode": 10110
  },
  "bank_account": {
    "bank": "kbank",
    "number": "742-0-17141-1",
    "name": "Mr. X"
  }
}
```

This endpoint allow you to get detail of specific account, for example, sender detail. Benefit on contact and billing to sender.

### HTTP Request

`GET https://<ENDPOINT>/accounts/<ID>`

### Request Parameters

Parameter | Description
--------- | -----------
id | Account id

### Response Body

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | true | Account id
name | false | Account name
phone | true | Account phone, free text
email | true | Account email, allow null
address | false | (Address object) Address detail
bank_account | true | (Bank account object) Bank account detail

#### Address object

Parameter | Allow Null | Description
--------- | ---------- | -----------
text | false | Address detail on text, free text
postcode | false | Postcode

#### Bank account object

Parameter | Allow Null | Description
--------- | ---------- | -----------
bank | false | Name of bank company
number | false | Bank number
name | false | Bank owner's name
