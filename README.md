# Bank iD's Wallet API documentation

## About 
This is documentation of Bank iD's API for integration of Relaying parties to EU Digital Wallet ecosystem

This API documentation is Work in Progress and aims at supporting Wallet PoC.
Not all functionality may be supported. Please consult us via issue or Discord before using it.

Should you find any bug or inconsistency please create issue in repo.
Should you have any request for functionality please create issue in repo.

You can also contact us by Discord https://discord.gg/XnK7yvHD

## Structure
Docs has following parts:
- yaml file with authoritative description of API
- md file with examples and comments
- sequence diagrams for QES

## Supported protocols and RFC
On Relaying party:
- OAuth 2.0
- OpenID Connect Core
- OpenID Connect Dynamic Client Registration
- OpenID Connect Session Management
- OAuth 2.0 Authorization Server Metadata
- OAuth2 Pushed Authorization Request

For Wallet integration:
- OpenID4VCI draft 15
- OpenID4VP draft 21,24,28 (selectable by variable)
- Iso 18013:5 extension for OpenID4VP
- SD-JWT for issuance
- High Assurance profile for OIDC4VC
- High Assurance profile for OIDC4VP

Please note that Presentation definition is supported only in OpenIDVP draft 21 and 24, and will be removed in near future. 
## Funcionality and Protocols in progress

QEAA Issuer

Relaying party
- OAuth2 Rich Authorization Request
- CSC 2.0 API
- QES resolution between Wallet Centric and QTSP model
- Proof Key for Code Exchange by OAuth Public Clients (PKCE)

Wallet integration:
- Transaction data in OpenID4VP (Document Signature, Payments)
- Wallet centric QES
- QTSP centric QES
 


