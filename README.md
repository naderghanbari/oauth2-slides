---

# OAuth 2.0 Authorization Framework
## Nader Ghanbari
### YoppWorks 2019


---

# Abstract

- Is an **authorization** framework
- Enables a **third-party app** to obtain limited access to an **HTTP service**
  - **on behalf** of a **resource owner** by orchestrating an **approval interaction**
  - or on its own behalf
- Is designed only for `HTTP`
- Supports several flows appropriate for different topologies/scenarios
---

# History

- Developed by the *IETF* 
- Next generation of OAuth (now known as OAuth 1)
  - Unlike its predecessor OAuth 2.0 relies completely on TLS
  - No signature/encryption
    - Except for **JWT Bearer Tokens**
  - No backward compatibility
- 31 drafts between 2010 and 2012
- Finalized in [RFC 6749](https://tools.ietf.org/html/rfc6749) 
- Widely accepted and used by the community

---

# Extensions

- [Open ID Connect](https://openid.net/connect/)
  - An identity layer on top of OAuth 2.0
- [Assisted Token Flow](https://tools.ietf.org/id/draft-ideskog-assisted-token-00.html)
  - Designed to make the life easier for single-page apps
  - Could be used in normal web apps as well
  - Still in draft
  - Not implemented yet by providers and clients 
- [Device Flow](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-15)
  - Still in draft but widely used
  - For devices and embedded systems without a browser 
- [Bearer Token Usage RFC 6750](https://tools.ietf.org/pdf/rfc6750.pdf)
- [This Python library](https://github.com/lepture/authlib) implements most of these
specs and is a [good starting point](https://github.com/lepture/authlib#spec-implementations)

---

# Traditional Authentication Models

- **Client** provides **user's credentials** to access protected resources

- Several issues
  - Clients has to store the password, typically in plain-text
  - Servers need to support password authentication
  - Usually client gains an all or nothing access
  - Any intention to revoke access from one client revokes all clients
    - Only feasible way is changing the password
  - Compromise of any third-party client results in compromise of users' password     

--- 

# OAuth's proposal

- A new authorization layer
- Separating the role of **client** from that of **resource owner**
- **Access token**
  - String denoting a specific 
    - **scope**
    - **lifetime**
    - other **access attributes**
  - Issued to clients by the **authorization server** 
    - with approval of the resource owner 

---

# Example

- An **end-user** grants a **printing service** access to her **protected photos** stored at 
a **photo-sharing service**
 - Without sharing her **username and password** with the printing service. 
 - Instead, she authenticates directly with **a server trusted by the photo-sharing service**
   - That server issues the printing service **delegation-specific credentials**.

![](diagrams/001-example.svg)

---

# OAuth 2.0 Terminology

- `Resource Owner`: End-User              
- `Client`: Printing Service      
- `Authorization Server`: Trusted Server        
- `Resource Server`: Photo-Sharing Service 

- `Access Token`: Delegation-specific credentials

---

# Four Roles

- `Resource Owner`
  - An entity capable of granting access to a protected resource.
- `Client`
  - An app making protected resource requests on behalf of the
     resource owner and with its authorization
   - Just a term, does not imply the nature of the app
- `Authorization Server`
  - The server issuing access tokens to the client after successfully
     authenticating the resource owner and obtaining authorization
- `Resource Server` 
  - The server hosting the protected resources, capable of accepting and 
  responding to protected resource requests using access tokens

---

# Four Roles

- Before getting into details of each role let us look at the abstract flow

---

# Abstract Flow

![](diagrams/002-abstract-flow.svg)

___

# Step (A) 

- The client requests authorization from the resource owner. 
- The authorization request can be made 
  - directly to the resource owner (as shown) 
- or **preferably** indirectly via the authorization server as an intermediary
  - We will see this flow later

- The initial and the most important step
- Branching factor
  - The flow and consequently other steps depend on it

---

# Authorization Grant

- Credential representing the resource owner’s authorization
- Will be used by the client to obtain the final access token 

- Four standard types:
  - `authorization code`
  - `implicit`
  - `resource owner password credentials`
  - `client credentials`

- Extensible

---

# Authorization Code Flow

![](diagrams/003-authorization-code.svg)

---

# Authorization Code Flow
## Benefits
  - Resource owner only authenticates with the authorization server like other flows
  - Ability to authenticate the client
  - Transmission of the access token directly to the client without
   passing it through the resource owner’s user-agent.

# Authorization Code Flow
## Authorization Request (Step A)

- `Client` constructs a URI by adding the following params to the `authorization endpoint`
of the `Authorization Server`
- `Client` then redirects the `User Agent` to this URI

- Params 
  - `response_type`: must be `code` for this flow
  - `client_id`: id of the `client`, registered in `Authorization Server`
  - `redirect_uri`
  - `scope`: Access cope
  - `state`: An opaque value used by the client to maintain state between the request and callback

---

# Authorization Code Flow
## Authorization Request Example

`GET`

`/authorize?response_type=code&client_id={ID}&state={STATE}&redirect_uri=https://client.com/callback`

`Host: server.example.com`

---

# Authorization Code Flow
## Authorization Response (Step C)

- `Authorization Server` issues an `authorization code` and delivers it to the
`Client`
- Delivery to the `redirect_uri` specified in step A

- Params 
  - `code`
    - MUST expire shortly after it is issued 
    - maximum lifetime of 10 minutes is RECOMMENDED. 
    - The client MUST NOT it more than once. 
    - If used more than once the `Authorization Server` MUST deny the request 
  - `state`
     - If it was present in step A
     - MUST BE the exact value received from the client

---

# Authorization Code Flow
## Authorization Response Success Example

`HTTP/1.1 302 Found`
 
`Location: https://client.com/callback?code={CODE}&state={STATE}`

---

# Authorization Code Flow
## Authorization Response Errors

`HTTP/1.1 302 Found`
 
`Location: https://client.com/callback?error={ERROR_TYPE}&error_description={DESCRIPTION}&state={STATE}`

- Error types
  - `invalid_request`: missing required params
  - `unauthorized_client`: client not authorized to use this flow
  - `access_denied`: resource owner or authorization server denied the request
  - `unsupported_response_type`: flow not supported
  - `invalid_scope`: invalid, unknown, or malformed scope
  - `server_error`: unexpected server error
  - `temporarily_unavailable`: acts as a `503 Service Unavailable`
    - because `503` status cannot be returned to the client via a redirect
- `state`
  - If it was present in step A
  - MUST BE the exact value received from the client

---

# Authorization Code Flow
## Access Token Request (Step D)

- `Client` makes a `POST` request to the `token endpoint` with the following
parameters in the body

- `grant_type`: MUST be `authorization_code` for this flow
- `code`: Authorization code received in Step C
- `redirect_uri`: MUST match the value given in Step A
- `client_id`: REQUIRED if the client is confidential
  - MUST match Step A
- `client_secret`: REQUIRED if the client is confidential

- Client authentication can be done with HTTP `Basic Auth` if the client
has a password

---

# Authorization Code Flow
## Access Token Request Example

`POST /token`

`Host: server.com`

- Headers 

  - `Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW` 
    - Only if client has a `password`

  - `Content-Type: application/x-www-form-urlencoded`
 

- Body
 
  - `grant_type=authorization_code&code={CODE}&redirect_uri=https://client.com/callback&client_id={ID}&client_secret={SECRET}`
  
---

# Authorization Code Flow
## Access Token

`Authorization Server` MUST

  - require authentication for confidential clients
  - authenticate the client if authentication is included
  - ensure `authorization code` was issued to the same client
  - if the client is public ensure code was issued to "client_id" in the request
  - verify validity of the `Authorization Code` 
  - ensure `redirect_uri` is present and matches if it was included in Step A

---

# Authorization Code Flow
## Access Token Response (Step E)

`OK`
 
`Content-Type: application/json;charset=UTF-8`
`Cache-Control: no-store`
`Pragma: no-cache`

```json
{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "example",
  "expires_in": 3600,
  "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA",
  "example_parameter": "example_value"
}
```

---

# Four Roles

- Now we are ready to take a closer look at each role

---

# Resource Owner

- Usually an end-user
- All a user needs to know about the OAuth framework
  - A third-party app (`Client`) will make requests on their behalf
  - They have to provide credentials only to the source (`Authorization Server`)
  - They should never give their credentials to the third-party app
  - They can revoke the access at any time at source (`Authorization Server`)
  - Access scopes, if any, and their semantics

- Advanced users
  - How to configure an app (`Client`)
    - Client id and secret
    - Client's redirection URI, if any or if needed
    - Allowed scopes
    - The flow used by the Client
      - If multiple flows are supported by the Client

--- 

# Resource Server

- Verifying 
  - Authenticity of the `Access Token`
  - Validity of the `Access Token`
  - The scope 
    - Denying access if requested resource is beyond the scope

- The method used to validate is left to implementations
  - Most web/HTTP frameworks have support for extracting OAuth access tokens from the request
  - Token verification is usually left as an abstract operation

- Depending on the type of `Access Token` verification can be done
  - By interacting with the `Authorization Server`
  - In the `Resource Server` itself 
- See [Bearer Token Usage RFC 6750](https://tools.ietf.org/pdf/rfc6750.pdf) for more info

---

# Client
## Client Types

- `Confidential`
  - Capable of keeping confidentiality of their credentials
  - On a secure server
  - Alternatively, capable of secure authentication by other means
- `Public`
  - Incapable of maintaining the confidentiality of credentials
    - Native or on-device apps
    - Single-page or browser-based apps
  - No other means of secure authentication

---

# Client
## Client Types

- Needs to implement the full spec of the flow(s) used
- Registered in `Authorization Server`
- Needs a secure secret if `Confidential` 
- Expose a `Redirection URI` if needed by the flow 
- Has to use the `state` param to keep a state between steps of the flow
  - This is also recommended to prevent cross-site forgery (CSRF) attacks
- Securely store the `Access Token` and not expose it by any means

- Much easier to implement compared to the `Authoriztion Server`
  - A multitude of open source libraries
    - [google-oauth-client](https://github.com/googleapis/google-oauth-java-client)
    - [authlib](https://github.com/lepture/authlib)
  
---

# Authorization Server

- Needs to implement the full spec and the extensions
  - A subset of the spec if advertised as such
- Supports manual or automatic `Client` registration
- Capable of revoking `Access Token`s

---

# Refresh Token

- Credentials used to obtain access tokens.
- Issued to the `Client` by `Authorization Server`
- To obtain a new access token when the current one expires
- Optional, fully in the discretion of `Authorization Server`
- Not supported for all flows

---

# Refresh Token Abstract Flow

![](diagrams/004-refresh-token.svg)

