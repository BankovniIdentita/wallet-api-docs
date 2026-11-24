# Transaction data types and TS12 behavior 

In this document we describe transaction_data types functionality for payment initiation and authorization with Wallet API. 
This document right now describes TS12 functionality, PaSO will be added in later development and results of 
SCARLET initiative will be also added once they are public. 

## Changelog
 Versions:
- 0.0.1 - initial draft to support WE BUILD functionality for TS12

 ## How does transaction_data types work in Wallet API 

Wallet API support now 4 basic types from TS12
- urn:eudi:sca:payment:1
- urn:eudi:sca:login_risk_transaction:1
- urn:eudi:sca:account_access:1
- urn:eudi:sca:emandate:1

These are basic types that are mandatory by TS12 and SCA attestation should support them via its vct metadata types.

Basic working is simple 
- in Pushed Authorize Request or regular OAuth2 authorize you need to add field authorization_details which follows 
    Rich Authorization Request standard, matching transaction type scheme registered in Wallet API
- Wallet API will validate this against configured schemes and transform this to transaction_data claim in request to Wallet
- When Wallet collects request transaction data and process it by TS12 rules
- On response Wallet API validates transaction_data_hashes and returns response accordingly in API endpoints 


## On transaction_type extensibility 
Only 4 basic types might not always be sufficient for all use case i.e. some entities might want additional fields 
in transaction data.
First we start with some limitations: 
- Wallet API has all transaction_data types managed instance wide for all clients
  - this is given fact that more that one client may use same types and this data are public in first place anyway. 
- Given 1st point transaction_data types must be unique in instance, and in fact we would recommend for it to be unique globally 
- Wallet may not resolve your custom metadata and you should have  fallback for that. 

### How to create your own transaction_type 
1. Decide with base type you will be extending 
2. Prepare additional claims in your custom transaction_type scheme with proper naming
3. Create us issue on Github for new transaction data type
4. Add new transaction_type to your SD-JWT vct metadata 
5. Profit? :) 

We provide for testing purposes of this functionality type `urn:eudi:sca:payment:2`, which has additional 
field `payee.account_number` in scheme and this may be tested with our BankId SUA IBAN attestation. 

### Known issues and limitations
- Right now filename must be in format `ts12-<transaction_type>-data-model.json` transaction_type has replaced : with -
  - example file `ts12-urn-eudi-sca-payment-2-data-model.json` is for transaction type `urn:eudi:sca:payment:2`
  - In future version we will also support transaction_types that are url for scheme resolution directly
- Requester has to add credentials_ids into authorization_details, this can be retrieved via simple request with proper scope 
and extracting credential_ids from Wallet request. 
  - This will be fixed before sandbox launch and Wallet API will automatically add all credential_ids matching scope 
  and requesting SCA attestation
- Support for TS12 is not mandatory in Wallet and Wallet may refuse request containing unknow transaction data types 

## Examples 

