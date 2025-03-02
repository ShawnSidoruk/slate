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

The Betable API is a RESTful service providing the ability to create a compliant gaming solution with integrated wallet, CRM, fraud, and payment facilities.

## Change History

Version | Date | Summary of Changes
--------- | ------- | -----------
1.0 | 2020-03-23 | Initial release of Betable API.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## Quick Reference

All API access is over HTTPS and must be authenticated or a `401 Unauthorized error` will be returned (see [Authentication](#authentication)).

A `User` corresponds to each unique player in the partner system and contains profile and preference information.

A `Session` is created for every login event by a player.

A `Round` is a container for gaming interactions which typically consists of a series of wager and/or payout actions.

A `Wager` is any entry or bet placed by a player.

A `Payout` represents an amount won by a player.


# API Overview

## API Domains

On staging, all API endpoints begin with `https://ns-api-staging.betable.com/lfs/v100`.

On production, all API endpoints begin with `https://ns-api.betable.com/lfs/v100`.

## IP Whitelisting
The Betable API exposed in this document is not a public API.  In addition to other security measures, only a list of static IP Address(es) will be permitted to communicate on this API.

<aside class="warning">
Any API method calls made from a non-whitelisted IP Address will be blocked.
</aside>

## Authentication

```shell
curl \
-H "X-API-Key: [partner-api-key]" \
...
```

`X-API-Key: [partner-api-key]`

An API key will be provided to the integration partner which is to be used in all requests to the Betable API unless stated. The API key must be kept secret and should only be used when making requests to the Betable API as a form of authentication.

<aside class="notice">
A separate API key will be provided for both staging and production environments.
</aside>
<aside class="warning">
A missing or invalid API key will produce a <code>401 Unauthorized</code> error.
</aside>

## Content Type

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
...
```

`Content-Type: application/json`

The Betable API is implemented using a REST-style service, using a JSON content type for both the request and response payloads.

## Request Tracing (Optional)

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-H "X-Trace-ID: [partner-generated-trace-id]" \
...
```

`X-Trace-ID: [partner-generated-trace-id]`

Although not required, all requests are encouraged to include a unique identifer in the header. Tracing is not used in operational integrity but can be used as a reference in post-processing as the header will be reflected on the response and can be used to correlate a request and response. The `X-Trace-ID` reference is only stored for for 14 days.

<aside class="notice">
The <code>X-Trace-ID</code> should also be provided, where possible, for support requests.
</aside>

## Idempotent Requests (Optional)

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-H "X-Idempotency-Key: [partner-generated-key]" \
...
```

`X-Idempotency-Key: [partner-generated-key]`

The API supports idempotency for safely retrying requests without performing the same operation twice. A unique identifier added to the header on `POST` or `PUT` requests ensures that the same request is not processed multiple times. If a duplicate request is sent, the original response will be returned without any processing. Only successfully processed requests will have their responses stored for idempotency, thus when retrying the same request a new idempotency key does not need to be generated.

<aside class="notice">
The Betable API will store the idempotency keys and responses for 30 days.
</aside>

## Response Codes and Statuses

### HTTP Statuses

HTTP Status Code | Description
---- | -----------
`200 OK` | Request processed, response contains details.
`201 Created` | Request processed, entity created, response contains details.
`204 No Content` | Request processed, no response details.
`208 Already Reported` | Request previously processed successfully, response contains details from original request.
`400 Bad Request` | Validation error.
`401 Unauthorized` | Unauthorized, see section: Authentication
`404 Not Found` | Requested entity not found.
`409 Conflict` | Request not processable due to business logic, entity state, or other reason(s).
`500 Internal Error` | Unexpected runtime error.  May try again.

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
error\_description | string | A description of the error encountered; see [common errors](#common-error-responses) below or each API method section.

## Common Data Structures

### JSON Data Types
The Betable API uses the standard JSON data types for all fields of the request and response payloads with the addition of these custom data types:

Data Type | JSON Type | Description | Example Value
--------- | ------- | ----------- | ------
`Date` | string | Using the following format: yyyy-MM-dd | 1969-04-20
`Money` | string | All monetary amounts are represented as a string. | 199.99


### Common Error Responses

Each API method can return a number of common error responses:

HTTP Status Code | Error Description
---- | -----------
`400 Bad Request` | Validation error.
`401 Unauthorized` | Unauthorized, see section: [Authentication](#authentication)
`500 Internal Error` | Unexpected runtime error.  May try again.

# Health Check

## Health

`GET /health`

```shell
curl \
-H "Content-Type: application/json" \
https://ns-api-staging.betable.com/lfs/v100/health
```

This method is used to ensure communication with the Betable API is functional. The main usage of this endpoint is to assure that the IP Addresses are properly whitelisted.

<aside class="notice">
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
| alive | boolean | true | True if the Betable API is operational and ready for communication. |



# Users

## Create User

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X POST \
-d '{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@doe.com",
  "external_user_id": "partner-user-id-1",
  "date_of_birth": "1969-04-20",
  "mobile_phone": "18881112223",
  "address": {
    "address1": "Unit 7",
    "address2": "Cornell Tower",
    "address3": "Gladwell Street",
    "postcode": "12345",
    "city": "Springfield",
    "state": "IL",
    "country": "US"
  },
  "comms_preferences": {
    "email": true,
    "sms": true,
    "phone": false,
    "post": false
  },
  "affiliate_details": {
    {
      "affiliate_program_id": "adwords",
      "affiliate_campaign_id": "ga-campaign-2020-xmas"
    }
  }
}' https://ns-api-staging.betable.com/lfs/v100/users
```
`POST /users`

This method is used to create a new user in the Betable API and should be called at the same time that a new user is created in the partner's platform.

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| first\_name | string | John | The user's first name. |
| last\_name | string | Doe | The user's last name. |
| email | string | john@doe.com | The user's email. |
| external\_user\_id | string | partner-user-id-1 | A unique ID that can be used to identify the user's account in the partner platform. |
| date\_of\_birth | Date | 1969-04-20 | Optional. The user's date of birth. |
| mobile\_phone | string | 18881112223 | Optional. The user's mobile phone number. |
| address | object | \- | Optional. |
| address.address1 | string | Unit 7 | The user's address. Note that formatting of the user's address and the use of the subsequent address fields may vary by region. |
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

<aside class="success">
This <code>user_id</code> is used in subsequent calls when required as part of the URI.
</aside>

| HTTP Status Code | Error Description |
| ---| --- |
| `201 Created` | Success
| `409 Conflict` | Email address already in use |
| `409 Conflict` | `external_user_id` already in use |

## Update User

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
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
    "country": "US"
  },
  "comms_preferences": {
    "email": true,
    "sms": true,
    "phone": false,
    "post": false
  },
  "affiliate_details": {
    {
      "affiliate_program_id": "adwords",
      "affiliate_campaign_id": "ga-campaign-2020-xmas"
    }
  }
}' https://ns-api-staging.betable.com/lfs/v100/users
```

`PUT /users/{user_id}`

This method updates a user's account information.

### URI Parameters
| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| first\_name | string | John | Optional. The user's first name. |
| last\_name | string | Doe | Optional. The user's last name. |
| email | string | john@doe.com | Optional. The user's email. |
| date\_of\_birth | Date | 1969-04-20 | Optional. The user's date of birth. |
| mobile\_phone | string | 18881112223 | Optional. The user's mobile phone number. |
| address | object | \- | Optional. |
| address.address1 | string | Unit 7 | The user's address. Note that formatting of the user's address and the use of the subsequent address fields may vary by region. |
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

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Description |
| ---| --- |
| `204 No Content` | Success, No Content |
| `404 Not Found` | User not found |
| `409 Conflict` | Email address already in use |

# Sessions

## Create Session

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X POST \
https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/sessions
```

