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
  - 31 drafts between 2010 and 2012
  - Finalized in [RFC 6749](https://tools.ietf.org/html/rfc6749) 
  - Widely accepted and used by the community

- Next generation of OAuth (now known as OAuth 1)
  - Unlike its predecessor OAuth 2.0 relies completely on TLS
  - No signature/encryption
    - Except for **JWT Bearer Tokens**
  - No backward compatibility
  
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

## Authorization Code Flow

![](diagrams/003-authorization-code.svg)

---

## Authorization Code Flow
   
- Benefits
  - Resource owner only authenticates with the authorization server like other flows
  - Ability to authenticate the client
  - Transmission of the access token directly to the client without
   passing it through the resource owner’s user-agent.
