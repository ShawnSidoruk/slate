---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - code

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:

search: true

code_clipboard: true
---
# Introduction

Betable API is a RESTful service providing the ability to create a compliant wallet solution while powering CRM, fraud, and payment facilities for a regulated gaming solution.

## Change History

Version | Date | Summary of Changes
--------- | ------- | -----------
1.0 | 2020-03-23 | Initial release of Betable-LFS API.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## Quick Reference

All API access is over HTTPS, and accessed from the `ns-api.betable.com` domain. All API requests must be authenticated or a `401 Unauthorized error` will be returned (see Authentication).

A `User` corresponds to each unique player in the vendor system and contains profile and preference information.

A `Session` is created for every login event by a player.

A `Round` is a higher-level construct which can hold a series of wager and payout actions.

A `Wager` is any entry or bet placed by a player.

A `Payout` represents an amount won by a player.


# API Overview

`Content-Type: application/json`

The Betable API is implemented using a REST-style service, using a JSON content type for both the request and response payloads.

All API method calls must be made using a secure connection with TLS/1.2.

## API Endpoints

On staging, all API endpoints begin with `https://ns-api-staging.betable.com/lfs/v100`.

On production, all API endpoints begin with `https://ns-api.betable.com/lfs/v100`.

## IP Whitelisting
The Betable API exposed in this document is not a public API.  In addition to other protocol measures, only a list of static IP Address(es) will be permitted to communicate on this API.  

<aside class="warning">
Any API method calls made from a non-whitelisted IP Address will be blocked.
</aside>

## Authentication

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
...
```

`X-API-KEY: [vendor-api-key]`

An API key will be provided to the integration partner which is to be used in all requests to the Betable API unless stated. The API key must be kept secret and should only be used when making requests to the Betable API as a form of authentication.

<aside class="notice">
A separate API key will be provided for both staging and production environments.
</aside>
<aside class="warning">
A missing or invalid API key will produce a `401 Unauthorized` error.
</aside>

## Request Tracing (Optional)

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-H "X-Trace-ID: [vendor-generated-trace-id]" \
...
```

`X-Trace-ID: [vendor-generated-trace-id]`

Although not required, all requests are encouraged to include a unique identifer in the header. Tracing is not used in operational integrity but can be used as a reference in post-processing as the header will be reflected on the response and can be used to correlate a request and response. The `X-Trace-ID` reference is only stored for for 14 days.

<aside class="notice">
The <code>X-Trace-ID</code> should also be provided, where possible, for support requests.
</aside>