`POST /users/{user_id}/sessions`

This method associates a session to a user. A session should be created during any login event or other distinction associated to a session.

### URI Parameters
| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Request Parameters
The request payload is empty.

### Response

> Example JSON Response:

```json
{
  "grant_code": "cmY2xpZSUQ6Y2W50xpZW50U2VjV0"
}
```

A successful request returns the HTTP `201 Created` status code with the following JSON response body:

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| grant\_code | string | cmY2xpZSUQ6Y2W50xpZW50U2VjV0 | A temporary authorization grant code. |

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
-H "X-API-Key: [partner-api-key]" \
-X POST \
-d '{
  "external_round_id": "partner-round-id-1",
  "external_game_id": "ePz8u6hbncrNzFRCqLbX8e",
  "external_game_label": "Starburst"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds
```
`POST /users/{user_id}/rounds`

This method creates a round. A round is a container used for grouping together wagers and payouts that are associated to a user. Any combination of wagers and payouts can be contained within a round, as long as the round has not been closed.

A round can consist of zero or more Wagers, and zero or more Payouts. The choice of how many Wagers and Payouts to include in a round is up to the integration partner.

<aside>
Wagers and Payouts will be presented back to the player using the <code>external_game_label</code> provided for the Round.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external\_round_id | string | partner-round-id-1 | Unique identifier representing a group of wager and/or payout events for a user. |
| external_game_id | string | partner-game-id |  Identifies the game generating the wager and/or payout events in the round. |
| external_game_label | string | Starburst |  Human readable value presented to the player when accessing transaction history. |

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
| `404 Not Found` | User not found |
| `409 Conflict` | No game exists for `game_id` |
| `409 Conflict` | Round already exists with `external_round_id` |

## Close Round

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X PUT \
https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/close
```

