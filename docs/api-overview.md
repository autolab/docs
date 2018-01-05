## Overview

The Autolab REST API allows developers to create clients that can access features of Autolab on behalf of Autolab users.

V1 of the API allows clients to:

* Access basic user info
* View courses and assessments
* Submit to assessments
* View scores and feedback

All clients are required to be registered with the admin of your specific Autolab deployment in order to obtain a unique client_id and client_secret pair.

## Authorization

All endpoints of the Autolab API requires client authentication in the form of an access token. To obtain this access token, clients must obtain authorization from the user.

Autolab API uses the standard [OAuth2](https://tools.ietf.org/html/rfc6749) [Authorization Code Grant](https://tools.ietf.org/html/rfc6749#section-4.1) for user authorization. For clients with no easy access to web browsers (e.g. console apps), an alternative [device flow](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-07)-based authorization method is provided as well.

### Authorization Code Grant Flow

The authorization code grant consists of 5 basic steps:

1. Client directs the user to the authorization request endpoint via a web browser.
2. Authorization server (Autolab) authenticates the user.
3. If user grants access to the client, the authorization server provides an "authorization code" to the client.
4. Client exchanges the authorization code for an access token from the access token endpoint.
5. Client uses the access token for subsequent requests to the API.

The endpoint for obtaining user authorization is
`/oauth/authorize`

The endpoint for obtaning access tokens and refresh tokens is
`oauth/token`

[Section 4.1 of RFC 6749](https://tools.ietf.org/html/rfc6749#section-4.1) details the parameters required and the response clients can expect from these endpoints.

Autolab API provides a refresh token with every new access token. Once the access token has expired, the client can use the refresh token to obtain a new access token, refresh token pair. Details are also provided in RFC 6749 [here](https://tools.ietf.org/html/rfc6749#section-6).

### Device Flow

For devices that cannot use a web browser to obtain user authorization, the alternative device flow approach circumvents the first 3 steps in the authorization code grant flow. Instead of directing a user to the authorization page directly, the client obtains a user code that the user can enter on the Autolab website from any device. The website then takes the user through the authorization procedure, and returns the authorization code to the client. The client can then use this code to request an access token from the access token endpoint as usual.

Note that this is different from the "device flow" described in the Internet Draft linked above.

#### Obtaining User Code

Request Endpoint: `GET /device_flow_init`

Parameters:

* client_id: the client_id obtained when registering the client

Success Response:

* device_code: the verification code used by the client (should be kept secret from the user).
* user_code: the verification code that should be displayed to the user.
* verification_uri: the verification uri that the user should use to authorize the client. By default is `/activate`

The latter two should be displayed to the user.

#### Obtaining Authorization Code

After asking the user to enter the user code on the verification site, the client should poll the device_flow_authorize endpoint to find out if the user has completed the authorization step.

Request Endpoint: `GET /device_flow_authorize`

Parameters:

* client_id: the client_id obtained when registering the client
* device_code: the device_code obtained from the device_flow_init endpoint

Failure Responses:

* 400 Bad Request: {error: authorization_pending}<br>
  The user has not yet granted or denied the authorization request. Please try again in a while.
* 429 Too Many Requests: Retry later<br>
  The client is polling too frequently. Please wait for a while before polling again.<br>
  The default rate limit is once every 5 seconds.

Success Response:

* code: the authorization code that should be used to obtain an access token.

The client could then perform steps 4 and 5 of the Authorization Code Grant Flow.