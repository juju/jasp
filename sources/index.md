# Ubuntu Juju Application Security Protocol (JASP) Relationship Overview

![JASP Stack Overview Diagram](https://raw.githubusercontent.com/juju/jasp/master/img/jasp_stack_diagram.png =600x)

In the above diagram:

*  **Fuscia** [SCIM](http://simplecloud.info) interface or component

*  **Green** [UMA](http://tinyurl.com/umav1) interface or component

*  **Red** web application, native application, or headless API server

*  **Blue** [OpenID Connect](http://openid.net/connect/) interface or component

*  **Grey** Out of scope for JASP

## Relationship from OpenID Connect Relying Party (RP) to OpenID Provider (OP)

OpenID Connect defines HTTPS APIs that enable a website or mobile application--a "Relying Party" or RP--to identify a person in their home domain. It's like Google or Yahoo authentication for any domain on the Internet that hosts a compliant OpenID Provider (which is no more complex than DNS or email).

The RP can expect the OP of any compliant Internet domain to provide all the things required by the [OpenID Connect standard](http://openid.net/connect). The exact OpenID Connect endpoints, supported crypto and other operational information will be published by the OP at ''.well-known/openid-configuration''

The RP may register a callback to receive a notification of a user logout event. This notification is not guaranteed to be delivered to an RP.

Relationship Name: openid-connect

Relationship Direction:

      * From RP --> OP

Interface: [OpenID Connect](http://openid.net/connect)

Token: Access Token required to obtain id_token from userinfo endpoint

RP Sets:

      * **connect_discovery_url**  - Should be relied on exclusively **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**
      * **connect_logout_url_callback** Best efforts notification, probably from the person's browser. You may not get this, but its greener if you do.
      *     **op_host** Either the domain or hostname of the OP
      *     **op_port** HTTPS port : usually 443
      *     **client_id** Explicitly set the client_id
      *     **client_secret**  Explicitly set the client_secret
      *     **client_jks_keystore** Should contain the certificates and private key for the RP.
      *     **client_jks_truststore** May contain additional CA's to enable crypto verification.
      *     See [OpenID Connect Discovery Response](http://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationResponse)

##  Relationship from SCIM Client to Server

[SCIM](http://simplecloud.info) provides a standard that reduces the number of interfaces developers need to learn for user, group and device CRUD.

Relationship Direction:

      * From SCIM Client (or IDM) --> Persistence Layer Supporting OAuth2 services


Interface: Not Specified (could be LDAP, RDBMS, or any other persistence technology)

Token: Client must register SCIM endpoints, and enforce authorized UMA RPT token: same as RS -> AS

RP Sets:

      *     uma_discovery_url **//''REQUIRED''//**
      * scim_discovery_url - where to find the SCIM endpoints, required cryto, and required UMA authorization scopes **//''Required''//**
      *     connect_discovery_url **//''REQUIRED''//**
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

      *     uma_discovery_url **//''REQUIRED''//**
      *     connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


##  Relationship from UMA Client to UMA Resource Server (RS)

An UMA Client is a Web application or native application that requests an HTTP resource (i.e. an API or binary) from an UMA Resource Server. UMA clients can be registered and managed using the OpenID Connect client registration API. The client must use  UMA discovery published at ''.well-known/uma-configuration'' in order to obtain the required tokens to access an UMA protected resource.

Relationship Direction:

      * From Client --> RS

Interface: [UMA](http://tinyurl.com/umav1)

Token: Client must present an authorized RPT token to the RS

RP Sets:

      *     uma_discovery_url **//''REQUIRED''//**
      *     connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


##  Relationship from UMA Client to UMA Authorization Server (AS)

An UMA Client is a Web application or native application that requests an HTTP resource (i.e. an API or binary) from an UMA Resource Server. UMA clients can be registered and managed using the OpenID Connect client registration API. The client must use  UMA discovery published at ''.well-known/uma-configuration'' in order to obtain the required tokens to access an UMA protected resource.

Relationship Direction:

      * From Client --> AS

Interface: [UMA](http://tinyurl.com/umav1)

Token: Client must present AAT token to communicate with AS (obtain RPT, authorize RPT)

RP Sets:

      *     uma_discovery_url **//''REQUIRED''//**
      *     connect_discovery_url **//''REQUIRED''//**
      * **connect_redirect_url** - Where to find the application, i.e. ''https://example.com/myFolder/index.html'' **//''REQUIRED''//**


