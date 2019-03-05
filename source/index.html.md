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

# POST API

## Register Webhook

```ruby
HTTParty.post('https://page365.net/shippings', body: {
  name: 'SuperFastShipping',
  url: 'https://www.super-fast-shipping.com/webhook'
}.to_json)
```

```shell
will be available soon
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to register webhook url to Page365 system.

### HTTP Request

`POST https://page365.net/shippings`

### Request Parameters

Parameter | Description
--------- | -----------
name | Name of the shipping company.
url | Webhook url that Page365 will be send shipment information to.

<aside class="notice">
The register might take a few days, due to verification process before actual save into system is done by men.
</aside>

## Confirm Shipment Weight

```ruby
HTTParty.post('https://page365.net/shippings/1034234', body: {
  weight: 0.05
}.to_json)
```

```shell
will be available soon
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to update shipment weight.

### HTTP Request

`POST https://page365.net/shippings/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of shipment to update
weight | Weight in kilogram unit (0.05 = 50 gram)

## Update Shipment Statue

```ruby
HTTParty.post('https://page365.net/shippings/1034234', body: {
  status: 'shipping'
}.to_json)
```

```shell
will be available soon
```

> The above command returns JSON structured like this:

```json
{
  "success": 1
}
```

This endpoint allow you to update shipment status.

### HTTP Request

`POST https://page365.net/shippings/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of shipment to update
status | Current status of shipment: `shipping`, `completed`, `cancelled`
