---
title: Unloc API Reference v1.0.1

language_tabs:
  - code

toc_footers:
  - <a href='mailto:august@unloc.ai'>Apply for API key</a>
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

Unloc partners use API keys to get access to the API.

The API key must be included in all API requests to the server in an Authorization header:

`Authorization: Bearer <API KEY>`

<aside class="notice">You must replace <code>API KEY</code> with your own API key.
</aside>

## Invoking the API

Request data must be supplied as a JSON object in the request body. Response data will be returned as a JSON object in the response body.

Timestamps must be in UTC in ISO-8601 format. For example: `2018-01-28T18:00:00Z`

# Locks

## Get All Locks

```code
curl https://api.unloc.app/v1/partners/abcpartner/locks \
-H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe"
```
> The JSON response object contains an array of lock data.

```json
{
    "locks": [
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
}
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
curl -X POST \
-H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe" \
-H 'Content-Type: application/json' \
-d '{"lockId":"danalock-d4:b5:8f:59:47:79","start":"2018-06-01T20:30:00Z","end":"2018-06-01T21:00:00Z","msn":"+4790000000"}' \
https://api.unloc.app/v1/partners/abcpartner/keys
```

> The JSON response is the ID of the newly created key. The ID is a cryptographically secure UUID.

```json
{
  "id" : "117ec32d-5ac3-422b-82de-cbb64540bffd"
}
```
Digital keys are time-limited and assigned to exactly one person via their mobile phone number. All key use is logged in the Unloc backend.

`POST https://api.unloc.app/v1/partners/<Partner ID>/keys`

The request body must contain the following fields:

Field  | Meaning
---------- | -------
lockId | The ID of the lock that the key should be created for
start | When the key should begin to be valid (ISO-8601 datetime)
end | When the key should stop being valid (ISO-8601 datetime). Set `end` to `null` to make a never-expiring key.
msn | Mobile phone number of the recipient of the key, or null if the key is not assigned to a specific person. If `msn` is set to `null`, then the key can only be accessed via the Unloc Work app using App Switch.

## Access Key via App Switch

> Example App Switch URL

```code
ai.unloc.pro://use-key?id=86477029-5db2-4bc4-bdf9-eaf2e9aad759&r=myapp://&n=acbpartner&s=09ffe07e72343ebe456b2e65618f99fb74691e4c0323d0f98dba139d6bca83ba
```

Partner keys can be used on any mobile device that has the Unloc Partner app installed. To use the key, craft a URL with the following format:

`ai.unloc.pro://use-key?id=<key ID>&r=<return URL>&n=<partner id>&s=<HMAC>`

Tapping the URL on a mobile device will bring up the Unloc Work app and make the key available for use. Dismissing the key screen in the Unloc Work app will bring the user back to the originating app, via the `return URL`.

The App Switch URL has the following query parameters:

Parameter | Meaning
------ | ------
id | The key ID
r | The return URI that the Unloc app will open when the key screen is dismissed
n | The partner ID
s | HMAC-SHA256 signature

The Unloc Work app is available for Android on Google Play and iOS devices on the AppStore.
<div class="store-row" style="display: flex; padding: 0 0 0 20px;">
  <div class="store-column">
    <a href='https://play.google.com/store/apps/details?id=ai.unloc.pro&utm_source=devdocs&pcampaignid=MKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'><img width="150" alt='Get it on Google Play' style="flex: 33.33%;
  padding: 5px;" src='https://www.vr.fi/cs/vr/img/d/1/1/googleplay_EN.png?blobnocache=false'/></a>
  </div>
  <div class="store-column">
    <a href='https://itunes.apple.com/no/app/unloc-work/id1410680282?mt=8'><img width="150" alt='Download on the AppStore' style="flex: 33.33%;
  padding: 5px;" src='https://simplematters.zendesk.com/hc/en-us/article_attachments/202646338/App_Store_icon_1.png'/></a>
  </div>
</div>

https://itunes.apple.com/no/app/unloc-work/id1410680282?mt=8

### HMAC

```node
// node.js example
// generate app switch url with hmac signature

const crypto = require('crypto');
const hmacSecret = 'secret-123';
const hmac = crypto.createHmac('sha256', hmacSecret);
const returnUri = 'myapp://';
const partnerId = 'partner-x';

const keyId = '117ec32d-5ac3-422b-82de-cbb64540bffd';
console.log(appSwitchUrl(keyId));

function appSwitchUrl(keyId) {
    const params = `id=${keyId}&r=${returnUri}&n=${partnerId}`;
    hmac.update(params);
    const s = hmac.digest('hex');
    return `ai.unloc.pro://use-key?${params}&s=${s}`;
}
```
```ruby
# ruby example
# generate app switch url with hmac signature

require 'openssl'

hmac_secret = 'secret-123'
return_uri = 'myapp://'
partner_id = 'partner-x'
key_id = '117ec32d-5ac3-422b-82de-cbb64540bffd'

def app_switch_url(key_id)
    params = "id=#{key_id}&r=#{return_uri}&n=#{partner_id}"
    s = OpenSSL::HMAC.hexdigest('SHA256', hmac_secret, params)
    return "ai.unloc.pro://use-key?#{params}&s=#{s}"
end
```

The URL is authenticated with an HMAC-SHA256 signature. The signature is produced with the partner's HMAC secret.

<aside class="notice">Partners receive their HMAC secret when they sign up with Unloc.</aside>

The HMAC is generated from the query parameters as follows:

`HMAC_SHA256(hmacSecret, "id=<Key ID>&r=<return URI>&n=<partner ID>")`

## Get key details

```code
curl -H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe" \
https://api.unloc.app/v1/partners/abcpartner/keys/117ec32d-5ac3-422b-82de-cbb64540bffd

```
> The JSON response is the key with usage data (events).

```json
{
    "id": "117ec32d-5ac3-422b-82de-cbb64540bffd",
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

Partners can request key details. Details include the current state of the key, and an array of key usage events.

Keys can be in either of the following states.

State | Meaning
------ | ------
creating | The key is being created
scheduled | The key will become active (at the `start` time)
active | The key is currently active
expired | The key is not longer active (it expired at the `end` time)
error | The key is in an error state
revoked | The key has been revoked (by its creator or by the lock owner)

## Revoke a Key

```code
curl -X DELETE \
-H "Authorization: Bearer rGUM658NwnnwrT4xZXVQGia3o2pQJwYe" \
https://api.unloc.app/v1/partners/abcpartner/keys/117ec32d-5ac3-422b-82de-cbb64540bffd
```

> JSON response:

```json
{
  "response": "OK"
}
```

Keys can be revoked. Revoked keys become immediately unusable. Note that revoked keys are not deleted from the database, they only get their `state` changed to `revoked`.
