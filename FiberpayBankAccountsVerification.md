## BANK ACCOUNTS VERIFICATION

### Authentication
### API Keys
In order to use the API, it is necessary to generate API keys using the [developer panel](https://verify.fiberpay.pl/dashboard/keys).

Keys used in examples:
| Key | Value |
|-----|-------|
| API_KEY | JqkZE5zQ23DmQFmv |
| SECRET | 75cefd408bd195cead3de06196e885958cf444593603f503ae08602266f854db |

### Headers
Each API request must include the `API-Key` header with your generated public key.

## Encoding and Encryption

### JWT Token
The body of each HTTP request should be encoded as a JWT token using the secret key for signature. For GET methods, the token should contain an empty string ("") and be passed in the Authorization header in the following format: "Bearer {token}".

For information on how to decode or which libraries to use for handling JWT tokens, visit: https://jwt.io/

### Encryption
The API response body is always encoded as a JWT token signed with the secret key, and the payload data is additionally encrypted using AES-256 in CBC mode.

For decryption, you need to use the secret key and the initialization vector (IV, which is public and unique for each server response). It is transmitted in the token.

Note that libraries used for decryption require the key and initialization vector in binary format.

Example implementation in PHP:
```php
openssl_decrypt($ciphertext, 'aes-256-cbc', hex2bin($secret), 0, hex2bin($iv));
```

Example implementation in NodeJS:
```javascript
const crypto = require("crypto");

const decrypt = (ciphertext, secret, iv) => {
    const decipher = crypto.createDecipheriv(
        "aes-256-cbc",
        Buffer.from(secret, "hex"),
        Buffer.from(iv, "hex")
    );
    let plaintext = decipher.update(ciphertext, "base64", "utf8");
    plaintext += decipher.final("utf8");
    return plaintext;
};
```

### Callback Mechanism
When verification status changes, the system sends an HTTP request (POST method) to the previously specified address (`callbackUrl`), where:

- The request body contains a JWT token with encrypted (as described above) current order data (payload)
- The request's `API-Key` header contains the public API key

## Order Statuses

Bank account verification orders can have one of the following statuses:

| Status | Description |
|--------|-------------|
| new | Initial state of an order after creation |
| accepted | The verification was successful and the bank account(s) were verified |
| rejected | The verification failed or was rejected |

You will receive these statuses through the callback mechanism when the verification status changes.

## Create Order

### Endpoint
**POST** `https://apiver.fiberpay.pl/api/orders/bank-accounts`

### Request Parameters

| Name | Is Required | Value | Description |
|------|-------------|-------|-------------|
| type | Yes | individual | Type of verification order |
| description | Yes | String | Description that will be visible to the end user |
| callbackUrl | Yes | URL | URL that will receive callback notifications |
| redirectUrl | Yes | URL | URL where user will be redirected after verification |
| params | Yes | Object | Object containing additional parameters |
| params.firstName | Yes | String | First name of the individual |
| params.lastName | Yes | String | Last name of the individual |

### Request Body Example
```json
{
    "type": "individual",
    "description": "Description visible to the user",
    "callbackUrl": "https://some-callback-url.com",
    "redirectUrl": "https://some-redirect-url.com",
    "params": {
        "firstName": "John",
        "lastName": "Doe"
    }
}
```

### Encoded Body Example
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiaW5kaXZpZHVhbCIsImRlc2NyaXB0aW9uIjoiRGVzY3JpcHRpb24gdmlzaWJsZSB0byB0aGUgdXNlciIsImNhbGxiYWNrVXJsIjoiaHR0cHM6XC9cL3NvbWUtY2FsbGJhY2stdXJsLmNvbSIsInJlZGlyZWN0VXJsIjoiaHR0cHM6XC9cL3NvbWUtcmVkaXJlY3QtdXJsLmNvbSIsInBhcmFtcyI6eyJmaXJzdE5hbWUiOiJKb2huIiwibGFzdE5hbWUiOiJEb2UifX0.tl_PwJxtZ2g6dLnLRed9sM1WJETFe28D3Fvv2RgEXjs
```

### Example Request (cURL)
```sh
curl -X POST "https://apiver.fiberpay.pl/api/orders/bank-accounts" \
     -H "Content-Type: application/json" \
     -H "API-Key: JqkZE5zQ23DmQFmv" \
     -d "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiaW5kaXZpZHVhbCIsImRlc2NyaXB0aW9uIjoiRGVzY3JpcHRpb24gdmlzaWJsZSB0byB0aGUgdXNlciIsImNhbGxiYWNrVXJsIjoiaHR0cHM6XC9cL3NvbWUtY2FsbGJhY2stdXJsLmNvbSIsInJlZGlyZWN0VXJsIjoiaHR0cHM6XC9cL3NvbWUtcmVkaXJlY3QtdXJsLmNvbSIsInBhcmFtcyI6eyJmaXJzdE5hbWUiOiJKb2huIiwibGFzdE5hbWUiOiJEb2UifX0.tl_PwJxtZ2g6dLnLRed9sM1WJETFe28D3Fvv2RgEXjs"
```

### Example Response
```json
{
  "data": {
    "code": "kdspmjhb64wf",
    "type": "individual",
    "status": "new",
    "callbackUrl": "https://some-callback-url.com",
    "redirectUrl": "https://some-redirect-url.com",
    "createdAt": "2025-03-05T05:36:40.000000Z",
    "url": "https://verify.fiberpay.pl/bank-accounts/kdspmjhb64wf",
    "bankAccounts": [],
    "availableIdentityVerificationMethods": [
      "banqup",
      "fiberpay_transfer"
    ]
  }
}
```

---

## Callbacks

### Callback Handling Example
When a callback is triggered, you'll receive a POST request with an encoded payload.
Below is an example of how to handle an incoming callback:

#### Example cURL Call from our Server to your Callback URL
```sh
curl -X POST "https://some-callback-url.com" \
    -H "API-Key: JqkZE5zQ23DmQFmv" \
    -d "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXlsb2FkIjoiUlY4TUtraWpcL21KU0ordVhEZkpqRVI2bnQ5..."
```

### Failed callback payload example

#### Encoded JWT
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXlsb2FkIjoiUlY4TUtraWpcL21KU0ordVhEZkpqRVI2bnQ5cjhwZ2ZCYjlRcVlTaUJNTTliTXpYUE05TVBkSGFBSUN6TmM5VFdzQzhTZkNpSk1oZG12SFRwZXpoTkUxYTROYVFGOG5BYUU1OERlemZFZXllUWhldVg4TXZjQzJ2NGc0b0VseFhSNk1BN0VrWEJZbXdtRStpUkNZS01SMWxpXC9nNDBFRWhtVVZHenZySVMybWlqYUU4QVIxRFZ5a0NEQkpzTzI2NktBeTN6REl0YnZycW9MTTk1N1A4Tll6V1o5WEZFMUpPd2hqV3AzWHVvQWlEeEZNdTRcL0JUMGlTU2M3TUJBbHhEQmZwUG9tODJQZzVwaytXK2xCMVwvc2ZGTmo4dE15MWpxdUJFdUo2cE9zQXlvY2g1R1FsTEZoVzJiWlF4VjlXamtQbm4raytcL0lsRWZadFdseVRSd2VzSjlZMUxvWVBkaE9IdlwvSmpVcVZRNkJwc0tFRWlnWE9cL1E5Mm1tRlNDOTNIQkxaOGJpVnQ3XC9JRW1yZ2ZDZEZBK3U3eFJTYVdwVWE2bmN3Vk1EQlZyUjQrNXpaN21ONGxJcFZRUEZBRktoQUQzQmN0anEzWlR0UWZHaDF1dWFVR0FrejVkTFJCOFRxK0V4Mmh5TXRMWE5Wdz0iLCJpc3MiOiJWZXJpZmljYXRpb24iLCJpYXQiOjE3NDExNTM5ODMsIml2IjoiMjliYzliYWU4YjU5NzUwYjYyMWE2OTU3M2NlNTE4NTgifQ.hJyF3TgL1mt7PLd1_KeBPg_LW6CBPYID_577K695ops
```

#### Decoded JWT
```json
{
  "payload": "RV8MKkij/mJSJ+uXDfJjER6nt9r8pgfBb9QqYSiBMM9bMzXPM9MPdHaAICzNc9TWsC8SfCiJMhdmvHTpezhNE1a4NaQF8nAaE58DezfEeyeQheuX8MvcC2v4g4oElxXR6MA7EkXBYmwmE+iRCYKMR1li/g40EEhmUVGzvrIS2mijaE8AR1DVykCDBJsO266KAy3zDItbvrqoLM957P8NYzWZ9XFE1JOwhjWp3XuoAiDxFMu4/BT0iSSc7MBAlxDBfpPom82Pg5pk+W+lB1/sfFNj8tMy1jquBEuJ6pOsAyoch5GQlLFhW2bZQxV9WjkPnn+k+/IlEfZtWlyTRwesJ9Y1LoYPdhOHv/JjUqVQ6BpsKEEigXO/Q92mmFSC93HBLZ8biVt7/IEmrgfCdFA+u7xRSaWpUa6ncwVMDBVrR4+5zZ7mN4lIpVQPFAFKhAD3Bctjq3ZTtQfGh1uuaUGAkz5dLRB8Tq+Ex2hyMtLXNVw=",
  "iss": "Verification",
  "iat": 1741153983,
  "iv": "29bc9bae8b59750b621a69573ce51858"
}
```

#### Decrypted Payload
```json
{
  "code": "kdspmjhb64wf",
  "type": "individual",
  "status": "rejected",
  "callbackUrl": "https://some-callback-url.com",
  "redirectUrl": "https://some-redirect-url.com",
  "createdAt": "2025-03-05T05:36:40.000000Z",
  "url": "https://verify.fiberpay.pl/bank-accounts/kdspmjhb64wf",
  "bankAccounts": [],
  "availableIdentityVerificationMethods": [
    "banqup",
    "fiberpay_transfer"
  ]
}
```

---

### Success Callback Payload Example
#### Encoded JWT
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwYXlsb2FkIjoiWmRlMThcL2doZWFKSHV1NFdrTkc2MEZVVzI5ZCtOdXI5ZHFyNWZmbVRNSHR2UkoraWx4cmRuQ2U5am5uZnVDZjhXcmF5OUNucGtpNDQ4Nk9vT2NpQzd0YkpqOWM3T0lCdmN2eGEwQ3ZqaVBEdmh3bVwvaTh0cTg3KzhEczVUdnE2NHZpeXd6c3U4cFVmKzlkQmxhZU14UnZtWjFxSHBzQklXd2dmUkI3dmRoWW5hXC9KV015MGplNWJuMGMyd1l6VEdnR2FIanJLV3B1eG52bUh0aUs0NUUyTlc2VjVlUU5wV3VOc1A5XC85WnlYSFpKdXV2RSs4VVdnMllLaVV3STBNM2N5d0lydUJycWJMZ0p4cktvdVVBWVR0VWZPZ042ajY4dnJ4YnRxVWIwVEZHMFp2dlwvdUFcL3dBZTFFS1JEM3dMTFwvaER1YnNNNkp1SHV5UFJ0Nm1JUXdlR1N1SUF6RDBaVWFLcEZkRFNXc2FaWWRWbmc0a0FNRkVTZnZTdzBnTmxUZ1VWVmJndHYxdFdUVXd6NVlRWVlPeXJrTVpFazlFdEcrSitIOW1jRU9vYVNuMGdBdzdTRjNYcHl3U09QMDFTUmlPZktaNit6dm8rY3VzUkcxalJsVlpRVU5jZ3BjTjFjTzFZTnNpQnZIZmtCQjg3WjJUdEZjMnZUbTl0dmptTG5hR2t1UU9TVFBZdUViRFdTSWYwUk5YbXVLam01c3pBMFBZUCtic3NSU1wvWjdlRTk3MUtOVFBRcElcLzFFYm8rc1kwdlNqYXBRMlwvb3UzVkRCdDdzSXdpRE93cWxEVDRQSHRwdjl2NDJidTJtaXlleGlUQmR2RzFlWms4c0lVSTBVOWlTT1JGbkdla2FmQkdpd2Vkc1ZNajY0NndRVGc2NnQ5WGRZSjNCZ2FtcnpTZVRFXC9TazFHZ2ZDamJFeWNlR0V0aSIsImlzcyI6IlZlcmlmaWNhdGlvbiIsImlhdCI6MTc0MTE1NDQ5MSwiaXYiOiJkMGVjZjJhZTQ0NDY1ZDk4YTVjODRmZjIxM2JkZmFkZCJ9.QUZ0AT_XdS6tCdmNCuQwLllWSwKVjy20VzgyczjTnjI
```

#### Decoded JWT
```json
{
  "payload": "Zde18/gheaJHuu4WkNG60FUW29d+Nur9dqr5ffmTMHtvRJ+ilxrdnCe9jnnfuCf8Wray9Cnpki4486OoOciC7tbJj9c7OIBvcvxa0CvjiPDvhwm/i8tq87+8Ds5Tvq64viywzsu8pUf+9dBlaeMxRvmZ1qHpsBIWwgfRB7vdhYna/JWMy0je5bn0c2wYzTGgGaHjrKWpuxnvmHtiK45E2NW6V5eQNpWuNsP9/9ZyXHZJuuvE+8UWg2YKiUwI0M3cywIruBrqbLgJxrKouUAYTtUfOgN6j68vrxbtqUb0TFG0Zvv/uA/wAe1EKRD3wLL/hDubsM6JuHuyPRt6mIQweGSuIAzD0ZUaKpFdDSWsaZYdVng4kAMFESfvSw0gNlTgUVVbgtv1tWTUwz5YQYYOyrkMZEk9EtG+J+H9mcEOoaSn0gAw7SF3XpywSOP01SRiOfKZ6+zvo+cusRG1jRlVZQUNcgpcN1cO1YNsiBvHfkBB87Z2TtFc2vTm9tvjmLnaGkuQOSTPYuEbDWSIf0RNXmuKjm5szA0PYP+bssRS/Z7eE971KNTPQpI/1Ebo+sY0vSjapQ2/ou3VDBt7sIwiDOwqlDT4PHtpv9v42bu2miyexiTBdvG1eZk8sIUI0U9iSORFnGekafBGiwedsVMj646wQTg66t9XdYJ3BgamrzSeTE/Sk1GgfCjbEyceGEti",
  "iss": "Verification",
  "iat": 1741154491,
  "iv": "d0ecf2ae44465d98a5c84ff213bdfadd"
}
```

#### Decrypted Payload
```json
{
  "code": "kdspmjhb64wf",
  "type": "individual",
  "status": "accepted",
  "callbackUrl": "https://some-callback-url.com",
  "redirectUrl": "https://some-redirect-url.com",
  "createdAt": "2025-03-05T05:36:40.000000Z",
  "url": "https://verify.fiberpay.pl/bank-accounts/kdspmjhb64wf",
  "bankAccounts": [
    {
      "code": "wvzrh32ce9fp",
      "iban": "PL94105018231000009739652767",
      "provider": "banqup",
      "created_at": "2025-03-05T06:01:31.000000Z",
      "updated_at": "2025-03-05T06:01:31.000000Z"
    }
  ],
  "availableIdentityVerificationMethods": [
    "banqup",
    "fiberpay_transfer"
  ]
}
```