## Idempotent Requests (Optional)

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-H "X-Idempotency-Key: [vendor-generated-key]" \
...
```

`X-Idempotency-Key: [vendor-generated-key]`

The API supports idempotency for safely retrying requests without accidentally performing the same operation twice. A unique identifier added to the header on `POST` or `PUT` requests ensures that the same request is not mistakenly processed multiple times. If a duplicate request is sent then the original response will be returned without any processing occurring. This includes all response codes including `500` errors, so a new key should be created for each independent request. The Betable API will store the idempotent response for 30 days.

## Common Data Types

The Betable API uses the standard JSON data types for all fields of the request and response payloads with the addition of these custom data types:

Data Type | JSON Type | Description | Example Value(s)
--------- | ------- | ----------- | ------
`Date` | string | Using the following format: yyyy-MM-dd | 1969-04-20
`Money` | string | All monetary amounts are represented as a string. | 199.99

## Response Codes and Statuses

### HTTP Statuses

HTTP Status Code | Description
---------------- | -----------
`200` | Request processed, response contains details.
`201` | Request processed, entity created, response contains details.
`204` | Request processed, no response details.
`208` | Request previously proccessed successfully, response contains details from original request.
`400` | Validation error.
`401` | Unauthorized, see section: Authentication
`404` | Requested entity not found.
`409` | Request not processable due to business logic, entity state, or other reason(s).
`500` | Unexpected runtime error.  May try again.

### Error Codes

> An example JSON response when receiving error code `400`:

```json
{
  "error_description": "user_id cannot be empty"
}
```

For each non-`2XX` status code returned, a JSON body will be returned to provide more context about the error encountered.


Name | Type | Description
 ---| ---| ---
error\_description | string | A description of the error encountered; see common errors below or each API method section.

# Health Check
## Health

`GET /health`

This method is used to ensure communication with the Betable API is functional. The main usage of this endpoint is to assure that the IP Addresses are properly whitelisted.

<aside class="comment">
The <code>X-API-Key</code> is not required to access this API method.
</aside>


### Response

> Example JSON Response:

```json
{
  "alive": true
}
```

A successful request returns the HTTP `200 OK` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| alive | boolean | true | True if the NSBS is operational and ready for communication. |



# Users

## Create User

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X POST \
-d '{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@doe.com",
  "external_user_id": "vendor-user-id-1",
  "date_of_birth": "1969-04-20",
  "mobile_phone": "18881112223",
  "address": {
    "address1": "Unit 7",
    "address2": "Cornell Tower",
    "address3": "Gladwell Street",
    "postcode": "12345",
    "city": "Springfield",
    "state": "IL",
    "country": "US",
  },
  "comms_preferences": {
    "email": true,
    "sms": true,
    "phone": false,
    "post": false,
  },
  "affiliate_details": [
    {
      "affiliate_program_id": "adwords",
      "affiliate_campaign_id": "ga-campaign-2020-xmas",
    },
  ],
  "last_accepted_terms": "1999-12-25 11:59:59.999"
}' https://ns-api-staging.betable.com/lfs/v100/users
```
`POST /users`

This method is create a new user in the Betable API and should be called at the same time that a new user is created in the vendor's platform.

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| first\_name | string | John | The user's first name. |
| last\_name | string | Doe | The user's last name. |
| email | string | john@doe.com | The user's email. |
| external\_user\_id | string | vendor-user-id-1 | A unique ID that can be used to identify the user's account in the PGP. |
| date\_of\_birth | Date | 1969-04-20 | Optional. The user's date of birth. |
| mobile\_phone | string | 18881112223 | Optional. The user's mobile phone number. |
| address | object | \- | Optional. |
| address.address1 | string | Unit 7 | Optional. The user's address. Note that formatting of the user's address and the use of the subsequent address fields may vary by region. |
| address.address2 | string | Cornell Tower | Optional. Additional address information, may vary by region. |
| address.address3 | string | Gladwell Street | Optional. Additional address information, may vary by region. |
| address.city | string | Springfield | The user's city/town. |
| address.state | string | IL | The user's state/province. |
| address.postcode | string | 12345 | The user's postcode/zipcode. |
| address.country | string | US | The user's country represented by it's ISO-3166 2-letter code. |
| comms\_preferences | object | \- | Optional. |
| comms\_preferences.email | boolean | true | Optional. True if the user has opted-in to email marketing communications. False or not present if the user has opted-out. |
| comms\_preferences.sms | boolean | true | Optional. True if the user has opted-in to SMS marketing communications. False or not present if the user has opted-out. |
| comms\_preferences.phone | boolean | true | Optional. True if the user has opted-in to telephone marketing communications. False or not present if the user has opted-out. |
| comms\_preferences.post | boolean | true | Optional. True if the user has opted-in to post/lettermail marketing communications. False or not present if the user has opted-out. |
| affiliate\_details | object | \- | Optional. |
| affiliate\_details.affiliate\_program\_id | string | adwords | A unique identifier for the affiliate program the user was acquired by. |
| affiliate\_details.affiliate\_campaign\_id | string | ga-campaign-2020-xmas | A unique identifier for the affiliate marketing campaign the user was acquired by. |

### Response

> Example JSON Response:

```json
{
  "user_id": "QEaakM0qumTJd5tj"
}
```

A successful request returns the HTTP `201 Created` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |


| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `external_user_id` was already processed |
| `409 Conflict` | Email address already in use |