Authorization Request with authorization details containing payment
```
GET https://wallet.stage.bankid.cz/oauth2/authorize?
    client_id=oidc-client
    &state=206d51d5-2433-4bf2-8e7d-28854897c90b
    &nonce=x5vVekwnw-GC70anwmP5e_DRzfUdxrVfOgOyXOIOEORwwIzKhKoZ4uYbuUFiOdl6
    &redirect_uri=https://rp-demo.lab.euwallet.cz/callback
    &scope=sua_iban
    &response_type=code
    &max_age=360
    &credential_format=SD_JWT
    &authorization_details=%5B%7B%22type%22%3A%22urn%3Aeudi%3Asca%3Apayment%3A1%22%2C%22credential_ids%22%3A%5B%22sm_https___wallet-stage-bankid-cz_d3884895-816a-45e6-86e2-1024783fa295_vct_bankid_sua_iban-g1%22%5D%2C%22transaction_data_hashes_alg%22%3A%5B%22sha-256%22%5D%2C%22payload%22%3A%7B%22transaction_id%22%3A%2201fc8061-41a3-4a38-a618-8b7a353623e7%22%2C%22date_time%22%3A%222026-11-24T09%3A54%3A09.701530203Z%22%2C%22payee%22%3A%7B%22name%22%3A%22Bank%20iD%20CZ%22%2C%22id%22%3A%2200000000-0000-0000-0000-000000000000%22%2C%22account_number%22%3A%22123%2F0300%22%7D%2C%22pisp%22%3A%7B%22legal_name%22%3A%22Bank%20iD%20Wallet%20API%22%2C%22brand_name%22%3A%22Awesome%20Connector%22%2C%22domain_name%22%3A%22https%3A%2F%2Fwallet.stage.bankid.cz%22%7D%2C%22execution_date%22%3A%222026-11-24%22%2C%22currency%22%3A%22CZK%22%2C%22amount%22%3A1.0%7D%7D%5D

```
Decoded authorization_details
```json
[
  {
    "type": "urn:eudi:sca:payment:1",
    "credential_ids": [
      "sm_https___wallet-stage-bankid-cz_d3884895-816a-45e6-86e2-1024783fa295_vct_bankid_sua_iban-g1"
    ],
    "transaction_data_hashes_alg": [
      "sha-256"
    ],
    "payload": {
      "transaction_id": "01fc8061-41a3-4a38-a618-8b7a353623e7",
      "date_time": "2026-11-24T09:54:09.701530203Z",
      "payee": {
        "name": "Bank iD CZ",
        "id": "00000000-0000-0000-0000-000000000000",
        "account_number": "123/0300"
      },
      "pisp": {
        "legal_name": "Bank iD Wallet API",
        "brand_name": "Awesome Connector",
        "domain_name": "https://wallet.stage.bankid.cz"
      },
      "execution_date": "2026-11-24",
      "currency": "CZK",
      "amount": 1.0
    }
  }
]

```

Vct metadata example with custom types
```json
{
  "vct": "https://wallet.stage.bankid.cz/d3884895-816a-45e6-86e2-1024783fa295/vct/bankid_sua_iban",
  "name": "Bank iD SCA IBAN",
  "claims": [
    {
      "sd": "never",
      "path": [
        "sub"
      ],
      "display": [
        {
          "label": "Subject",
          "locale": "en-US",
          "description": "Anonymous identifier of attestation holder"
        }
      ],
      "mandatory": true
    },
    {
      "sd": "allowed",
      "path": [
        "masked_iban"
      ],
      "display": [
        {
          "label": "Masked IBAN",
          "locale": "en-US",
          "description": "Masked IBAN"
        }
      ],
      "mandatory": true
    },
    {
      "sd": "allowed",
      "path": [
        "iban"
      ],
      "display": [
        {
          "label": "IBAN",
          "locale": "en-US",
          "description": "Full IBAN"
        }
      ],
      "mandatory": true
    },
    {
      "sd": "allowed",
      "path": [
        "national_bank_code"
      ],
      "display": [
        {
          "label": "National bank code",
          "locale": "en-US",
          "description": "National bank code of issuing bank"
        }
      ],
      "mandatory": true
    },
    {
      "sd": "allowed",
      "path": [
        "bic"
      ],
      "display": [
        {
          "label": "BIC",
          "locale": "en-US",
          "description": "BIC of issuing bank"
        }
      ],
      "mandatory": true
    },
    {
      "sd": "allowed",
      "path": [
        "currency"
      ],
      "display": [
        {
          "label": "Currency",
          "locale": "en-US",
          "description": "Currency of account"
        }
      ],
      "mandatory": true
    }
  ],
  "display": [
    {
      "name": "Bank iD SCA IBAN",
      "locale": "en-US",
      "description": "SCA Attestation containing IBAN issued by Bank iD for WE BUILD"
    }
  ],
  "extends": "https://webuildconsortium.eu/sca/sca-iban/1.0",
  "transaction_data_types": {
    "urn:eudi:sca:payment:1": {
      "claims": [
        {
          "path": [
            "payload",
            "transaction_id"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Transaction ID",
              "description": "Unique identifier of the transaction"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "payee",
            "name"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Payee's Name",
              "description": "Name of countrer party"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "pisp",
            "legal_name"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "PISP Legal name",
              "description": "Legal name of PISP"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "pisp",
            "brand_name"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "PISP name",
              "description": "Brand name of PISP"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "currency"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Currency",
              "description": "Currency of the transaction"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "amount"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Amount",
              "description": "Amount of the transaction"
            }
          ],
          "visualisation": 4
        }
      ],
      "schema": "urn:eudi:sca:payment:1",
      "ui_labels_uri": "https://wallet.stage,bankid.cz/json/ui_labels.json"
    },
    "urn:eudi:sca:payment:2": {
      "claims": [
        {
          "path": [
            "payload",
            "transaction_id"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Transaction ID",
              "description": "Unique identifier of the transaction"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "payee",
            "name"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Payee's Name",
              "description": "Name of countrer party"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "pisp",
            "legal_name"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "PISP Legal name",
              "description": "Legal name of PISP"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "pisp",
            "brand_name"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "PISP name",
              "description": "Brand name of PISP"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "currency"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Currency",
              "description": "Currency of the transaction"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "amount"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Amount",
              "description": "Amount of the transaction"
            }
          ],
          "visualisation": 4
        },
        {
          "path": [
            "payload",
            "payee",
            "account_number"
          ],
          "display": [
            {
              "lang": "en-US",
              "label": "Account Number",
              "description": "Account number of Payee if id is no sufficient"
            }
          ],
          "visualisation": 4
        }
      ],
      "schema": "urn:eudi:sca:payment:2",
      "ui_labels_uri": "https://wallet.stage,bankid.cz/json/ui_labels.json"
    }
  }
}
```
Transaction type scheme with custom claim example

