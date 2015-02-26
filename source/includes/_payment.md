# Payment API

## HTTP Interface

The system consists of different API commands, or resources accessible via HTTPS protocol using the defined mandatory headers for authentication, UTF-8 JSON body for customizing the POST requests.

HTTP response code is always 200 for successful HTTP calls. Anything else indicates a system or communication level failure.

Mandatory headers for both requests and responses are: Content-Type, Content-Length, Signature and all the other ones defined in the Message Headers section. Valid values for “Content-Type” and “Content-Length” are "application/json; charset=utf-8" and the correct body byte length using UTF-8 encoding. The value for header “Signature” is explained in the following chapter.

### Headers

In addition to the standard “Content-Type” and “Content-Length” headers, the following custom headers are used.

All the HTTP requests must contain the following headers:
1. “Signature”: ”” = The signature header used for authentication and message
consistency, see the following section.
2. “SPH-Account”: ”example_account”= (High level) account name associated with the
authentication key.
3. “SPH-Merchant”: ”example_merchant” = The account’s sub-account, which provides
optional merchant level customization.
4. “SPH-Timestamp”: “yyyy-MM-dd'T'HH:mm:ss'Z'” = Contains the client’s request time
in UTC format. Server will check the timestamp does not differ more than five (5)
minutes from the correct global UTC time.
5. “SPH-Request-Id”: ”12ade018-c562-40bc-a4e6-7f63c69fd90a” = Unique one-time-
use Request ID in UUID4 format.

Optionally the HTTP request *may* contain the following headers:
1. “Schema-Version”: “yyyyMMdd” = The version date of the JSON response schema.
Defaults to “20141215”.

All the HTTP responses contain the following headers:
1. “Signature”: ”” see the following section.
2. “SPH-Response-Id”: ”03c15388-bebc-4872-b3f5-faed0ca65ff6” = Unique one-time-
use Response ID in UUID4 format.
3. “SPH-Timestamp”: “yyyy-MM-dd'T'HH:mm:ss'Z'” which contains the servers
response time in UTC format. When the client receives the response, the timestamp must be checked and it must not differ more than five (5) minutes from the correct global UTC time.
4. “SPH-Request-Id”: ”12ade018-c562-40bc-a4e6-7f63c69fd90a” = Same as the request UUID4.

### Signatures