`PUT /users/{user_id}/rounds/{round_id}/close`

This method is used for closing a Round. A round can consist of zero or more Wagers, and zero or more Payouts. Once a round is closed, no additional Wagers or Payouts can be associated to it.

<aside class="warning">
All Wagers and Payouts must be resolved (ie. settled or void) before the Round can be closed.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |

### Request Parameters

No request payload.

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `409 Conflict` | Round was already closed |
| `409 Conflict` | Wager is not resolved |
| `409 Conflict` | Payout is not resolved |

# Wagers

## Create Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X POST \
-d '{
  "external_wager_id": "partner-wager-id-1",
  "amount": "5.00"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/wagers
```

`POST /users/{user_id}/rounds/{round_id}/wagers`

This method is used to create and authorize a Wager within the supplied round. The player funds required for this transaction will be immediately deducted from the player's account balance.

An authorized Wager needs to be settled using [Settle Wager](#settle-wager) to complete the wager processing.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external\_wager\_id | string | partner-wager-id-1 | A unique identifier to map a partner wager request to one in the Betable API. |
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
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `409 Conflict` | Wager with `external_wager_id` was already created |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Insufficient funds |

## Settle Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
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

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | game:Weekend Bonanza | Optional. Human readable text to associate with the wager.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Wager not found |
| `409 Conflict` | Wager was already settled |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Wager already void |


## Void Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X PUT \
-d '{
  "context": "game error"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/wagers/rD9c1IqMhlgLlwTEz0xbB3/void
```

`PUT /users/{user_id}/rounds/{round_id}/wagers/{wager_id}/void`

This method is used to void (cancel) a previously authorized Wager. Additional contextual information can be provided when voiding the Wager, such as a reason for why the Wager was voided.

