---
title: Page365 Shipping API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby

toc_footers:
  - <a href='https://github.com/lord/slate'>Powered by Slate</a>
  - <a href='mailto:peace@page365.net'>Contact</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to Page365 Shipping API! You can use our API to register webhook endpoint, confirm shipment weight, and update shipment status. Afterwards, Page365 will be responsible for create the shipment via webhook.

We have language bindings in Shell, and Ruby. You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# How API works

Here is the API call, webhook flow that is designed for Page365 Shipping API:

1. [Register webhook](#register-webhook) - first, register your company name along with webhook url that Page365 Shipping API will push data to, after confirmed, `secret_key` will be send to you.
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
Staging | `TBD`
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

<aside class="notice">
Optional: webhook normally expect response :ok, but can also receive <a href="#update-shipment-details">shipment object</a> too.
</aside>
<aside class="warning">
Warning: any server error code (5xx), will trigger retry logic, 3 times in total. For any client error (4xx), system will not retry at all.
</aside>

# POST API

## Register Webhook

```ruby
HTTParty.post('https://<ENDPOINT>/couriers', body: {
  name: 'SuperFastShipping',
  url: 'https://www.super-fast-shipping.com/webhook'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --data '{"name":"SuperFastShipping", "url":"https://www.super-fast-shipping.com/webhook"}' \
     https://<ENDPOINT>/couriers
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to register webhook url to Page365 system.

### HTTP Request

`POST https://<ENDPOINT>/couriers`

### Request Parameters

Parameter | Description
--------- | -----------
name | Name of the shipping company
url | Webhook url that Page365 will be send shipment information to

<aside class="notice">
The register might take a few days, due to verification process before actual save into system is done by men.
</aside>

## Update Shipment Details

```ruby
HTTParty.post('https://<ENDPOINT>/shipments/1034234', basic_auth: { username: secret_key }, body: {
  status: 'shipping',
  tracking_code: "ABQZ1234KL",
  weight: 0.05,
  reference_id: '003-11245',
  note: 'dimention size: 3 * 3 * 3'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
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

`POST https://<ENDPOINT>/shipments/<ID>`

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
name | Name of the shipping company

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

`GET https://<ENDPOINT>/account/<ID>`

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
