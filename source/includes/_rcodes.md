# Response and Status Codes

##RCODE Result Codes

RCODE | RMSG | Description, actions
----- | ---- | --------------------
100 | Description of the succesful request | Request successful.
200 | Reason for the failure | Authorization failed, unable to create a Debit transaction or Revert failed.<br />For Debit transactions, please initialize a new transaction the next day (in case there was insufficient funds) and/or contact the cardholder.<br />For transaction reverts, please see the status of the transaction with `GET /transaction/<id>`
300 | | Transaction in progress. Given when there is already a debit transaction being processed with the ID.
900 | Description of the error | Could not process the transaction, please try again.
901 | Description of the validation error | Invalid input. Detailed information is in the `message field.
910 | Description of the operation type error | Invalid operation type. Either a credit transaction is performed on a debit ID or vice versa.
920 | Description of the erroneous request parameters. | Unmatched request parameters. Request parameters do not match the previous parameters with the current transaction ID.
990 | Description of the error | Permanent failure. Cannot repeat the transaction. Please initialize a new transaction.

##TCODE Transaction Status Codes

TCODE | TSTATE
----- | ------
3000 | "in_progress"
4000 | "ok"
4100 | "ok_pending"
4200 | "ok_estimated_pending"
5700 | "reverted"
5800 | "reverting"
7000 | "failed"
7100 | "timeout"


##SCODE Settlement Status Codes

SCODE | SSTATE | Description
----- | ------ | -----------
3000 | "pending" | Settlement has not yet been transmitted to the acquiring bank.
4000 | "ok" | Settlement is processed OK.