# Wallet API documentation
## History 
Version: 0.0.1 - Filip's first notices
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
  "issuer": "https://idp.example.com",
  "authorization_endpoint": "https://idp.example.com/auth",
  "token_endpoint": "https://idp.example.com/token",
  "userinfo_endpoint": "https://idp.example.com/userinfo",
  "jwks_uri": "https://idp.example.com/.well-known/jwks",
  "registration_endpoint": "https://idp.example.com/register",
  "scopes_supported": ["openid", "profile.email"],
  "response_types_supported": ["code"],
  "response_modes_supported": ["query"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "acr_values_supported": ["string"],
  "subject_types_supported": ["pairwise"],
  "id_token_signing_alg_values_supported": ["PS512"],
  "id_token_encryption_alg_values_supported": ["A256GCM"],
  "id_token_encryption_enc_values_supported": ["A256GCM"],
  "userinfo_signing_alg_values_supported": ["PS512"],
  "userinfo_encryption_alg_values_supported": ["A256GCM"],
  "userinfo_encryption_enc_values_supported": ["A256GCM"],
  "profile_signing_alg_values_supported": ["PS512"],
  "profile_encryption_alg_values_supported": ["A256GCM"],
  "profile_encryption_enc_values_supported": ["A256GCM"],
  "request_object_signing_alg_values_supported": ["PS512"],
  "request_object_encryption_alg_values_supported": ["A256GCM"],
  "request_object_encryption_enc_values_supported": ["A256GCM"],
  "token_endpoint_auth_methods_supported": ["private_key_jwt"],
  "token_endpoint_auth_signing_alg_values_supported": ["PS512"],
  "display_values_supported": ["page"],
  "service_documentation": "https://idp.example.com/docs",
  "claims_locales_supported": ["en-CA", "en"],
  "ui_locales_supported": ["cs", "en-US", "en"],
  "claims_parameter_supported": false,
  "request_parameter_supported": false,
  "request_uri_parameter_supported": true,
  "require_request_uri_registration": false,
  "op_policy_uri": "https://idp.example.com/policy",
  "op_tos_uri": "https://idp.example.com/tos",
  "backchannel_logout_supported": true,
  "claims_supported": ["string"]
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
Host: somebank.cz
```
Valid auth request is processed into SIOPv2 request and depending on scopes following is done:
- if scope is sub only, response type for SIOPv2 is id_token
- otherwise response type for SIOPv2 is id_token vp_token and scopes are tranformed into Presentation definition
  - also signed JWT is created for request with identifier

### Response redirection:

```http
HTTP/1.1 302 Found
Location: https://bankid.cz/callback?
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
# Internal logic

## Create SIOPv2 auth request 
Internal API 
