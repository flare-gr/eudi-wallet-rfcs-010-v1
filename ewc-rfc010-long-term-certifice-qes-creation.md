# RFC-010 Document Signing on a Remote Signing Service Provider using Long-Term Certificates (v1)

**Status: Under development**

**Authors:**

- Mr. Kyriakos Giannakis (Intesi Group, Italy | Flare, Greece)

**Reviewers:**

TBA

**Table of Contents:**

- [1.0 Summary](#10-summary)
- [2.0 Motivation](#20-motivation)
- [3.0 The Signing Architecture](#30-the-signing-architecture)
    - [3.1 Phase 1: Service Provider Access & User Authentication](#31-phase-1-service-provider-access--user-authentication)
    - [3.2 Phase 2: Certificate Listing and Selection](#32-phase-2-certificate-listing-and-selection)
    - [3.3 Phase 3: Signature Confirmation & Private Key Unlocking (Credential Authorization)](#33-phase-3-signature-confirmation--private-key-unlocking-credential-authorization)
    - [3.4 Phase 4: Signature Creation](#34-phase-4-signature-creation)
    - [3.5 Phase 5: Signed Document Formation and Retrieval](#35-phase-5-signed-document-formation-and-retrieval)
- [4. Signing Process](#4-signing-process)
    - [4.0 Overview](#40-overview)
    - [4.1 Phase 1: Service Provider Access & User Authentication](#41-phase-1-service-provider-access--user-authentication)
        - [4.1.1 Service Access by User](#411-service-access-by-user)
        - [4.1.2 User Authentication](#412-user-authentication)
    - [4.2 Phase 2: Certificate Listing and Selection](#42-phase-2-certificate-listing-and-selection)
    - [4.3 Phase 3: Signature Confirmation & Private Key Unlocking (Credential Authorization)](#43-phase-3-signature-confirmation--private-key-unlocking-credential-authorization)
        - [4.3.1 Signing Confirmation as Willful Act](#431-signing-confirmation-as-willful-act)
        - [4.3.2 Private Key Unlocking (Credential Authorization)](#432-private-key-unlocking-credential-authorization)
    - [4.4 Phase 4: Signature Creation](#44-phase-4-signature-creation)
    - [4.5 Phase 5: Signed Document Formation and Retrieval](#45-phase-5-signed-document-formation-and-retrieval)
- [5. Reference](#5-reference)

**Changelog:**

- Nov. 11 2024: Initialization of authoring process.
- Nov. 28 2024: Phase 4,5 authoring. Reformatting of headings and content. Addition of references.
- Nov. 29 2024: Added Overview section.

# 1.0 Summary:

This Specification defines the procedures for using the EUDI wallet to digitally sign a document, using Long-Term certificates, on a Remote Signing Service Provider (SSP). It uses the user's PID to identify the user's Certificates. The Signer's Private Keys are stored safely in a Remote Qualified Electronic Signature (RQES) service.

**Remote QES services shall adhere to the [CSC (Cloud Signature Consortium)](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf) specifications that are also the basis for the JSON part of the ETSI TS 119 432 standard on protocols for remote digital signature creation.**

# 2.0 Motivation:

The motivation for this specification is to provide a robust and secure framework for enabling remote digital document signing using long-term certificates and the EUDI Wallet as a means of user authentication. As organizations and individuals increasingly transition to digital workflows, the need for verifiable and legally binding electronic signatures has grown significantly. This document aims to establish a standardized and interoperable approach that ensures trust, security, and ease of use and implementation in the signing process.

# 3.0 The Signing Architecture:

The architecture covered in this specification follows the process of remotely signing a document using long-term certificates, handled by a Remote QES (or AES) Service, as detailed in D4.8.

The architecture will be broken down in 4 main phases:
1. Phase 1: Service Provider Access & User Authentication
2. Phase 2: Certificate Listing and Selection
3. Phase 3: Signature Confirmation & Private Key Unlocking (Credential Authorization)
4. Phase 4: Signature Creation
5. Phase 5: Signed Document Formation and Retrieval

**Remote QES services shall adhere to the [CSC (Cloud Signature Consortium)](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf) specifications that are also the basis for the JSON part of the ETSI TS 119 432 standard on protocols for remote digital signature creation.**

> **Note**: The “Signature Creation Application” is shown as a separate Signing Service but may be integrated into the Service Provider. This depends on available software that the service provider can use.

> **Note**: The Signer's Document (SD) uploading process is out of scope of this RFC. The SD can be uploaded either by the user or the Service Provider, prior to the execution of the signing procedure.

# 4. Signing Process:

## 4.0 Overview:

```mermaid
   sequenceDiagram
    participant User
    participant EUDI Wallet
    participant Service Provider
    box Purple CSC Protocol Usage
    participant Signing Service
    participant RQES Provider
    end

    Note over User, Signing Service: Phase 1: Service Provider Access & User Authentication
    User->>Service Provider: Service Access
    Service Provider->>Signing Service: Request Signing of Document
    Signing Service->>User: PID Presentation Request via OID4VP
    EUDI Wallet->>Signing Service: PID Presentation

    Note over Signing Service, RQES Provider: Phase 2: Certificate Listing and Selection
    Signing Service->>RQES Provider: POST /csc/v2/credentials/list
    RQES Provider->>Signing Service: { credentialIDs: [...], credentialInfos: [...] }

    Note over User, RQES Provider: Phase 3: Signature Confirmation & Private Key Unlocking (Credential Authorization)
    Signing Service->>User: PID Presentation Request (Signature Confirmation)
    EUDI Wallet->>Signing Service: PID Presentation

    
    User->>RQES Provider: Credential Authorization (for oauth2code flow)
    activate RQES Provider
    activate Signing Service
    User->>Signing Service: Credential Authorization (for explicit flow)
    deactivate Signing Service
    

    Signing Service->>RQES Provider: POST /csc/v2/credentials/authorize
    deactivate RQES Provider
    RQES Provider->>Signing Service: SAD

    Note over Signing Service, RQES Provider: Phase 4: Signature Creation
    Signing Service->>RQES Provider: POST /csc/v2/signatures/signHash
    RQES Provider->>Signing Service: Signed Hash

    Note over User, Signing Service: Phase 5: Signed Document Formation and Retrieval
    Signing Service->>Service Provider: Signed Document
    Service Provider->>User: Signed Document
```

## 4.1 Phase 1: Service Provider Access & User Authentication

#### Overview:

```mermaid
   sequenceDiagram
  participant User
  participant EUDI Wallet
  participant Service Provider
  participant Signing Service

  User->>Service Provider: Service Access
  Service Provider->>Signing Service: Request Signing of Document
  Signing Service->>User: PID Presentation Request via OID4VP
  EUDI Wallet->>Signing Service: PID Presentation
```

In the first part of the signing procedure, the **User** accesses (through their browser) the **Service Provider** to request a document to be signed.

### 4.1.1: Service Access by User:

Initially, the User accesses the Service Provider through their browser. Upon arrival, the user should be presented with an Authentication screen, requesting presentation of their PID.

### 4.1.2: User Authentication

The Service Provider should (unless previously authenticated) require the user is authenticated. Authentication of the user happens through a presentation of their PID.

Requested data should include, at a minimum, the [mandatory attributes](https://eu-digital-identity-wallet.github.io/eudi-doc-architecture-and-reference-framework/1.4.0/annexes/annex-3/annex-3.01-pid-rulebook/#23-pid-attributes) of the user's PID to be presented (`family_name`, `given_name`, `birth_date`, `age_over_18`, `issuance_date`, `expiry_date`).

Checks should be performed in order to make sure the presented document is valid and current (out of scope).

## 4.2 Phase 2: Certificate Listing and Selection

**The presented claims from the User's PID** are then used for the **determination and selection** of the User's Signing Certificate (`Credential`) from the Remote QES Service:

**credentials/list**

```mermaid
%% credentialList
  sequenceDiagram
    participant Signing Service
    participant RQES Provider
    Signing Service->>RQES Provider: POST /csc/v2/credentials/list
    RQES Provider->>Signing Service: { credentialIDs: [...], credentialInfos: [...] }
```

**Sample Request**:
```http request
POST /csc/v2/credentials/list HTTP/1.1
Host: rqes.example.com
Authorization: Bearer ...
Content-Type: application/json
{
    "credentialInfo": true,
    "certificates": "chain",
    "certInfo": true,
    "authInfo": true
}
```

**Sample Response**:

```http request
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "credentialIDs": [
    "GX0112348",
    "HX0224685"
  ],
  "credentialInfos": [
    {
      "credentialID": "GX0112348",
      "key": {
        "status": "enabled",
        "algo": [
          "1.2.840.113549.1.1.11",
          "1.2.840.113549.1.1.10"
        ],
        "len": 2048
      },
      "cert": {
        "status": "valid",
        "certificates": [
          "<Base64-encoded_X.509_end_entity_certificate>",
          "<Base64-encoded_X.509_intermediate_CA_certificate>",
          "<Base64-encoded_X.509_root_CA_certificate>"
        ],
        "issuerDN": "<X.500_issuer_DN_printable_string>",
        "serialNumber": "5AAC41CD8FA22B953640",
        "subjectDN": "<X.500_subject_DN_printable_string>",
        "validFrom": "20200101100000Z",
        "validTo": "20230101095959Z"
      },
      "auth": {
        "mode": "explicit",
        "expression": "PIN AND OTP",
        "objects": [
          {
            "type": "Password",
            "id": "PIN",
            "format": "N",
            "label": "PIN",
            "description": "Please enter the signature PIN"
          },
          {
            "type": "Password",
            "id": "OTP",
            "format": "N",
            "generator": "totp",
            "label": "Mobile OTP",
            "description": "Please enter the 6 digit code you received by SMS"
          }
        ]
      },
      "multisign": 5,
      "lang": "en-US"
    }
  ]
}
```

**credentials/info (optional)**:

SSPs can utilize the `credentials/info` endpoint to receive info about a specific credential:

**Sample Request**:
```http request
POST /csc/v2/credentials/info HTTP/1.1
Host: rqes.example.com
Authorization: Bearer ...
Content-Type: application/json
{
    "credentialID": "GX0112348",
    "certificates": "chain",
    "certInfo": true,
    "authInfo": true
}
```

The actual process of the certificate selection is not detailed in this RFC, as different Signing Services might use different methods for certificate labeling and mapping to User data.

> **Note**: Authentication to the RQES Provider is out of scope. Implementors will need to follow the CSC API Spec Guidelines for Service Authentication & Authorization. The user might need to be redirected to the RQES Provider to complete authorization.

## 4.3: Phase 3: Signature Confirmation & Private Key Unlocking (Credential Authorization)

During this step of the process, the Signing of the Signer's Document (SD) must be confirmed by the user and the Private Key of the User's Certificate will need to be unlocked (authorized for use), in order to obtain the `Signature Activation Data (SAD)`.

### 4.3.1: Signing Confirmation as Willful Act

In order to confirm that the signature approval is a willful act, a second PID Presentation must be requested by the Signing Service:

```mermaid
   sequenceDiagram
  participant User
  participant EUDI Wallet
  participant Signing Service

  Signing Service->>User: PID Presentation Request via OID4VP
  EUDI Wallet->>Signing Service: PID Presentation
```

During this step, the Signing Service must confirm that the PID attributes presented exactly match the PID attributes presented in Phase 1. Any deviation should result in the immediate termination of the process.

### 4.3.2: Private Key Unlocking (Credential Authorization)

The Signing Service will need to parse the `auth.mode` object of the user's credential to determine the mode of credential authorization:

#### Authorization Code Flow (oauth2code):

If the auth mode is set to follow the **OAuth2 Authorization Code Flow**, the Signing Service will need to redirect the user to the RQES Provider's
`oauth2/authorize` and the `oauth2/token` endpoints, as defined by [RFC-6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1) and while following the procedure in the CSC API v2 Spec.

**oauth2/authorize:**

```http request
GET https://rqes.example.com/oauth2/authorize?
    response_type=code&
    client_id=<OAuth2_client_id>&
    redirect_uri=<OAuth2_redirect_uri>&
    scope=credential&
    code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bww-uCHaoeK1t8U&
    code_challenge_method=S256&
    credentialID=GX0112348&
    numSignatures=1&
    hashes=MTIzNDU2Nzg5MHF3ZXJ0enVpb3Bhc2RmZ2hqa2zDtnl4&
    hashAlgorithmOID=2.16.840.1.101.3.4.2.1&state=12345678
```

**oauth2/token:**

```http request
POST https://rqes.example.com/oauth2/token HTTP/1.1
Host: www.domain.org
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=FhkXf9P269L8g&
client_id=<OAuth2_client_id>&
client_secret=<OAuth2_client_secret>&
redirect_uri=<OAuth2_redirect_uri>
```

#### Explicit Flow (explicit):

In the case of `explicit` credential authorization, the Signing Service will need to parse the `expression` parameter of the respective
credential and present the required authorization prompts to the User (for example, a PIN prompt).

For each step of the authorization, the specific CSC API endpoints will need to be queried by the Signing Service (for example, the
`credentials/getChallenge` endpoint, to receive an OTP).

**credentials/authorize:**

After the respective input from the user has been collected, the `credentials/authorize` endpoint can be queried by the Signing Service
to finalize the authorization process:

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant Signing Service
    participant RQES Provider
    Signing Service->>RQES Provider: POST /csc/v2/credentials/authorize
    RQES Provider->>Signing Service: SAD
```

```http request
POST /csc/v2/credentials/authorize HTTP/1.1
Host: rqes.example.com
Authorization: Bearer ...
Content-Type: application/json
{
  "credentialID": "GX0112348",
  "numSignatures": 1,
  "hashes": [
    "sTOgwOm+474gFj0q0x1iSNspKqbcse4IeiqlDg/HWuI="
  ],
  "hashAlgorithmOID": "2.16.840.1.101.3.4.2.1",
  "authData": [
    {
      "id": "PIN",
      "value": "123456"
    },
    {
      "id": "OTP",
      "value": "738496"
    }
  ]
}
```

> Note: The hash of the document should be passed onto the authorization request, to bind to the SAD to the hash to be signed, as to not be able to be used to sign a different content.

**Sample Response**:

```http request
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "SAD": "_TiHRG-bAH3XlFQZ3ndFhkXf9P24/CKN69L8gdSYp5_pw"
}
```

## 4.4 Phase 4: Signature Creation

After the SAD has been obtained by authorizing the credential for use, the SAD can be used to sign the document's hash (Data To Be Signed Representation, DTBSR) using the `signature/signHash` endpoint of the RQES Provider. The endpoint must respond with the Signed Hash (DSV, Digital Signature Value).

****signatures/signHash:****

```mermaid
%% Signature Metadata Endpoint
  sequenceDiagram
    participant Signing Service
    participant RQES Provider
    Signing Service->>RQES Provider: POST /csc/v2/signatures/signHash
    RQES Provider->>Signing Service: Signed Hash
```

```http request
POST /csc/v2/signatures/signHash HTTP/1.1
Host: rqes.example.com
Content-Type: application/json
Authorization: Bearer 4/CKN69L8gdSYp5_pwH3XlFQZ3ndFhkXf9P2_TiHRG-bA

{
  "credentialID": "GX0112348",
  "SAD": "_TiHRG-bAH3XlFQZ3ndFhkXf9P24/CKN69L8gdSYp5_pw",
  "hashes": [
    "sTOgwOm+474gFj0q0x1iSNspKqbcse4IeiqlDg/HWuI="
  ],
  "hashAlgorithmOID": "2.16.840.1.101.3.4.2.1",
  "signAlgo": "1.2.840.113549.1.1.1",
  "clientData": "12345678"
}
```

**Sample Response:**

```json
{
  "signatures": [
    "KedJuTob5gtvYx9qM3k3gm7kbLBwVbEQRl26S2tmXjqNND7MRGtoew=="
  ]
}
```

## 4.5 Phase 5: Signed Document Formation and Retrieval

After the signing of the hash (SDR) has been completed, the AdES digital signature can be formed and the final document can be sent back to the Service Provider for further handling and/or for download by the user.

The transfer of the document to the Service Provider is out of scope of this RFC.

# 5. Reference:

1. OpenID Foundation (2023), 'OpenID for Verifiable Presentations (OID4VP)', Available at: [https://openid.net/specs/openid-4-verifiable-presentations-1_0-ID2.html](https://openid.net/specs/openid-4-verifiable-presentations-1_0-ID2.html)
2. European Commission (2023) The European Digital Identity Wallet Architecture and Reference Framework (2023-04, v1.1.0)  [Online]. Available at: [https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/releases)
3. Cloud Signing Consortium API Specification v2 (2023), Available at: [https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf](https://cloudsignatureconsortium.org/wp-content/uploads/2023/04/csc-api-v2.0.0.2.pdf)
4. ETSI TS 119 432 V1.2.1 (2020), Available at: [https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/01.02.01_60/ts_119432v010201p.pdf](https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/01.02.01_60/ts_119432v010201p.pdf)