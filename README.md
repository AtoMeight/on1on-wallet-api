# On1on Wallet API

## Base URL:

https://sandbox-api.on1on-wallet.com/v1
https://api.on1on-wallet.com/v1

## API Authentication

ApiKey: pk_xxxxx
SecretKey: sk_xxxxx

```text
Header: Authorization={pk_xxxx}
```

* ApiKey is for client identification.
* SecretKey is for request signing.
* Never share either of the keys with public.

## Idempotency

For POST deduplication, optional

```html
Header: ow-idempotency-key={your idempotency key}
```

## Basic Request

In order to prevent replay attacks and sniffing attacks, we need to sign and verify the request body.

```text
Header: ow-signature={timestamp_second}-{signature}

timestamp_second = unix timestamp of milliseconds, e.g. 1652683011792
signature = HMAC_SHA256(str(RequestBody) + str(timestamp_second), SecretKey).
```

e.g.

```text
RequestBody={
  "user_id": 123
}
TS=1652622423
SecretKey = abc
HMAC_SHA256(str(RequestBody) + str(timestamp_second), SecretKey)=
HMAC_SHA256({"user_id": 123}1652683011792, abc)=
48ba83ce2676f123e3db6c03fe2174bd96c4f058ceb68e23b2fb6523f1afa164
```

Request will expire in 5 minutes. Please make sure your machine has the correct time.

## Create Customer

To create a customer in On1on1 Wallet.
Idempotency: If the user's email already exists and matching the customer_id, fulfill silently.

```text
POST /customer
```

| Field       | Description                                | Type   | Length | SampleValue         |
|-------------|--------------------------------------------|--------|--------|---------------------|
| email       | user email                                 | String | <255   | sample@test.com     |
| customer_id | Customer Id in your system, must be unique | String | <255   | User-00001          |
| metadata    | Anything you want to store in our database | String | <2048  | {"data":"anything"} |

# Get Customer Info

Get customer by email or by customer_id

```text
GET /customer/{identifier}/info
```

| Field       | Description                                | Type   | Length | SampleValue         |
|-------------|--------------------------------------------|--------|--------|---------------------|
| identifier  | user email or customer_id                  | String | <255   | sample@test.com     |

# Update Customer

Update customer's metadata.
Other fields cannot be updated.

```text
PATCH /customer/{identifier}
```

| Field       | Description                                | Type   | Length | SampleValue         |
|-------------|--------------------------------------------|--------|--------|---------------------|
| metadata    | Anything you want to store in our database | String | <2048  | {"data":"anything"} |

# Create Transfer (Synchronized)

Transfer assets from one account to another.
Both account should be existing in On1on Wallet

```text
POST /transfer
```

| Field       | Description                                                                     | Type   | Length | SampleValue         |
|-------------|---------------------------------------------------------------------------------|--------|--------|---------------------|
| source      | User identity for source, either email or customer_id                           | String | <255   | sample@test.com     |
| destination | User identity for destination, either email or customer_id                      | String | <255   | sample@test.com     |
| currency    | Token identifier                                                                | String | <255   | GOLDEN              |
| amount      | Token amount to transfer                                                        | String | <64    | "123.456789"        |
| reference   | A reference you can later use to identify this payment, such as an order number | String | <64    | "123.456789"        |
| description | Description for this transfer                                                   | String | <128   | "airdrop"           |
| metadata    | Anything you want to store in our database                                      | String | <128   | {"data":"anything"} |

Response Data Fields

| Field         | Description                                              | Type   | Length | SampleValue            |
|---------------|----------------------------------------------------------|--------|--------|------------------------|
| tx_id         | Transaction unique Id                                    | String | <=18   | OW262382364717125      |
| status        | Transfer status, pending, done, failed                   | String | <16    | done                   |
| code          | Transfer code, error code if failed                      | String | <64    | ERR_BALANCE_NOT_ENOUGH |
| request_time  | Timestamp in milliseconds when the transfer is requested | Int64  | 64bit  | 1652683011792          |
| captured_time | Timestamp in milliseconds when the transfer is captured  | Int64  | 64bit  | 1652683011792          |