## Update User

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@doe.com",
  "date_of_birth": "1969-04-20",
  "mobile_phone": "18881112223",
  "address": {
    "address1": "Unit 7",
    "address2": "Cornell Tower",
    "address3": "Gladwell Street",
    "city": "Springfield",
    "state": "IL",
    "country": "US",
  },
  "comms_preferences": {
    "email": true,
    "sms": true,
    "phone": false,
    "post": false,
  },
  "affiliate_details": [
    {
      "affiliate_program_id": "adwords",
      "affiliate_campaign_id": "ga-campaign-2020-xmas",
    },
  ],
  "last_accepted_terms": "1999-12-25 11:59:59.999"
}' https://ns-api-staging.betable.com/lfs/v100/users
```

`PUT /users/{user_id}`

This method updates a user's account information.

### URI Parameters
| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| first\_name | string | John | Optional. The user's first name. |
| last\_name | string | Doe | Optional. The user's last name. |
| email | string | john@doe.com | Optional. The user's email. |
| date\_of\_birth | Date | 1969-04-20 | Optional. The user's date of birth. |
| mobile\_phone | string | 18881112223 | Optional. The user's mobile phone number. |
| address | object | \- | Optional. |
| address.address1 | string | Unit 7 | Optional. The user's address. Note that formatting of the user's address and the use of the subsequent address fields may vary by region. |
| address.address2 | string | Cornell Tower | Optional. Additional address information, may vary by region. |
| address.address3 | string | Gladwell Street | Optional. Additional address information, may vary by region. |
| address.city | string | Springfield | The user's city/town. |
| address.state | string | IL | The user's state/province. |
| address.postcode | string | 12345 | The user's postcode/zipcode. |
| address.country | string | US | The user's country represented by it's ISO-3166 2-letter code. |
| comms\_preferences | object | \- | Optional. |
| comms\_preferences.email | boolean | true | Optional. True if the user has opted-in to email marketing communications. False or not present if the user has opted-out. |
| comms\_preferences.sms | boolean | true | Optional. True if the user has opted-in to SMS marketing communications. False or not present if the user has opted-out. |
| comms\_preferences.phone | boolean | true | Optional. True if the user has opted-in to telephone marketing communications. False or not present if the user has opted-out. |
| comms\_preferences.post | boolean | true | Optional. True if the user has opted-in to post/lettermail marketing communications. False or not present if the user has opted-out. |
| affiliate\_details | object | \- | Optional. |
| affiliate\_details.affiliate\_program\_id | string | adwords | A unique identifier for the affiliate program the user was acquired by. |
| affiliate\_details.affiliate\_campaign\_id | string | ga-campaign-2020-xmas | A unique identifier for the affiliate marketing campaign the user was acquired by. |

### Response

| HTTP Status Code | Description |
| ---| --- |
| 204 | Success, No Content |
| 404 | user not found |
| 409 | email address already in use |

<aside class="success">
The <code>user_id</code> returned should be used in subsequent API calls where a <code>user_id</code> is required as part of the URI.
</aside>

# Sessions

## Create Session

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X POST \
https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/sessions
```

`POST /users/{user_id}/sessions`

Associate a session to a user. A session should be created during any login event or other distinction associated to a session.

### URI Parameters
| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Query Parameters
The request payload is empty.

### Response

> Example JSON Response:

```json
{
  "grant_code": "nsbs-temp-grant-code-1"
}
```

A successful request returns the HTTP `200 OK` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| grant\_code | string | nsbs-temp-grant-code-1 | A temporary authorization grant code. |

<aside class="comment">
The returned <code>grant_code</code> will be used in future phases of the integration.
</aside>


| HTTP Status Code | Description |
| ---| --- |
| `201 Created` | Success
| `404 Not Found` | User not found |

# Rounds

