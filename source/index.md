---
title: API Reference

language_tabs:
  - code

toc_footers:
  - <a href='mailto:august@unloc.ai'>Apply for authorization token</a>
  - <a href='https://unloc.no'>unloc.no</a>

includes:
  - errors

search: true
---

# Unloc API Documentation

Unloc is a system that enables key sharing and in-home deliveries of goods and services.

Unloc creates time-limited digital keys, which are forwarded to electronic locks installed in end-users’ homes. The digital keys can be granted to merchants and service providers (partners).

The Unloc API allows partners to create digital keys programmatically.

## Authentication

Unloc uses authorization tokens to allow access to the API. An authorization token represents you as an Unloc partner.

The authorization token must be included in all API requests to the server in an Authorization header:

`Authorization: Bearer TOKEN`

<aside class="notice">You must replace <code>TOKEN</code> with your own authorization tokens.
</aside>

## Invoking the API

Request data must be supplied as a JSON object in the request body. Response data will be returned as a JSON object in the response body.

Timestamps must be in UTC in ISO-8601 format. For example: `2018-01-28T18:00:00Z`

# Locks

## Get All Locks

```code
curl https://api.unloc.app/v1/partners/abcpartner/locks -H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe"
```
> The JSON response is an array of lock data.

```json
[
    {
        "auth": "opt-out",
        "data": "customer-001",
        "id": "danalock-00:00:00:00:00:01",
        "name": "Hjemme"
    },
    {
        "auth": "opt-out",
        "data": "customer-001",
        "id": "doora-00:00:00:00:00:01",
        "name": "Hovedinngang"
    }
]
```

`GET https://api.unloc.app/v1/partners/<Partner ID>/locks`

Get all locks that you can create keys for. Lock owners can selectively authorize partners to issue digital keys for their locks by selecting either `opt-in`, `opt-out`, or `deny` in the Unloc app.

The response lock data contains the following fields:

Field  | Meaning
---------- | -------
id | The ID of the lock
name | The owner-assigned name of the lock
auth | Either `opt-in` or`opt-out`
data | Partner-specific (your) data associated with this lock

If a lock is marked as `opt-in`, then you need to be aware that the key cannot be used unless the lock owner has accepted the key in the Unloc app.

The `data` field contains data that is only visible to you as a partner. For example, it can be useful to associate a customer ID or similar to the lock so that you can recognize your customers' locks.


# Keys

## Create a Key

```code
curl -X POST -H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe" -H 'Content-Type: application/json' -d '{"lockId":"danalock-d4:b5:8f:59:47:79","start":"2018-06-01T20:30:00Z","end":"2018-06-01T21:00:00Z","msn":"+4790000000"}' https://api.unloc.app/v1/partners/abcpartner/keys
````

> The JSON response is the ID of the newly created key.

```json
{
  "id" : "466b1350abc-abcdef-danalock-d4:b5:8f:59:47:79"
}
```
Digital keys are time-limited and assigned to exactly one person via their mobile phone number. All key use is logged in the Unloc backend.

Partners can choose to assign a secret token to a key. The partner can then later use the secret token to access the key anonymously via a URL, using Unloc App Switch.

`POST https://api.unloc.app/v1/partners/<Partner ID>/keys`

The request body must include the following fields:

Field  | Meaning
---------- | -------
lockId | The ID of the lock that the key should be created for
start | When the key should begin to be valid (ISO-8601 datetime)
end | When the key should stop being valid (ISO-8601 datetime)
msn | Mobile phone number of the recipient of the key
secretToken | A secret token (at least 32 characters) that the key can be access with later

Note that `msn` and `secretToken` are mutually exclusive.

## Access Key By Secret Token

A key that has been created with a `secretToken` can be used on any mobile phone that has the Unloc PRO app installed. To use the key, craft a URL with the following format:

`ai.unloc.pro://use-key?t=<secretToken>&r=<return_uri>&n=<partner name>`

Tapping the URL will bring up the Unloc PRO app and make the key available for use. Dismissing key screen in the Unloc PRO app will bring the user back to the originating app.

The URL has the following query parameters:

Parameter | Meaning
------ | ------
t | The secretToken used to create the key
r | The return URI that the Unloc app will open when the key screen is dismissed
n | The display name of the partner (will be displayed to the l in the Unloc app)

## Revoke a Key

```code
curl -X DELETE -H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe" https://api.unloc.app/v1/keys/466b1350abc-abcdef-danalock-d4:b5:8f:59:47:79```

> JSON response:

```json
{
  "response": "OK"
}
```

Keys can be revoked. Revoked keys become immediately unusable.

## Get key details

```code
curl https://api.unloc.app/v1/partners/abcpartner/key/abc-1234 -H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe"
```
> The JSON response is the key with usage data (events).

```json
{
    "id": "4656efbe41-helthjem-danalock-d4:b5:8f:59:47:79",
    "lockId": "danalock-d4:b5:8f:59:47:79",
    "state": "revoked",
    "start": "2018-06-05T11:00:00.000Z",
    "end": "2018-06-10T17:00:00.000Z",
    "events": [
        {
            "action": "locked",
            "created": "2018-06-05T11:36:49.235Z",
            "msn": "+4792443609",
            "name": "August Flatby"
        },
        {
            "action": "unlocked",
            "created": "2018-06-05T12:37:47.024Z",
            "msn": "",
            "name": "Partner X"
        },
    ]
}

```

`GET https://api.unloc.app/v1/partners/<Partner ID>/locks`