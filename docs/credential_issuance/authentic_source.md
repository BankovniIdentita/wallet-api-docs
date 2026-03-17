# Authentic Source Development Manual

### Changelog

- 0.0.1 - First draft for review by partners. Author: @filip-hladky
    
## Introduction

Bank iD Wallet API issuer functionality requires for organization wishing to issue credentials via Wallet API 
to have a dedicated Authentic Source. Authentic Source is a component responsible for managing and issuing data for credentials. It is a critical component of the Wallet API ecosystem and plays a vital role in ensuring secure and efficient credential issuance.

The Authentic Source is designed to be flexible and scalable, allowing organizations to issue credentials to a wide range of users and applications. It is also designed to be secure, with robust authentication and authorization mechanisms in place to protect user data and prevent unauthorized access.

In this manual, we will provide detailed instructions on how to develop Authentic Source. In further version we will also add manual for set up and configure an Authentic Source, as well as how to integrate it with the Wallet API. 

By following the instructions in this manual, organizations can
ensure that their data for credentials are handled securely and efficiently, and that their users have a seamless experience when using the Wallet API.

Please note that this manual is intended for developers and technical staff who are responsible for developing  and managing Authentic Sources. If you are not familiar with the Wallet API or credential issuance, we recommend that you consult with a Bank iD's  Wallet API expert before proceeding.

## API requirements

API of Wallet API authentic source needs to have following properties
- fulfill required endpoints of authentic_source.yaml Open API description
- provide one of 2 option of API authentication
  - OAuth/Open ID Connect (will be supported in future versions)
  - Client Assertion by Wallet API issuer 
- optionally can require mTLS – in that case Authentic Source is responsible for issuing client certificate

In future we will also support Authentic Sources fulfilling API by ETSI TS 119 478
As per API endpoints, now only .well-known endpoint and /retrieve endpoint are required. We assume in near future we will also add /authorize endpoint for validation of data from Wallet prior to credentials issuance

### Client Assertion authentication 
The following code describes structure of client assertion sent by Wallet API issuer to Authentic Source

        val jwtClaimsSet = JWTClaimsSet.Builder().
            issuer(issuerDomain)
            .audience(authenticSourceUrl)
            .issueTime(Date())
            .jwtID(UUID.randomUUID().toString())
            .subject(issueriD)
            .claim("scope", credentialConfigurationId)
            .claim("user_auth", userAuth)

Please note that Authentic source needs to validate and  understand following claims
- iss needs to be URL of Bank iD's Wallet API or your private instance
- audience will be base URL of your Authentic Source
- subject is Wallet API issuer id for audit purposes
- scope will be credential configuration id, that authentic source needs to resolve to proped attributes that needs to be send in response
- user_auth will be user authentication token. This might be one of 2 things:
  - map of claims with data returned from Wallet that Authentic Source needs to match to its data
  - claim "token" with token returned from future /authorize endpoint (WIP)
Moreover JWT is singed by private key of Issuer. Signature MUST be validated by public key that needs to be resolved at .well-known/jwks/{issuerId} url 

### Attributes matching and Wallet request by Authentic source

Issuer requests some credentials from Wallet for Authentic Source to match to it's own internal data. 
In this first iteration we assume that Issuer will reuquest following data
- attributes from PID for cross-border matching by CIR (first name, last name, date of birth, country, PID metadata)
- attributes from Czech Pub-EAA (once published)
- SUA/SCA in case Wallet is already onboarded and used for login
- WUA for issuance

Authentic Source must match those data from Wallet to it's internal database 
and resolve by its own configuration which attributes will be returned 
to Issuer for credential issuance. 

Plase note that if data from authentic source are insufficient for issuance, issuer will return error to Wallet

# Final note
This is preliminary documentation for PoC and testing WE BUILD and APTITUDE consortia as well as implementation partners. 
This is not final version and implmentation might be ahead of specification. In such case let us know by creating issue or email to 
[filip-hladky](mailto:filip@bankid.cz)