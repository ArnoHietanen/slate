# Form API

Payment Highway Form API allows merchants to tokenize payment cards and create payments using an HTML form interface.

## Request and Response format

### Requests
The “sph”-prefixed form fields and the [request signature](#request-signature-calculation) should be calculated server side and set in the html form as hidden fields.

<aside class="notice">
Request method must be POST and the character set must be UTF-8.
</aside>

### Responses

Responses are delivered to URLs given in the request and signed by using the same key as in the request. Response parameters are added as GET parameters to the URL.

## Authentication

### Request signature calculation

Signature is calculated from the request parameters with HMAC-SHA256 algorithm using one of the merchant secret keys. The signature value contains “SPH1”, the key ID and the calculated authentication hash as a hexadecimal string separated with spaces “ ” (0x20).

The authentication hash value is calculated from the authentication string using the chosen merchant secret key. The authentication string is formed from the request method, URI and the request parameters beginning with “sph-”-prefix. Values are trimmed and the key-value pairs are concatenated in alphabetical order by the key name. The parameter keys must be in lowercase. Each key and value is separated with a colon (“:”) and the different parameters are separated with a new line (“\n”) at the end of each value.

### Response signature calculation

Response parameters are formatted into a single value and HMAC-SHA256 signature is calculated with the same secret key as in the request. The signature value contains “SPH1”, the key ID and the calculated authentication hash as a hexadecimal string separated with spaces “ ” (0x20).

The authentication hash value is calculated from the authentication string using the chosen merchant secret key. The authentication string is formed from the request method, newline and response parameters beginning with “sph-”-prefix. Values are trimmed and the key-value pairs are concatenated in alphabetical order (by the key name). The parameter keys must be in lowercase. Each key and value is separated with a colon (“:”) and the different parameters are separated with a new line (“\n”) at the end of each value.

## Add Card

Adding a new card stores the payment card information to Payment Highway and returns a tokenization id that can be used to fetch a card token for payments.

<img src="/images/sph-add-new-card.png">

### HTTP Request

`POST /form/view/add_card`

Parameter | Data type | M/O | Description
--------- | --------- | --- | -----------
sph-account | AN | M | Account identifier
sph-merchant | AN | M | Account merchant identifier
sph-request-id | UUID4 | M | Request identifier
sph-timestamp | TIMESTAMP | M | Request timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. "2025-09-18T10:32:59Z"
sph-success-url | URL | M | Success URL the user is redirected to on success
sph-failure-url | URL | M | Failure URL the user is redirected to on failure
sph-cancel-url | URL | M | Cancel URL the user is redirected to on cancel
language | A | O | Two letter language code (ISO 639-1). Supported languages are DE, EN, ES, FI, FR, RU, SV. Defaults to browser language.
signature | ANS |  M | Message signature in the format key-id:authentication-string

#### Success Response for Add Card

On a successful operation the user is redirected to the given success URL `sph-success-url`.

Parameter | Data type | Description
--------- | --------- | -----------
sph-account | AN | Account identifier from request
sph-merchant | AN | Account merchant identifier from request
sph-request-id | UUID4 | Request identifier from request
sph-tokenization-id | AN | Generated sph-tokenization-id
sph-timestamp | AN | Response timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. 2025-09-18T10:33:49Z
sph-success | AN | Static text “OK”
signature | ANS | Message signature

#### Failure Response for Add Card

On failure the user is redirected to the given failure URL `sph-failure-url`.

Parameter | Data type | Description
--------- | --------- | -----------
sph-account | AN | Account identifier from request
sph-merchant | AN | Account merchant identifier from request
sph-request-id | UUID4 | Request identifier from request
sph-timestamp | AN | Response timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. 2025-09-18T10:33:49Z
sph-failure | AN | Failure reason, one of: <ul><li>"UNAUTHORIZED"</li><li>"INVALID"</li><li>"FAILURE"</li></ul>
signature | ANS | Message signature

#### Cancel Response for Add Card

If the user cancels the operation they are redirected to the given cancel URL `sph-cancel-url`.

Parameter | Data type | Description
--------- | --------- | -----------
sph-account | AN | Account identifier from request
sph-merchant | AN | Account merchant identifier from request
sph-request-id | UUID4 | Request identifier from request
sph-timestamp | AN | Response timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. 2025-09-18T10:33:49Z
sph-cancel | AN | Cancel reason "CANCEL"
signature | ANS | Message signature

## Payment

The payment card form is shown in the Payment Highway. The response to the `success-url` contains a `sph-transaction-id` for committing the transaction through the [Payment API](#payment-api).

<img src="/images/sph-pay-with-card.png">

### HTTP Request

`/form/view/pay_with_card`

## Payment & Add Card

This method combines a payment and adding a new card to allow getting the card token after a successful payment with a single request.

<img src="/images/sph-add-and-pay-with-card.png">

### HTTP Request

`/form/view/add_and_pay_with_card`

> POST the form

```shell
curl -i --data-urlencode '
sph-account=sampleAccount001
sph-merchant=sampleMerchant001
sph-order=1000123A
sph-request-id=f47ac10b-58cc-4372-a567-0e02b2c3d479
sph-amount=990
sph-currency=EUR
sph-timestamp=2014-09-18T10:32:59Z
sph-success-url=https://merchant.example.com/payment/success
sph-failure-url=https://merchant.example.com/payment/failure
sph-cancel-url=https://merchant.example.com/payment/cancel
language=fi
description=Example payment of 10 balloons á 0,99EUR
signature= SPH1 account001-key001 5f8c9c9311cc83de8c240e2255ce86a44f209671be466170e4cfe4a2430911de' \
	https://v1-hub-staging.sph-test-solinor.com/form/view/add_and_pay_with_card
```

> Response

```html
HTTP/1.1 401 Unauthorized
Content-Type: text/plain; charset=UTF-8
Date: Tue, 10 Feb 2015 09:26:49 GMT
Content-Length: 0
Connection: keep-alive
```

Parameter | Data type | M/O | Description
--------- | --------- | ------ | --- | -----------
sph-account | AN | M | Account identifier
sph-merchant | AN | M | Account merchant identifier
sph-order | AN | M | Merchant defined order identifier. Should be unique per transaction.
sph-request-id | UUID4 | M | Request identifier
sph-amount | N | M | Amount in the lowest currency unit. E.g. 99,00 € = 9900
sph-currency | A | M | Currency code "EUR"
sph-timestamp | AN | M | Request timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. "2025-09-18T10:32:59Z"
sph-success-url | URL | M | Success URL user is redirected to on success
sph-failure-url | URL | M | Failure URL user is redirected to on failure
sph-cancel-url | URL | M | Cancel URL user is redirected to on cancel
language | A | O | Language code (ISO 639-1)(FI/EN/SV)
description | ANS | O | The order description shown to the user
signature | ANS | M | Message signature in the format key-id:authentication-string

### Success Response for Payment

On a successful operation the user is redirected to the given success URL `sph-success-url`.

Parameter | Data type | Description
--------- | --------- | -----------
sph-account | AN | Account identifier
sph-merchant | AN | Account merchant identifier
sph-order | AN | Merchant defined order identifier
sph-request-id | UUID4 | Request identifier
sph-amount | N | Amount in the lowest currency unit. E.g. 99,00 € = 9900
sph-currency | A | Currency code "EUR"
sph-transaction-id | AN | Payment transaction identifier
sph-timestamp | AN | Request timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. "2025-09-18T10:32:59Z"
sph-success | AN | Static text “OK”
signature | ANS | Message signature

### Failure Response for Payment

On failure the user is redirected to the given failure URL `sph-failure-url`.

Parameter | Data type | Description
--------- | --------- | -----------
sph-account | AN | Account identifier from request
sph-merchant | AN | Account merchant identifier from request
sph-order | AN | Merchant defined order identifier
sph-request-id | UUID4 | Request identifier from request
sph-timestamp | AN | Response timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. 2025-09-18T10:33:49Z
sph-failure | AN | Failure reason, one of: <ul><li>"UNAUTHORIZED"</li><li>"INVALID"</li><li>"FAILURE"</li></ul>
signature | ANS | Message signature

### Cancel Response for Payment

If the user cancels the operation they are redirected to the given cancel URL `sph-cancel-url`.

Parameter | Data type | Description
--------- | --------- | -----------
sph-account | AN | Account identifier from request
sph-merchant | AN | Account merchant identifier from request
sph-order | AN | Merchant defined order identifier
sph-request-id | UUID4 | Request identifier from request
sph-timestamp | AN | Response timestamp in ISO 8601 <br>combined date and time in UTC.<br>E.g. 2025-09-18T10:33:49Z
sph-cancel | AN | Cancel reason "CANCEL"
signature | ANS | Message signature