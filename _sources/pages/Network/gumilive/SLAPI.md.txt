# SLAPI
This server is responsable for account login and billing checks.

This is a newer version of the original Live API server, which
seems to be made to support other games than BF as well.

**URL**: https://api-sl.gl.gumi.sg/
**Decomp**: This code is available in the class GumiLiveNetworkManagement

The client will send a custom header to any of this request which is constructed by the following:
`GCLIENTID: (PlatformName)_(AppId)`

for example:
`GCLIENTID: Android_sg.gumi.bravefrontier`

## Accounts
This API is responsable for login and creating new accounts, also manages Facebook login.

This API contains a bunch of generic PHP arguments listed here:

| Name | Description | Example |
| ---- | ----------- | ------- |
| dn | Device model | GT-I9505 |
| dp | Device platform | android |
| vid | Device virtual id | uuid |
| v | System version | 9 |
| device_id | Device ID | id |
| altvid | Device alternative vid | uuid |
| identifiers | Custom data specific to the platform | {"a": "b"} |
| ak | Game API key | 0839899613932562 |

### Request: /accounts/signup/
**Type**: GET

This request tries to register a new user to the API server.

Custom PHP arguments:
| Name | Description | Example |
| ---- | ----------- | ------- |
| username | Username | arves100 |
| password | Password | 0000 |

Response:
```
Unknown...
```

### Request: /accounts/guest/login/
**Type**: GET

This request tries to login a normal user to the API server.

Response:
```
{
    "status": "string",
    "game_user_id", "string",
    "token": "string",
    "status_no": int,
    "servers": {}
}
```

| Name | Description | Optional | Possible values |
| ---- | ----------- | -------- | ------ |
| status | If the request was a success or not | No | "successful" or "error" |
| game_user_id | Player user ID | No | a string |
| token | Token that can be used for identify the user | No | a string |
| status_no | ID of the error (if any) | No | See the list of possible status_no |
| servers | Unknown | No | Empty json array ({}) |

List of possible status ID:
| ID | Description |
| -- | ---------- |
| 0 | Success |
| 1 | Generic error |
| 15 | Invalid "ak" (API Key) |


### Request: /accounts/facebook/login/
**Type**: GET

This request tries to do a Facebook login to the API server.

Response:
```
Same as /guest/login/, refeer to that response
```
