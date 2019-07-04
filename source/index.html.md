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

1. [Register webhook](#register-webhook) - first, register your company name along with webhook url that Page365 Shipping API will push data to.
2. [Wait for webhook](#shipment-created) - then wait for webhook that will be send, after shipment being create. You will get shipment information from the webhook, such as, id, sender address, and receiver address.
3. [Update tracking code](#update-shipment-tracking-code) - then you will send tracking code that will be on parcel label back to us.
4. Customer bring parcel to your service store
5. [Update shipment weight](#confirm-shipment-weight) - call api update weight, after parcel being weighted.
6. [Keep update shipment status](#update-shipment-status) - customer will be able to track shipment status via the update on this api.

In case of any error on webhook, you can [query list of shipments](#get-shipment-list), or [specific shipment](#get-specific-shipment) by GET API anytime.

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
sender | false | (User object) Sender detail
receiver | false | (User object) Receiver detail
parcel | false | (Parcel object) Parcel detail

#### User object

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | true | User id (Mandatory on sender detail)
name | false | User name
phone | true | User phone, free text
email | true | User email, allow null
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

# POST API

## Register Webhook

```ruby
HTTParty.post('https://{{ENDPOINT}}/couriers', body: {
  name: 'SuperFastShipping',
  url: 'https://www.super-fast-shipping.com/webhook'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --data '{"name":"SuperFastShipping", "url":"https://www.super-fast-shipping.com/webhook"}' \
     https://{{ENDPOINT}}/couriers
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to register webhook url to Page365 system.

### HTTP Request

`POST https://{{ENDPOINT}}/couriers`

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
HTTParty.post('https://{{ENDPOINT}}/shipments/1034234', body: {
  status: 'shipping',
  tracking_code: "ABQZ1234KL",
  weight: 0.05,
  reference_id: '003-11245',
  note: 'dimention size: 3 * 3 * 3'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --data '{"status":"shipping", "tracking_code":"ABQZ1234KL", "weight":0.05, "reference_id": "003-11245", "note": "dimention size: 3 * 3 * 3"}' \
     https://{{ENDPOINT}}/shipments/1034234
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to update shipment status, weight, ref id, or note.

### HTTP Request

`POST https://{{ENDPOINT}}/shipments/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of shipment to update

### Request Parameters

Parameter | Mandatory | Description
--------- | --------- | -----------
status | No | Current status of shipment: <ul><li>`shipping`: in process of shipping</li><li>`completed`: the delivery completed</li><li>`cancelled`: any error that occur and make the shipping incomplete</li></ul>
tracking_code | No | Tracking code that will be printed on the parcel
weight | No | Weight in kilogram unit (0.05 = 50 gram)
reference_id | No | Any reference id that will be needed on other end
note | No | Any free text for given shipment

# GET API

## Get Shipment List

```ruby
HTTParty.get('https://{{ENDPOINT}}/shipments', body: {
  name: 'SuperFastShipping'
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --request GET \
     --data '{"name":"SuperFastShipping"}' \
     https://{{ENDPOINT}}/shipments
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

`GET https://{{ENDPOINT}}/shipments`

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
HTTParty.get('https://{{ENDPOINT}}/shipments/1124232')
```

```shell
curl https://{{ENDPOINT}}/shipments/1124232
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

`GET https://{{ENDPOINT}}/shipments/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of shipment to update

### Response Body

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | false | Shipment id
status | false | Current status of shipment (Default: `new`)
sender | false | (User object) Sender detail
receiver | false | (User object) Receiver detail
parcel | false | (Parcel object) Parcel detail

#### User object

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | true | User id (Mandatory on sender detail)
name | false | User name
phone | true | User phone, free text
email | true | User email, allow null
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

## Get User

```ruby
HTTParty.get('https://{{ENDPOINT}}/users', body: {
  id: 1123
}.to_json)
```

```shell
curl --header "Content-Type: application/json" \
     --request GET \
     --data '{"id":1123}' \
     https://{{ENDPOINT}}/users
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
  }
}
```

This endpoint allow you to get detail of specific user, for example, sender detail. Benefit on contact and billing to sender.

### HTTP Request

`GET https://{{ENDPOINT}}/users`

### Request Parameters

Parameter | Description
--------- | -----------
id | User id

### Response Body

Parameter | Allow Null | Description
--------- | ---------- | -----------
id | true | User id
name | false | User name
phone | true | User phone, free text
email | true | User email, allow null
address | false | (Address object) Address detail

#### Address object

Parameter | Allow Null | Description
--------- | ---------- | -----------
text | false | Address detail on text, free text
postcode | false | Postcode

