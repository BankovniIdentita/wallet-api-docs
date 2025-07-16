# Wallet API documentation
## History 
- Version: 0.0.2 - Update API based on experiences from LSPs 
- Version: 0.0.1 - Filip's first notices
## Introduction
Wallet API is new specific component developed by Bank iD for integration of Wallet functionality
for Czech market. This documentation for now describes only functionality required for POTENTIAL POC 
and does not reflect full scope of Wallet API

# API

## OpenID Configuration Discovery API

Exposed by the **Wallet API**. Authoritatively described in
[Wallet API](wallet-api/wallet-api.yaml). **Not authenticated**.

This endpoint provides information about OpenID Connect configuration.

**GET** `/.well-known/openid-configuration`
**GET** `/.well-known/oauth-authorization-server`

```json

{
  "issuer": "https://wallet.stage.bankid.cz",
  "authorization_endpoint": "https://wallet.stage.bankid.cz/oauth2/authorize",
  "pushed_authorization_request_endpoint": "https://wallet.stage.bankid.cz/oauth2/par",
  "device_authorization_endpoint": "https://wallet.stage.bankid.cz/oauth2/device_authorization",
  "token_endpoint": "https://wallet.stage.bankid.cz/oauth2/token",
  "token_endpoint_auth_methods_supported": [
    "client_secret_basic",
    "client_secret_post",
    "client_secret_jwt",
    "private_key_jwt",
    "tls_client_auth",
    "self_signed_tls_client_auth"
  ],
  "jwks_uri": "https://wallet.stage.bankid.cz/oauth2/jwks",
  "userinfo_endpoint": "https://wallet.stage.bankid.cz/userinfo",
  "end_session_endpoint": "https://wallet.stage.bankid.cz/connect/logout",
  "response_types_supported": [
    "code"
  ],
  "grant_types_supported": [
    "authorization_code",
    "client_credentials",
    "refresh_token",
    "urn:ietf:params:oauth:grant-type:device_code",
    "urn:ietf:params:oauth:grant-type:token-exchange"
  ],
  "revocation_endpoint": "https://wallet.stage.bankid.cz/oauth2/revoke",
  "revocation_endpoint_auth_methods_supported": [
    "client_secret_basic",
    "client_secret_post",
    "client_secret_jwt",
    "private_key_jwt",
    "tls_client_auth",
    "self_signed_tls_client_auth"
  ],
  "introspection_endpoint": "https://wallet.stage.bankid.cz/oauth2/introspect",
  "introspection_endpoint_auth_methods_supported": [
    "client_secret_basic",
    "client_secret_post",
    "client_secret_jwt",
    "private_key_jwt",
    "tls_client_auth",
    "self_signed_tls_client_auth"
  ],
  "code_challenge_methods_supported": [
    "S256"
  ],
  "tls_client_certificate_bound_access_tokens": true,
  "dpop_signing_alg_values_supported": [
    "RS256",
    "RS384",
    "RS512",
    "PS256",
    "PS384",
    "PS512",
    "ES256",
    "ES384",
    "ES512"
  ],
  "subject_types_supported": [
    "public"
  ],
  "id_token_signing_alg_values_supported": [
    "RS256"
  ],
  "scopes_supported": [
    "openid"
  ],
  "registration_endpoint": "https://wallet.stage.bankid.cz/connect/register"
}

```
## JWK Keys Discovery API

Exposed by the **Relying Part** and **Bank iD**. Authoritatively described in
[Wallet API](wallet-api/wallet-api.yaml). **Not authenticated**.

This endpoint returns JSON Web Keys to be used as public keys for verifying OIDC ID Tokens and 
responses, as well as for encrypting requests.

**GET** `/.well-known/jwks.json`

### Response 200 OK:

```json
{
  "keys": [
    {
      "alg": "RS256",
      "kid": "1603dfe0af8f4596",
      "key_ops": ["sign", "verify"],
      "use": "sig",
      "kty": "RSA",
      "x5c": "MIICUDCCAbmgAwIBAgIBADANBgkqhkiG9w0BAQsFADBFMQswCQYDVQQGEwJjejEOMAwGA1UECAwFUHJhaGExEDAOBgNVBAoMB0V4YW1wbGUxFDASBgNVBAMMC2V4YW1wbGUuY29tMB4XDTIwMDExNjE2NDExOFoXDTIxMDExNTE2NDExOFowRTELMAkGA1UEBhMCY3oxDjAMBgNVBAgMBVByYWhhMRAwDgYDVQQKDAdFeGFtcGxlMRQwEgYDVQQDDAtleGFtcGxlLmNvbTCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEArOEYBRyBhcd6u3phrbU2xvTaBoy6W14CpqqfsBrfsUsuSB+JELBCj3a+zRIvy4EY9cnQbF7cPNxbXdCbGEokAUjIIuVBk/I6XhKRe01vlax82o+eFfIhUfl7Xb2Bx9U3m98Qbt3WNrv+VYJjjFP8HWSsWCHKCazj+yvozjuFXUsCAwEAAaNQME4wHQYDVR0OBBYEFN5SUrsStd4aLhBs+MWGRDxLeUP4MB8GA1UdIwQYMBaAFN5SUrsStd4aLhBs+MWGRDxLeUP4MAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQELBQADgYEAL59fE6itiRrck6Z7RCjwnOnebQJxpoB/L7TUC/aUIXss40mCviBVKD+Hl4+3sGyp4J2LlzzqhFcPgR9NyxQt0bkahJGH0UXvZETJe719UA0kGFrPMdt6ujwB6/rafT6TinzXN0lEEGikersTrh3BR9Hjw+v7nCQ0D5RfuDn6s5s="
    }
  ]
}
```