<aside class="warning">
A wager cannot be voided after it has been settled; instead a Rollback should be performed.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| wager\_id | string | rD9c1IqMhlgLlwTEz0xbB3 | The Betable wager\_id as returned from a [Create Wager](#create-wager) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | game error | Optional. Human readable text to associate with the voided wager.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Wager not found |
| `409 Conflict` | Wager was already voided |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Wager already settled |

## Rollback Wager

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
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
Only Wagers that do not have any associated settled Payouts can be rolled back.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |
| wager\_id | string | rD9c1IqMhlgLlwTEz0xbB3 | The Betable wager\_id as returned from a [Create Wager](#create-wager) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | game error | Optional. Human readable text to associate with the rolled back Wager.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Wager not found |
| `409 Conflict` | Wager was already rolled back |
| `409 Conflict` | Wager already voided |
| `409 Conflict` | Wager has associated payouts; rollback payout first |


# Payouts

## Create Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X POST \
-d '{
  "external_payout_id": "partner-payout-id-1",
  "source_wager_id": "rD9c1IqMhlgLlwTEz0xbB3",
  "amount": "10.00"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/rounds/D0SsdoSIN4FvyslTXnsGDF/payouts
```

`POST /users/{user_id}/rounds/{round_id}/payouts`

This method is used for creating and authorizing a Payout, such as when awarding winnings to a player. When possible, a Payout should be associated to a Wager.

An authorized Payout needs to be settled using [Settle Payout](#settle-payout) before the funds are transfered to the player's wallet.

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |
| round\_id | string | D0SsdoSIN4FvyslTXnsGDF | The Betable round\_id as returned from a [Create Round](#create-round) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| external\_payout\_id | string | partner-payout-id-1 | A unique identifier to map a partner payout request to one in the Betable API. |
| amount | Money | 10.00 | The amount of funds to transfer to the user's balance. |
| source_wager_id| string | rD9c1IqMhlgLlwTEz0xbB3 | Optional. The Betable `wager_id` that is associated with this payout. |

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
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `409 Conflict` | Payout with `external_payout_id` already created |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Source wager not found |
| `409 Conflict` | Source wager is not settled |


## Settle Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
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

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | Liberty Bell, Liberty Bell, Liberty Bell | Optional. Human readable text to associate with the payout.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Payout not found |
| `409 Conflict` | Payout was already settled |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Payout already void |

## Void Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
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

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | T+Cs violated | Optional. Human readable text to associate with the voided payout.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Payout not found |
| `409 Conflict` | Payout already voided |
| `409 Conflict` | Round already closed |
| `409 Conflict` | Payout already settled |

## Rollback Payout

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
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

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| context | string | incorrect amount awarded | Optional. Human readable text to associate with the rolled back Payout.|

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success |
| `404 Not Found` | User not found |
| `404 Not Found` | Round not found |
| `404 Not Found` | Payout not found |
| `409 Conflict` | Payout already rolled back |
| `409 Conflict` | Payout already voided |

# Balances

## Get Balance

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X GET \
https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/balance
```

`GET /users/{user_id}/balance`

Returns the player's wallet balance. A possible use case for calling this method would be for displaying the player's balance.

<aside class="notice">
This API method is provided for convenience though may not serve a purpose in future phases of integration.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

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
| currency | string | USD | Currency.  When relevant this will reference ISO-4217 3-letter code. |


<aside class="comment">
Currency code <code>XXX</code> will be used to represent simulated currencies.
</aside>

<aside class="comment">
Possible currencies returned are setup within the Betable API during the initial phases of partner integration
</aside>

| HTTP Status Code | Error Description |
| ---| --- |
| `200 OK` | Success
| `404 Not Found` | User not found |

## Set Balance

```shell
curl \
-H "Content-Type: application/json" \
-H "X-API-Key: [partner-api-key]" \
-X PUT \
-d '{
  "amount": "500.00"
}' https://ns-api-staging.betable.com/lfs/v100\
/users/QEaakM0qumTJd5tj/balance
```

`PUT /users/{user_id}/balance`

This method is used to set the user's wallet balance for play money funds.

<aside class="warning">
Set Balance method completely overwrites the player's balance.
</aside>

In order to add funds to the user's wallet (instead of overwriting the player's balance), a Get Balance method call would have to be performed first to retrieve the user's existing balance, followed by a Set Balance method call using the value returned from Get Balance, adding the appropriate amount to the value returned from the user's current balance.

<aside class="warning">
This method <b><i>should not</i></b> be used for resolving a Payout. That is, if a player should be awarded winnings, then the <a href="#create-payout">Create Payout</a> method should be used.
</aside>

<aside class="notice">
This API method is provided for convenience though may not serve a purpose in future phases of integration.
</aside>

### URI Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| user\_id | string | QEaakM0qumTJd5tj | The Betable user\_id as returned from the [Create User](#create-user) method call. |

### Request Parameters

| Parameter | Type | Example | Description |
| ---| ---| ---| --- |
| amount | Money | 500.00 | The user's balance to a maximum of 2 decimal places. Acceptable range between 0 and 99,999.99 inclusive. |

### Response

A successful request returns the HTTP `204 No Content` status code with no JSON response body.

| HTTP Status Code | Error Description |
| ---| --- |
| `204 No Content` | Success
| `404 Not Found` | User not found |
