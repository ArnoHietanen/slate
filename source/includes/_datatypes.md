# Data Types

Type | Format | Example
---- | ------ | ------
N | ^[0-9]+$ | 1200
ACCOUNT | ^[-\w.]{1,500}$ |
AMOUNT | ^\d{1,12}$ | 9900 (meaning 99.00 for duodecimal currencies)
AN | ^[0-9a-zA-Z]+$ | sampleText001
ANS | ^[ -~]+$ | ~/sample/text! &lt;with@specials&gt;
CURRENCY | ^(EUR)$ | EUR
MERCHANT | ^[-\w.]{1,500}$ |
MONTH | ^[0-9]{2}$ | 02
NETAMOUNT | ^-?\d{1,12}$ | -1590 (meaning -15.90 for duodecimal currencies)
PAN | ^[3-6]\d{12,18}$ |
RCODE | ^\d{1,6}$ | 100
REFERENCE | ^\d{12}$ |
REVTYPE | ^(cancellation&#124;refund)$ | cancellation
RMSG | ^[.,'-=/\w;\s]{0,1023}$ |
SCODE | ^\d{3,4}$ | 3000
SMSG | ^[.,'-=/\w;\s]{0,1023}$ |
SSTATE | ^[.,'-=/\w;\s]{0,254}$ |
TCODE | ^\d{4}$ | 4000
TMSG | ^[.,'-=/\w;\s]{0,1023}$ |
TSTATE | ^[.,'-=/\w;\s]{0,254}$ | ok
UUID4 | ^[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}$ | f47ac10b-58cc-4372-a567-0e02b2c3d479
URL | Valid URL with HTTPS scheme | https://example.com/success?myparam=abc
TIMESTAMP | ^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z$ | 2014-09-18T10:32:59Z
YEAR | ^[0-9]{4}$ | 2015
