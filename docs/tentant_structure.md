# Tenant structure of Wallet API 

Wallet API provides multiple levels on which calling application are separated from Wallet
Following are entities that are implementing tenancy:
- Wallet API's OpenID Client (OIDC Client)
- Wallet Client 
- Issuer as special case of OIDC Client 



```mermaid


    classDiagram
    direction LR
    class Entity {
        +name
    }

    class System {
        +name
    }

    class OIDCClient {
        +clientId
        +redirectUris
        +scopes
    }


    class WalletClient {
        +walletClientId
        +accessCertificate : AccessCertificate
        +registrationCertificates : List~RegistrationCertificate~
    }

    class AccessCertificate {
        +certificateId
        +subject
        +validity
    }

    class RegistrationCertificate {
        +certificateId
        +purpose
        +requestedAttributes
        +validity
    }


    Entity "1" *-- "1..*" System : has multiple
    System  "1" --> "1..*" OIDCClient : uses single
    OIDCClient "0..*" --> "1" WalletClient: uses single
    WalletClient *-- "1" AccessCertificate : has single
    WalletClient *-- "1..*" RegistrationCertificate : has multiple


note for WalletClient "One Wallet Client can be used by multiple OIDC Clients,<br/> directly or as intermediary"
note for OIDCClient "One OIDC Client may serve multiple Systems within a single Entity"
note for RegistrationCertificate "One Wallet Client may have multiple Registration certificates for different Systems or usecases.
Intermediary role is when registration certificate is issued for different entity than Wallet Client's access certificate"

```