### Authorization endpoint
This GET endpoint is a starting point for OAuth2 and OpenID Connect authorization code flows.
This request authenticates the user and returns tokens to the client application 
as a part of the callback response.

Exposed by the **Wallet API**. Authoritatively described in [Wallet API](wallet-api/wallet-api.yaml). **Not authenticated**.

```http
GET /oatuh2/authorize?
  redirect_uri=https://bankid.cz/callback
  &client_id=D40D25DB-C330
  &response_type=code
  &state=1234
  &scope=openid%20profile.name%20profile.gender
Host: wallet.stage.bankid.cz
```

Valid auth request is processed into OpenID4VP request and depending on scopes following is done:
- scopes are transform one of two ways:
  - if bankid scopes are used, scopes are validated against registered scopes
  - if wallet scopes are used (form namespace:claim), scopes are not validated and are directly processed
- Please note that combining bankid scopes and wallet scopes is not supported right now
- after validation scopes are transformed into DCQL (or temporarily for LSPs Presentation Defintion, depends on profile used in auth request)
  - also signed JWT is created for request with identifier

### Response redirection:

```http
HTTP/1.1 302 Found
Location: https://service-provider.cz/callback?
  code=6a72a932a67cf859570a8fb986dcefce19c844995d30fe1ad32d1e5af5579eb2
  &state=1234
```

### Error redirection:

```http
HTTP/1.1 302 Found
Location: https://bankid.cz/callback?
  error=unauthorized_client
  &state=1234
```

## Token exchange API

The token endpoint is used by the client to obtain an access token by
presenting its authorization grant or refresh token.

Exposed by the **Wallet API**. Authoritatively described in [Wallet API](wallet-api/wallet-api.yaml). 
**Uses client credentials for authentication**.

**POST** `/token`

### Authorization code exchange request:

```http
POST /oauth2/token HTTP/1.1
Host: bankid.cz
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=8BFAC1DA-3F94-4BBD-A743-473080FB6073
&redirect_uri=https://serviceprovider.cz/callback
&client_secret=368b8099c14c0964a4a9f958c8b5786c46845ec1
```
---
### Authorization code exchange response 200 OK:

```json
{
  "access_token": "c03e997c-aa96-4b3f-ad0c-98626833145d",
  "token_type": "Bearer",
  "refresh_token": "1f703f5f-75da-4b58-a1b0-e315700e4227",
  "expires_in": 3600,
  "id_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```


### Response 400 Request invalid:

```http
HTTP/2 400 Bad Request
Content-Type: application/json;charset=utf-8
Cache-Control: no-store
Pragma: no-cache

{
  "error": "invalid_request",
  "error_description": "The request is missing a required parameter"
}
```
---

## GET /userinfo 
Userinfo endpoint is used to retrieve information send by Wallet to backend. 
Backend processes returned vp_token with disclosed presentations and transforms it to plain JSON structure. 
You may get the complete vp_token details from the `/vptoken` endpoint.
Authenticated by access_token from token endpoint.
Please note that currently endpoint does not returns data as signed or encrypted JWT opposed to current Bank iD interface

```http
GET /userinfo HTTP/2
Host: bankid.cz
Accept: application/json
Authorization: Bearer c03e997c-aa96-4b3f-ad0c-98626833145d
```

### Response 200 OK
Returns Userinfo response with data. 
```http
HTTP/2 200 OK
Content-Type: application/json;charset=utf-8


{
  "given_name": "Bernd",
  "family_name": "Abt",
  "raw": {
    "eu.europa.ec.eudi.pid.1": {
      "given_name": "Bernd",
      "family_name": "Abt"
    }
  },
  "sub": "dummy",
  "txn": "52be9ce4-c16e-4b72-aaaf-549e1c75849d"
}
```

### Response 400
```html
HTTP/2 400 Bad Request
Content-Type: application/json;charset=utf-8
Cache-Control: no-store
Pragma: no-cache

{
  "error": "invalid_request",
  "error_description": "The request is missing a required parameter"
}
```

## GET /vptoken
Returns decrypted presentations as JSON array, each element contains sigle presentation in mdoc or SD-JWT format. 
Authenticated by access_token from token endpoint. 

```http
GET /vptoken HTTP/2
Host: bankid.cz
Accept: application/json
Authorization: Bearer c03e997c-aa96-4b3f-ad0c-98626833145d
```
### Response 200 OK
Returns Userinfo response with data.
```http
HTTP/2 200 OK
Content-Type: application/json;charset=utf-8

[ "eyJ... ",
  "o2d2ZXJzaW9uYzEuMGlk..."
]
```