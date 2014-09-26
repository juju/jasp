# Proposed Interop Outline

## Interop 1 : Basic OpenID Connect 

  * The website will test that it can use a Juju relation to automatically switch from one OP to another. 
    Ideally several different web applications will be tested.

  * OP must support OpenID Connect Discovery and Dynamic Client Registration
  
  * OP must release the `openid` scope with the `sub` user claim
  
  * OP must support Session Management to show logout across multiple tabs in the browser. 

## Interop 2 : Advanced OpenID Connect

  * Configure Native RP : (1) PAM Unix ssh login (2) mobile application
  
  * RP Certificate Authentication
  
  * Support for client requesting strong user authentication (using acr_values param in OpenID Connect)

## Interop 3 : UMA

  * Resource Registration
  
  * Support for native application calling protected API
  
  * Support for web container folder protection using single UMA AS and required scopes for HTTP methods GET, POST, PUT, DELETE

## Interop 3 : Advanced UMA 

  * Trust elevation

  * Mutli-AS authorization

## Interop 4 : SCIM

  * SCIM 1.0 User API
  