Request authentication and the response signatures are calculated by applying the HMAC- SHA256 method (RFC 2104 - Keyed-Hashing for Message Authentication, http://www.ietf.org/rfc/rfc2104.txt) to the defined concatenated string using the secret key provided to the account user.

A signature is transmitted Hex encoded in the ”Signature” HTTP header with the signature value prefixed with the strings “SPH1” and “secretKeyID”, where the secretKeyID is the ID of the key used in the HMAC calculation. The strings are separated with blank spaces (0x20) thus the header would look like: “Signature: SPH1 secretKeyId signature”

The concatenated string consists of the following fields separated with a new line (“\n”):

> Signature HTTP content

```shell
POST
/transaction/859cefdf-41fa-453a-a6a5-beff35e2f3b8/debit
sph-account:example_account
sph-merchant:example_merchant
sph-request-id:12ade018-c562-40bc-a4e6-7f63c69fd90a
sph-timestamp:2014-09-18T14:09:25Z
{”some”:”body”{”json”:1}}
```

* HTTP method (e.g. “POST” or “GET”)
* HTTP URI without server URL (e.g. ““/transaction/859cefdf-41fa-453a-a6a5-
beff35e2f3b8/debit”).
* All the SPH- headers’ trimmed key value pairs concatenated in alphabetic order (by
the header name). The header keys must be in lowercase. Each header’s key and value are separated with a colon (“:”) and the different headers are separated with a new line (“\n”).
* HTTP body if exists, empty string if not.

<aside class="notice">
An empty body “” (e.g. in GET requests), is still included in the concatenated string, resulting in one new line character after the headers.
</aside>

The client must check the response signature.

### Message body

The messaging format is standard compliant JSON using UTF-8 encoding. Each request and response has a set of mandatory and optional fields, which values are validated using the included regular expression rules. A GET request does not have a request body whereas a POST request often does, and both of these always receive response body.

## Financial Transactions

In order to do safe transactions, an execution model is used where the first call to `/transaction` acquires a financial transaction handle, later referred as “ID”, which ensures the transaction is executed exactly once. Afterwards it is possible to execute a debit or credit transaction by calling `/transaction/<ID>/debit` or `/transaction/<ID>/credit` respectively using the received ID handle. If the execution fails, the command can be repeated in order to confirm the transaction with the particular ID has been processed. After executing the command, the status of the transaction can be checked by executing a `GET` request to the `/transaction/<ID>` resource.

If the `blocking` parameter is set to false, the debit and credit calls return immediately, without waiting for the transaction to be fully processed. In this case one is encouraged to poll the `/transaction/<ID>` GET request to check the result of the transaction.

### Init transaction handle

Init the transaction handle and get the ID for it.

> Init request

```shell
POST /transaction
```

> Init response

```json
{
	"result":
	{
		"code": 100,
		"message": "OK"
	},
	"id": "4d185afe-3ef4-11e4-9d9f-164230d1df67"
}
```

#### HTTP Request

`POST /transaction`

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
id | M | UUID4 | The handle for the following requests.
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 

### Charge and refund transactions

> Debit request

```shell
POST /transaction/4d185afe-3ef4-11e4-9d9f-164230d1df67/debit

{
	"amount": 9900,
	"currency": "EUR",
	"token":
	{
		"id": "859aafdf-41fa-453a-a6a5-beff35e2f3b8"
	}
}
```

> Debit response

```json
{
	"result":
	{
		"code": 100,
		"message": "OK"
	}
}
```

#### HTTP Requests

`POST /transaction/<id>/debit`

`POST /transaction/<id>/credit`

Create a debit or a credit transaction. A debit transaction charges the card and a credit transaction refunds the card.

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
amount | M | AMOUNT | Amount in the smallest currency unit. E.g. 99.99 € = 9999
currency | M | CURRENCY |
blocking | O | | Default: true
card | M/O | |  Either `card` or `token` must be included
card.pan | M | |
card.expiry_year | M | |
card.expiry_month | M | |
card.cvc | O | SCODE |
card.verification | O | |
token | M/O | | Either `card` or `token` must be included
token.id | M | UUID4 |
token.cvc | O | SCODE |

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 

### Revert Transaction

Reverts an existing debit or credit transaction. Reverting a credit transaction needs to be supported by the acquirer.

> Revert request

> Revert response

#### HTTP Request

`POST /transaction/<id>/revert`

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
amount | M | AMOUNT | Amount in the smallest currency unit. E.g. 99.99 € = 9999
blocking | O | | Default: true

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 

### Retreive Transaction Status

> Status request

> Status response

#### HTTP Request

`GET /transaction/<id>`

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 
transaction | M | |
transaction.id | M | UUID4 | |
transaction.acquirer | M | | |
transaction.acquirer.id | M | |
transaction.acquirer.name | M | |
transaction.type | M | | Credit or debit
transaction.amount | M | AMOUNT | Original transaction amount in the smallest currency unit. E.g. 99.99 € = 9999
transaction.current_amount | O | AMOUNT | Amount after possible reversals
transaction.currency | M | |
transaction.timestamp | M | TIMESTAMP |
transaction.modified | M | TIMESTAMP | Latest transaction modification time
transaction.filing_code | M | |
transaction.authorization_code | O | |
transaction.verification_method | O | |
transaction.token | O | |
transaction.status | O | |
transaction.status.state | M | TSTATE |
transaction.status.code | M | TCODE |
transaction.status.messge | O | TMSG |
transaction.card | M | |
transaction.card.partial_pan | M | |
transaction.card.type | M | |
transaction.card.expire_year | M | |
transaction.card.expire_month | M | |
transaction.reverts | O | JSON Array | Present if the transaction is partially or fully reverted
transaction.reverts.type | M | REVTYPE |
transaction.reverts.status | M | |
transaction.reverts.status.state | M | TSTATE |
transaction.reverts.status.code | M | TCODE |
transaction.reverts.status.message | M | TMSG |
transaction.reverts.amount | M | AMOUNT | Amount in the smallest currency unit. E.g. 99.99 € = 9999
transaction.reverts.timestamp | M | TIMESTAMP |
transaction.reverts.modified | M | TIMESTAMP | Latest revert modification time
transaction.reverts.filing_code | O | |

## Tokenization

In order to be sure that a tokenized card is valid and is able to process payment transactions the corresponding sph-tokenization-id must be used to get the actual card token.

### How to get a Card Token

The card token is fetched by calling the tokenization URI with the `sph-tokenization-id`.

Technically the tokenization is already done by a Form API call such as `/form/view/add_card` as processing an authorization requires a valid CVC given by the cardholder. If the card is valid the Tokenization API response contains a card token that can be used to make transactions.

<aside class="notice">
The sph-tokenization-id from the Form API is random. However the tokenization always gives the same card_token for the same card information.
</aside>

> Tokenization request

> Tokenization response

#### HTTP Request

`GET /tokenization/<id>`

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
card_token | O | UUID4 | Card Token if verification was successful
card | O | |  Card if verification was successful
card.type | O | | Card type, for example 'Visa'
card.partial_pan | O | | Last four digits of the card
card.expire_year | O | | Card expiration month
card.expire_month | O | | Card expiration year
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 

### Transaction Commit

In order to charge a card given in the Form API `/form/view/pay_with_card` or `form/view/add_and_pay_with_card` the corresponding `sph-transaction-id` must be committed.

#### HTTP Request

`POST /transaction/<id>/commit`

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
amount | M | AMOUNT | Amount in the smallest currency unit. E.g. 99.99 € = 9999
currency | M | CURRENCY |
blocking | O | | Default: true

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
card_token | O | UUID4 | Card token is present if the commit was successful and the card tokenization was originally requested using `/form/view/add_and_pay_with_card`
card | O | | Card is present with the same rules as the `card_token` above
card.partial_pan | O | |
card.type | O | | Card type, for example ‘Visa’
card.partial_pan | O | | Last four digits of the card
card.expire_year | O | | Card expiration month
card.expire_month | O | | Card expiration year
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 

### Daily Batch Report

Fetch a daily report of the settlements and their transactions for all of the merchants under the account.

#### HTTP Request

`GET /report/batch/<yyyyMMdd>`

#### HTTP Response

Field / Object | M/O | Data type | Description
-------------- | --- | --------- | -----------
result | M | |
result.code | M | RCODE |
result.message | M | RMSG | 
settlements[] | M | Array | May be empty
settlements[].status | M | |
settlements[].status.state | M | SSTATE |
settlements[].status.code | M | SCTATE |
settlements[].status.message | O | SMSG |
settlements[].id | M | UUID4 |
settlements[].batch | M | String |
settlements[].timestamp | M | TIMESTAMP |
settlements[].reference | O | REFERENCE |
settlements[].merchant | M | |
settlements[].merchant.id | M | String |
settlements[].merchant.name | M | String |
settlements[].acquirer | M | |
settlements[].acquirer.id | M | String |
settlements[].acquirer.name | M | String |
settlements[].transaction_count | M | Integer |
settlements[].net_amount | M | NETAMOUNT | Total amount, can be negative
settlements[].currency | M | String |
settlements[].transactions[] | M | Array | May be empty
settlements[].transactions[].id | M | UUID4 | Non unique! I.e. related transactions may share and ID
settlements[].transactions[].timestamp | M | TIMESTAMP |
settlements[].transactions[].type | M | String | Debit or credit
settlements[].transactions[].partial_pan | M | String | Masked PAN
settlements[].transactions[].amount | M | AMOUNT |
settlements[].transactions[].currency | M | String |
settlements[].transactions[].filing_code | M | String |
settlements[].transactions[].authorization_code | O | String |
settlements[].transactions[].verification_method | O | String | Applies to 3D Secure verification
settlements[].transactions[].status | M | |
settlements[].transactions[].status.state | M | SSTATE |
settlements[].transactions[].status.code | M | SCODE |
settlements[].transactions[].status.message | O | SMSG |