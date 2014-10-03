# Ubuntu Juju Application Security Protocol (JASP) Relationship Overview

![JASP Stack Overview Diagram](https://raw.githubusercontent.com/juju/jasp/master/img/jasp_stack_diagram.png =600x)

In the above diagram:

 *  **Blue** [OpenID Connect](http://openid.net/connect/) interface or component

 *  **Green** [UMA](http://tinyurl.com/umav1) interface or component

 *  **Fuscia** [SCIM](http://simplecloud.info) interface or component

 *  **Red** web application, native application, or headless API server

 *  **Grey** Out of scope for JASP

## Relationship from OpenID Connect Relying Party (RP) to OpenID Provider (OP)

An OpenID Connect Relying Party, or RP, calls the APIs of the OpenID Provider to identify a 
person at their home domain.  [OpenID Connect API](http://openid.net/connect) is a profile 
of OAuth2 which defines APIs for authentication, discovery, client registration and session 
management. 

### Relationship Name: 

    *  OPENID-CONNECT

### Relationship Direction:

    *  From RP to OP

### Overview of Web Browser Authentication:

 0. Person hits login button on Website which is the "RP" (Relying Party).

 1. RP sends discovery to https://host/.well-known/openid-configuration
       See [OpenID Connect Client Registration API](http://openid.net/specs/openid-connect-discovery-1_0.html)

 2. RP sends client registration request: receives response with clientID 
       and other registration details. Application should stores this json 
       for later. Client Registration request must include redirect-uri and 
       client name. See 
       [OpenID Connect Dynamic Registration API](http://openid.net/specs/openid-connect-registration-1_0.html)

 3. RP re-directs Person for authentication / authorization (note if client
       is already registered, it would start here). See [OpenID Connect Core API](http://openid.net/specs/openid-connect-core-1_0.html)

 4. RP gets access token

 5. RP sends access token to request id_token with user claims. Only present user 
       claims is `sub` as `openid` is the only scope required in OpenID Connect for 
       a dynamically registered client.

 5. Person logs out of application: all OpenID Connect tabs in browser are logged out.

### RP Configuration parameters

#### Required:

 * **JASP_OPENID_RP_CLIENT_NAME** This is a more friendly name for your application that 
    would make it easier for the admin at the OP to recognize your application. 

 * **JASP_OPENID_RP_REDIRECT_URI** The callback URI for your application, this is where
   the OpenID Connect Provider will send the access token and `id_token`. Even if a client
   can spoof a request, the response from the OP always goes back to the pre-registered
   callback redirect URI.

#### Optional 

 * **JASP_OPENID_REQUESTED_SCOPES** The RP can use this parameter to request additional 
   [OpenID Connect scopes](http://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims). 
   Many applications will not work with the username alone (i.e. the OpenID Connect `sub` claim,
   which is contained in the `openid` scope). 

 * **JASP_OPENID_REQUESTED_ACRS** An OpenID Connect RP may request a specific type of authentication
   using the `acr_values` parameter. See the 
   [Authentication Request](http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest) 
   for more details.


##  Relationship from SCIM Client to Server

[SCIM](http://simplecloud.info) provides a standard that reduces the number of interfaces developers need to learn for user, group and device CRUD.

Relationship Direction:

      * From SCIM Client (or IDM) --> Persistence Layer Supporting OAuth2 services


Interface: Not Specified (could be LDAP, RDBMS, or any other persistence technology)

Token: Client must register SCIM endpoints, and enforce authorized UMA RPT token: same as RS -> AS

RP Sets:

      * uma_discovery_url **//''REQUIRED''//**
      * scim_discovery_url - where to find the SCIM endpoints, required cryto, and required UMA authorization scopes **//''Required''//**
      * connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


While dynamic registration can be very efficient, some applications need to add an account before ther person actually logs in. For example, your account needs to be created in a payroll system before you can be paid, email accounts need to be created in advance, etc. The Access Management system fits into this category--it needs to be kept up-to-date with the latest identities, user claims and credentials.

A JASP complain SCIM service should support Web Finger discovery, and return a ''json'' object for  ''./well-known/scim-configuration'' The returned json discovery object should contain the endpoints, and required authorization servers and UMA scopes required for access.


##  Relationship from UMA Resource Server (RS) to UMA Authorization Server (AS)

A Resource Server must:

      * register all resources that are under protection with the Authorization Server
      * forbid access to resources if RPT doesn't have enough permissions and provide permission ticket to RP (so RP can authorize RPT with necessary permissions)

Relationship Direction:

      * From RS --> AS

Interface: [UMA](http://tinyurl.com/umav1)

Token: RS must present PAT token to  register resources as well as verify the permissions
in the RPT token presented by the Client.

RP Sets:

      * uma_discovery_url **//''REQUIRED''//**
      * connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


##  Relationship from UMA Client to UMA Resource Server (RS)

An UMA Client is a Web application or native application that requests an HTTP resource (i.e. an API or binary) from an UMA Resource Server. UMA clients can be registered and managed using the OpenID Connect client registration API. The client must use  UMA discovery published at ''.well-known/uma-configuration'' in order to obtain the required tokens to access an UMA protected resource.

Relationship Direction:

      * From Client --> RS

Interface: [UMA](http://tinyurl.com/umav1)

Token: Client must present an authorized RPT token to the RS

RP Sets:

      * uma_discovery_url **//''REQUIRED''//**
      * connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


##  Relationship from UMA Client to UMA Authorization Server (AS)

An UMA Client is a Web application or native application that requests an HTTP resource (i.e. an API or binary) from an UMA Resource Server. UMA clients can be registered and managed using the OpenID Connect client registration API. The client must use  UMA discovery published at ''.well-known/uma-configuration'' in order to obtain the required tokens to access an UMA protected resource.

Relationship Direction:

      * From Client --> AS

Interface: [UMA](http://tinyurl.com/umav1)

Token: Client must present AAT token to communicate with AS (obtain RPT, authorize RPT)

RP Sets:

      * uma_discovery_url **//''REQUIRED''//**
      * connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