# Get Transfer Detail

Get the detail information for a transfer event

```text
GET /transfer/{tx_identifier}
```

| Field         | Description                                       | Type   | Length | SampleValue       |
|---------------|---------------------------------------------------|--------|--------|-------------------|
| tx_identifier | Transaction identifier, either tx_id or reference | String | <255   | OW262382364717125 |

Response Data Fields

| Field         | Description                                                                     | Type   | Length | SampleValue            |
|---------------|---------------------------------------------------------------------------------|--------|--------|------------------------|
| tx_id         | Transaction unique Id                                                           | String | <=18   | OW262382364717125      |
| source        | User identity for source, either email or customer_id                           | String | <255   | sample@test.com        |
| destination   | User identity for destination, either email or customer_id                      | String | <255   | sample@test.com        |
| currency      | Token identifier                                                                | String | <255   | GOLDEN                 |
| amount        | Token amount to transfer                                                        | String | <64    | "123.456789"           |
| reference     | A reference you can later use to identify this payment, such as an order number | String | <64    | "123.456789"           |
| description   | Description for this transfer                                                   | String | <128   | "airdrop"              |
| metadata      | Anything you want to store in our database                                      | String | <128   | {"data":"anything"}    |
| status        | Transfer status, pending, done, failed                                          | String | <16    | done                   |
| code          | Transfer code, error code if failed                                             | String | <64    | ERR_BALANCE_NOT_ENOUGH |
| request_time  | Timestamp in milliseconds when the transfer is requested                        | Int64  | 64bit  | 1652683011792          |
| captured_time | Timestamp in milliseconds when the transfer is captured                         | Int64  | 64bit  | 1652683011792          |

# List Transfers

List the recent transfer records

```text
GET /transfers?source={source}&destination={destination}&identity={identity}&currency={currency}&
status={status}&start_time={start_time}&end_time={end_time}&order_by={order_by}&order_direction={order_direction}
```

Query Parameters

| Field           | Description                                                                                                                              | Type   | Length | SampleValue     |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------|--------|--------|-----------------|
| source          | User identity for source, either email or customer_id                                                                                    | String | <255   | sample@test.com |
| destination     | User identity for destination, either email or customer_id                                                                               | String | <255   | sample@test.com |
| identity        | User identity for either source or destination, either email or customer_id. If this provided, source or destination will be overridden. | String | <255   | sample@test.com |
| currency        | Token identifier                                                                                                                         | String | <255   | GOLDEN          |
| status          | Transfer status, pending, done, failed                                                                                                   | String | <16    | done            |
| start_time      | start request_time in ms                                                                                                                 | Int64  | 64bit  | 1652683011792   |
| end_time        | request_time in ms                                                                                                                       | Int64  | 64bit  | 1652683011792   |
| order_by        | request_time                                                                                                                             | Int64  | 64bit  | 1652683011792   |
| order_direction | ASC, DESC                                                                                                                                | String | 64bit  | DESC            |
| page            | Page starts from 1                                                                                                                       | Int    | 32bit  | 2               |
| size            | Size, default to 10, max 100                                                                                                             | Int    | 32bit  | 50              |

# Get Balance of Customer

Get balance of customer by email or by customer_id

```text
GET /customer/{identifier}/balance
```

| Field       | Description                                | Type   | Length | SampleValue         |
|-------------|--------------------------------------------|--------|--------|---------------------|
| identifier  | user email or customer_id                  | String | <255   | sample@test.com     |

Response Data Fields

| Field | Description       | Type      | Length | SampleValue |
|-------|-------------------|-----------|--------|-------------|
| data  | Array of Balances | Balance[] | ---    | []          |

struct Balance

| Field    | Description              | Type   | Length | SampleValue  |
|----------|--------------------------|--------|--------|--------------|
| currency | Token identifier         | String | <255   | GOLDEN       |
| amount   | Token amount to transfer | String | <64    | "123.456789" |
