---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href="https://paymenthighway.fi" target="_blank">Payment Highway</a> API reference
  - <a href="https://www.solinor.fi" target="_blank">Solinor Oy</a> 2015

includes:
  - sandbox
  - form
  - payment
  - rcodes
  - datatypes

search: true
---

# Introduction

This is the API reference and example documentation for <a href="https://paymenthighway.fi/index-en.html" target="_blank">Payment Highway</a>. The easy and enjoyable card payment solution for mCommerce and eCommerce.

Payment Highway API consists of two parts:

* [Form API](#form-api) - for displaying a secure card information input form
* [Payment API](#payment-api) - for charging and refunding a card and for reporting

## Usage

### Make a Payment

1. Show the form with Form API [`POST /form/view/pay_with_card`](#payment)<br />
—> returns an `sph-transaction-id` as a GET parameter to the given `success-url`
2. Commit the payment with Payment API [`POST /transaction/<sph-transaction-id>/commit`](#commit-form-payment)<br />
—> returns a result in JSON formatting

### Store a Card

1. Show the form with Form API [`POST /form/view/add_card`](#add-card)<br />
—> returns an `sph-tokenization-id` as a GET parameter to the given `success-url`
2. Get the token with Payment API [`GET /tokenization/<sph-tokenization-id>`](#tokenization)<br />
—> returns a `card_token` and card information in JSON formatting

### Pay with a Stored Card

1. Initialize a transaction with Payment API [`POST /transaction`](#charge-and-refund)<br />
—> returns a transaction `id` in JSON formatting
2. Charge the card with Payment API [`POST /transaction/<id>/debit`](#charge-and-refund-transactions)<br />
—> returns a result in JSON formatting