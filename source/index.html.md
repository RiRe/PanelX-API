---
title: PanelX API - sourceWAY GmbH

language_tabs:
  - php

toc_footers:
  - <a href='https://sourceway.de/imprint' target='_blank'>Imprint</a>

search: true
---

# Introduction

PanelX has an API available to be integrated in other software solutions. This documentation covers all available API actions, from authentication to endpoints. PanelX API is a RESTful API.

We use `https://panelx.de` as base URL, this is the usual case.

We provide example codes in PHP. You can view code examples in the dark area to the right.

# Return values & error handling

> Successful request without data

```json
{
  "status": true,
  "code": 1000,
  "msg": "Request successful",
  "data": {}
}
```

> Failed request

```json
{
  "status": false,
  "code": 1200,
  "msg": "An error occured.",
  "data": {}
}
```

PanelX API returns data in JSON format.

If you manage to send a request to a valid URL with a valid HTTP method, you will get everytime a HTTP status code `200`, independently of the actions result.

Each response has four root-elements:

`status` is a boolean describing if the call was successful and the action was performed correctly.

`code` is an error code, which can be useful for the developers to debug problems. In case of a successful call, the code is `1000` always.

`msg` is a message explaining the `code` returned. It can give more information about an error, so the message may not be equal for the same error code of a method.

`data` contains information that is returned by the API for your request. That may be e.g. IDs of created objects or a list of results that you have requested.

# Authentication

Every request sent to PanelX API requires authentication. That is done via HTTP Basic Auth, using the requesting users email address and password.

The HTTP Basic Auth header should be built and sent like this:

`Authentication: Basic base64_encode(email:password)`

The PanelX API is only available for administrators and resellers.

There are some global failure codes regarding authentication:

Failure Code | Meaning
---------- | -------
1500 | Invalid authentication header.
1501 | User authentication failed.
1502 | API usage not allowed (not an administrator or reseller)

# Create user

```php
<?php
$req = [
  "name" => "John Doe",
  "email" => "john.doe@example.com",
  "password" => "Secret123!",
];

$ch = curl_init("https://panelx.de/api/users");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($req));
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "User created",
  "data": {
    "id": 123
  }
}
```

`POST /api/users`

This endpoint can be used to create a new user.

## Query parameters

Parameter | Description
--------- | -----------
name | **Required** Full name
email | **Required** Email address
password | **Required** Password (min. 8 characters)

## Return values

Parameter | Description
--------- | -----------
id | ID of the newly created user

## Failure codes

Failure Code | Meaning
---------- | -------
1200 | No name specified.
1201 | No or invalid email specified.
1202 | Email already exists.
1203 | Password is too short (8 characters minimum).

# Modify user

```php
<?php
$req = [
  "status" => false,
];

$ch = curl_init("https://panelx.de/api/users/123");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($req));
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "User updated",
  "data": {}
}
```

`POST /api/users/USER_ID`

This endpoint can be used to modify an existing user. Currently, PanelX API supports modification of user status (active / locked) and password.

## Query parameters

Parameter | Description
--------- | -----------
status | If set, interpreted as boolean for user status (`true` = active, `false` = locked)
password | If set, new password for user

## Return values

None

## Failure codes

Failure Code | Meaning
---------- | -------
1200 | User not found.
1201 | Password is too short (8 characters minimum).

# Add contract

```php
<?php
$req = [
  "name" => "web1",
  "client" => 123,
  "server_id" => 1,
  "template" => 1,
];

$ch = curl_init("https://panelx.de/api/contracts");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($req));
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "Contract created",
  "data": {
    "id": 10
  }
}
```

`POST /api/contracts`

This endpoint can be used to create a new contract for an user you own.

## Query parameters

Parameter | Description
--------- | -----------
name | **Required** Name for the contract (min. 3 characters)
client | **Required** User ID
server_id | **Required** Server ID (parameter name has technical reasons)
template | **Required** Template ID

## Return values

Parameter | Description
--------- | -----------
id | ID of the newly created contract

## Failure codes

Failure Code | Meaning
---------- | -------
1200 | No contract name specified.
1201 | Specified contract name too short (3 characters minimum).
1202 | Contract name already exists.
1203 | No/invalid customer specified.
1204 | No/invalid server specified.
1205 | No/invalid template specified.
1206 | Internal error (see `msg`)

# Modify contract

```php
<?php
$req = [
  "template" => 2,
];

$ch = curl_init("https://panelx.de/api/contracts/10");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($req));
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "Contract updated",
  "data": {}
}
```

`POST /api/contracts/CONTRACT_ID`

This endpoint can be used to edit an existing contract.

## Query parameters

Parameter | Description
--------- | -----------
template | **Required** New template ID

## Return values

None

## Failure codes

Failure Code | Meaning
---------- | -------
1200 | Contract not found.
1201 | No/invalid template specified.

# Delete contract

```php
<?php
$ch = curl_init("https://panelx.de/api/contracts/10");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "DELETE");
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "Contract deleted",
  "data": {}
}
```

`DELETE /api/contracts/CONTRACT_ID`

This endpoint can be used to delete an existing contract.

## Query parameters

None

## Return values

None

## Failure codes

Failure Code | Meaning
---------- | -------
1200 | Contract not found.

# List templates

```php
<?php
$ch = curl_init("https://panelx.de/api/templates");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "Templates fetched",
  "data": [
    {
      "id": 1,
      "name": "Default template"
    }
  ]
}
```

`GET /api/templates`

This endpoint can be used to fetch all templates existing for your user.

## Query parameters

None

## Return values

List (array) of templates existing, containing template `id` and `name`.

## Failure codes

None

# List servers

```php
<?php
$ch = curl_init("https://panelx.de/api/server");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_USERPWD, "admin@panelx.de:test1234");
$res = curl_exec($ch);

if (curl_errno($ch)) {
  die(curl_error($ch));
}

curl_close($ch);
$res = json_decode($res, true);

print_r($res);
```

> The above request returns JSON structured like this:

```json
{
  "status": true,
  "code": 1000,
  "msg": "Servers fetched",
  "data": [
    {
      "id": 1,
      "hostname": "localhost"
    }
  ]
}
```

`GET /api/servers`

This endpoint can be used to fetch all servers useable by your user.

## Query parameters

None

## Return values

List (array) of servers existing, containing server `id` and `hostname`.

## Failure codes

None