## Create Round

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X POST \
-d '{
{
  "external_round_id": "vendor-round-id-1",
  "game_id": "game-1",
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds
```
`POST /users/{user_id}/rounds`

This method creates a round. A round is a container used for grouping together wagers and payouts that are associated to a user. Any combination of wagers and payouts can be contained within a round, as long as the round has not been closed.

One user may share many wager and payout events as long as the game_id and user_id remain the same.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external_round_id | string | vendor-round-id-1 | Unique identifier representing a group of wager and/or payout events for a user. |
| game_id | string | betable-game-id-1 |  The Betable game\_id as returned from the [Create Game](#create-game) method call. |

### Response

> Example JSON Response:

```json
{
  "round_id": "D0SsdoSIN4FvyslTXnsGDF"
}
```

A successful request returns the HTTP `201 Created` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | A unique identifier representing a round. |

<aside class="success">
This <code>round_id</code> is used in subsequent calls when required as part of the URI.
</aside>

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `external_round_id` was already processed |
| `404 Not Found` | User not found |
| `404 Not Found` | Game not found |
| `409 Conflict` | Round already closed |

## Close Round

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/close
```

`PUT /users/{user_id}/rounds/{round_id}/close`

This method is used for closing a Round. A round can consist of zero or more Wagers, and zero or more Payouts. After a round is closed then no additional Wagers or Payouts can be associated to the Round.

<aside class="warning">
All <code>authorized</code> Wagers and Payouts must be resolved (ie. settled or void) before the Round can be closed.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |

### Query Parameters

No request payload.

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `208 Already Reported` | Success, if the `round_id` was already closed |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `409 Conflict` | Wager is not resolved |
| `409 Conflict` | Payout is not resolved |

# Wagers

## Create Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X POST \
-d '{
  "external_wager_id": "vendor-wager-id-1",
  "amount": "5.00",
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/wagers
```

`POST /users/{user_id}/rounds/{round_id}/wagers`

This method is used to create and authorize a Wager within the supplied round. The player funds required for this transaction will be immediated deducted from the player's account balance.

An authorized Wager needs to be settled using [Settle Wager](#settle-wager) to complete the wager processing.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external\_wager\_id | string | vendor-wager-id-1 | A unique identifier to map a vendor wager request to one in the Betable API. |
| amount | Money | 5.00 | The amount the user has wagered. |


### Response

> Example JSON Response:

```json
{
  "wager_id": "rD9c1IqMhlgLlwTEz0xbB3"
}
```

A successful request returns the HTTP `201 Created` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| wager\_id | string | rD9c1IqMhlgLlwTEz0xbB3 | Unique identifier representing a wager.|

<aside class="success">
This <code>wager_id</code> is used in subsequent calls when required as part of the URI.
</aside>

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `external_wager_id` was already processed |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Insufficient funds |

## Settle Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "context": "additional information to include with the wager"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/wagers/rD9c1IqMhlgLlwTEz0xbB3/settle
```

`PUT /users/{user_id}/rounds/{round_id}/wagers/{wager_id}/settle`

This method settles a Wager that was previously authorized. Typically a Wager is settled immediately after it has been created, but this allows for other processing to be confirmed prior to finalizing the transaction.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| wager\_id | string | rD9c1IqMhlgLlwTEz0xbB3 | The Betable wager\_id as returned from a [Create Wager](#create-wager) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | game:Weekend Bonanza | Optional. Human readable text to associate with the wager.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `wager` was already settled |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Wager not found |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Wager already void |


## Void Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "context": "game error"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/wagers/rD9c1IqMhlgLlwTEz0xbB3/void
```

`PUT /users/{user_id}/rounds/{round_id}/wagers/{wager_id}/void`

This method is used to void (cancel) a previously authorized Wager. Additional contextual information can be provided when voiding the Wager, such as a reason for why the Wager was voided.

<aside class="warning">
A wager can not be voided after it has been settled; instead a Rollback should be performed.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| wager\_id | string | rD9c1IqMhlgLlwTEz0xbB3 | The Betable wager\_id as returned from a [Create Wager](#create-wager) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | game error | Optional. Human readable text to associate with the voided wager.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `wager` was already voided |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Wager not found |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Wager already settled |

## Rollback Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "context": "game error"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/wagers/rD9c1IqMhlgLlwTEz0xbB3/rollback
```

`PUT /users/{user_id}/rounds/{round_id}/wagers/{wager_id}/rollback`

This method is used for rolling back (reversing) a previously settled Wager. This transaction will appear in a player's history.

<aside class="comment">
A rollback is permitted on a round that is closed.
</aside>

<aside class="warning">
Only Wagers that do not have any associated Settled Payouts can be rolled back.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| wager\_id | string | rD9c1IqMhlgLlwTEz0xbB3 | The Betable wager\_id as returned from a [Create Wager](#create-wager) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | game error | Optional. Human readable text to associate with the rolled back Wager.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `wager` was already rolled back |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Wager not found |
| `409 Conflict` | Wager already voided |
| `409 Conflict` | Wager already forfeited |
| `409 Conflict` | Wager has associated payouts; rollback payout first |


# Payouts

## Create Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X POST \
-d '{
  "external_payout_id": "vendor-payout-id-1",
  "source_wager_id": "rD9c1IqMhlgLlwTEz0xbB3",
  "amount": "10.00"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/payouts
```

`POST /users/{user_id}/rounds/{round_id}/payouts`

This method is used for creating and authorizing a Payout such as when awarding winnings. When possible, a Payout should be associated to a Wager.

An authorized Payout needs to be settled using [Settle Payout](#settle-payout) before the funds are transfered to the player's wallet.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external\_payout\_id | string | vendor-wager-id-1 | A unique identifier to map a vendor wager request to one in the Betable API. |
| amount | Money | 10.00 | The amount of funds to transfer to the user's balance. |
| source_wager_id| string | rD9c1IqMhlgLlwTEz0xbB3 | Optional. The Betable `wager\_id` that is associated with this payout. |

<aside class="comment">
The <code>source_wager_id</code> is vital when bonus funds are included in the system for appropriately awarding funds.  It is optional to allow for situations when there is no wager that should be associated.
</aside>

<aside class="warning">
If <code>source_wager_id</code> is present then the associated wager must be settled before this payout can be created.
</aside>

### Response

> Example JSON Response:

```json
{
  "payout_id": "8OJyF5NX1AtIRLMHXHMrqC"
}
```

A successful request returns the HTTP `201 Created` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| payout\_id | string | 8OJyF5NX1AtIRLMHXHMrqC | Unique identifier associated to a payout |


<aside class="success">
This <code>payout_id</code> is used in subsequent calls when required as part of the URI.
</aside>

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `external_payout_id` was already processed |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `409 Conflict` | Round already closed |
| `409 Not Found` | Source wager not found |
| `409 Conflict` | Source wager is not settled |


## Settle Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "context": "Liberty Bell, Liberty Bell, Liberty Bell"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/payouts/8OJyF5NX1AtIRLMHXHMrqC/settle
```

`PUT /users/{user_id}/rounds/{round_id}/payouts/{payout_id}/settle`

This method is used to settle a Payout that was previously authorized. Typically a Payout is settled immediately after it has been created, but this allows for other processing to be confirmed prior to finalizing the transaction.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| payout\_id | string | 8OJyF5NX1AtIRLMHXHMrqC | The Betable payout\_id as returned from a [Create Payout](#create-payout) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | Liberty Bell, Liberty Bell, Liberty Bell | Optional. Human readable text to associate with the payout.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `payout` was already settled |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Payout not found |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Payout already void |

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

### Error Codes

| HTTP Status Code | Error Description |
| ---| --- |
| 404 | user not found |
| 404 | round not found |
| 404 | payout not found |
| 409 | round is closed |
| 409 | payout already void |

## Void Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "context": "game error"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/payouts/8OJyF5NX1AtIRLMHXHMrqC/void
```

`PUT /users/{user_id}/rounds/{round_id}/payouts/{payout_id}/void`

This method is used to void (cancel) a previously authorized Payout. Additional contextual information can be provided when voiding the Payout, such as a reason for why the Payout was voided.

<aside class="warning">
A payout cannot be voided after it has been settled.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| payout\_id | string | 8OJyF5NX1AtIRLMHXHMrqC | The Betable payout\_id as returned from a [Create Payout](#create-payout) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | T+Cs violated | Optional. Human readable text to associate with the voided payout.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `payout` was already voided |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Payout not found |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Payout already settled |

## Rollback Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "context": "incorrect amount awarded"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/payouts/8OJyF5NX1AtIRLMHXHMrqC/rollback
```

`PUT /users/{user_id}/rounds/{round_id}/payouts/{payout_id}/rollback`

This method is used for rolling back (reversing) a previously settled Payout.  This transaction will appear in a player's history.

<aside class="comment">
A rollback is permitted on a round that is closed.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| payout\_id | string | 8OJyF5NX1AtIRLMHXHMrqC | The Betable payout\_id as returned from a [Create Payout](#create-payout) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | incorrect amount awarded | Optional. Human readable text to associate with the rolled back Payout.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `208 Already Reported` | Success, if the `wager` was already rolled back |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Payout not found |
| `409 Conflict` | Payout already voided |
| `409 Conflict` | Payout already forfeited |



# Get Balance

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X GET \
https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/balance
```

`GET /users/{user_id}/balance`

Returns the player's wallet balance. A possible use case for calling this method would be for displaying the player's balance.

<aside class="comment">
This API method is provided for convenience though may not serve a purpose in future phases of integration.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Query Parameters

No request payload.

### Response

> Example JSON Response:

```json
{
  "amount": "100.00",
  "currency": "USD",
}
```

A successful request returns the HTTP `200 OK` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| amount | Money | 100.00 | The user's balance to a maximum of 2 decimal places. |
| currency | string | USD | Currency specified by an ISO-4217 3-letter code. |

### Error Codes

| HTTP Status Code | Error Description |
| ---| --- |
| `200 OK` | Success
| `404 Not Found` | User not found |

## Set Balance

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X PUT \
-d '{
  "amount": "500.00"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/balance
```

`PUT /users/{user_id}/balance`

This method is used to set the user's wallet balance for play money funds.

<aside class="warning">
Set Balance method completely overwrites the players's balance.
</aside>

In order to add funds to the user's wallet (as described in the above example), a Get Balance method call would have to be performed first to retrieve the user's existing balance, followed by a Set Balance method call using the value returned from Get Balance, adding the appropriate amount to the value returned from the user's current balance.

<aside class="warning">
This method _should not_ be used for resolving a Payout. That is, if a player should be awarded winnings, then the [Create Payout](#create-payout) method should be used.
</aside>

<aside class="comment">
This API method is provided for convenience though may not serve a purpose in future phases of integration.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| amount | Money | 500.00 | The user's balance to a maximum of 2 decimal places. Acceptable range between 0 and 99,999.99 inclusive. |

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `404 Not Found` | User not found |

# Create Game

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [vendor-api-key]" \
-X POST \
-d '{
  "external_game_id": "pgp-game-id-1",
  "game_name": "The Numeric Alphabet Game",
  "unencumber_value": "1.00",
}' https://ns-api-staging.betable.com/lfs/v100\
/games
```

`POST /games`

This method associates a game_id with essential information to be used during wallet interactions with Rounds.

<aside class="comment">
One game can be used by multiple players and multiple rounds.
</aside>

<aside class="warning">
A Game must be created before it can be interacted with by a player.
</aside>

### Query Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external\_game\_id | string | pgp-game-id-1 | A unique identifier for the game in the vendor's platform. |
| game\_name | string | The Numeric Alphabet Game | A human readable name representing the game. |
| unencumber\_value | string | 1.00 | Optional. Indicates rate a wager acts to unencumber a bonus requirement.  *Defaults to 1.*

This value is a string that represents any 2-digit decimal number between 0.01 and 1.00 inclusive.|

### Response
> Example JSON Response:

```json
{
  "game_id": "ePz8u6hbncrNzFRCqLbX8e"
}
```

A successful request returns the HTTP `201 Created` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| game\_id | string | ePz8u6hbncrNzFRCqLbX8e | A unique identifier for the game used for URIs and JSON requests when communicating with the NSBS. |

<aside class="success">
This <code>game_id</code> is used in subsequent calls when required as part of the URI.
</aside>

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `409 Conflict` | `external_game_id` was already processed |