```json 
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "properties": {
        "transaction_id": {
            "type": "string",
            "maxLength": 36,
            "minLength": 1,
            "examples": [
                "8D8AC610-566D-4EF0-9C22-186B2A5ED793"
            ]
        },
        "date_time": {
            "type": "string",
            "format": "date-time",
            "examples": [
                "2025-11-13T20:20:39+00:00"
            ]
        },
        "payee": {
            "type": "object",
            "properties": {
                "name": {
                    "type": "string"
                },
                "id": {
                    "type": "string"
                },
                "account_number": {
                    "type": "string"
                },
                "logo": {
                    "type": "string",
                    "format": "uri"
                },
                "website": {
                    "type": "string",
                    "format": "uri"
                }
            },
            "required": [
                "name",
                "id"
            ]
        },
        "pisp": {
            "type": "object",
            "properties": {
                "legal_name": {
                    "type": "string"
                },
                "brand_name": {
                    "type": "string"
                },
                "domain_name": {
                    "type": "string"
                }
            },
            "required": [
                "legal_name",
                "brand_name",
                "domain_name"
            ]
        },
        "execution_date": {
            "type": "string",
            "format": "date-time"
        },
        "currency": {
            "type": "string",
            "pattern": "^[A-Z]{3}$"
        },
        "amount": {
            "type": "number"
        },
        "amount_estimated": {
            "type": "boolean"
        },
        "amount_earmarked": {
            "type": "boolean"
        },
        "sct_inst": {
            "type": "boolean"
        },
        "recurrence": {
            "type": "object",
            "properties": {
                "start_date": {
                    "type": "string",
                    "format": "date-time"
                },
                "end_date": {
                    "type": "string",
                    "format": "date-time"
                },
                "number": {
                    "type": "integer"
                },
                "frequency": {
                    "type": "string",
                    "enum": [
                        "INDA",
                        "DAIL",
                        "WEEK",
                        "TOWK",
                        "TWMN",
                        "MNTH",
                        "TOMN",
                        "QUTR",
                        "FOMN",
                        "SEMI",
                        "YEAR",
                        "TYEA"
                    ]
                },
                "mit_options": {
                    "type": "object",
                    "properties": {
                        "amount_variable": {
                            "type": "boolean"
                        },
                        "min_amount": {
                            "type": "number"
                        },
                        "max_amount": {
                            "type": "number"
                        },
                        "total_amount": {
                            "type": "number"
                        },
                        "initial_amount": {
                            "type": "number"
                        },
                        "initial_amount_number": {
                            "type": "integer"
                        },
                        "apr": {
                            "type": "number"
                        }
                    }
                }
            },
            "required": [
                "frequency"
            ]
        }
    },
    "required": [
        "transaction_id",
        "payee",
        "currency",
        "amount"
    ],
    "additionalProperties": false
}

```

