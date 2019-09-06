# oauth plugin

## Summary
The purpose of the oauth plugin is to allow customers to secure API requests to the Apigee Edge Microgateway (EM) with both API Key and OAuth 2.0 (JWT validation).  It is important to note that when you include this plugin in the plugins sequence that both API Key validation and OAuth 2.0 validation are allowed by default on your API request.  Typically, this is not what customers want; instead they prefer either API Key validation **or** OAuth 2.0 (JWT validation) but not both.  

## OAuth plugin links
Please review the following OAuth plugin documentation.  
* [Microgateway existing plugins](https://docs.apigee.com/api-platform/microgateway/2.5.x/use-plugins#existingpluginsbundledwithedgemicrogateway)
* [Creating entities on Apigee Edge plugin](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part2createentitiesonapigeeedge)
* [Secure the Microgateway with OAuth2.0](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part4secureedgemicrogateway)
* [Secure the Microgateway with API Key validation](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part4secureedgemicrogateway-securingtheapiwithanapikey)


## When do I use this plugin?
Use this plugin when you want to enable **both**
* API Key validation **and**
* OAuth 2.0 for the same API request.

This plugin also allows you to change the default behavior above by setting one of two properties.
* allowOAuthOnly - set to `true` to only allow OAuth 2.0 access tokens (JWTs), which disables API Key validation
* allowAPIKeyOnly - set to `true` to only allow API key validation, which disables OAuth 2.0 access tokens

## Process Summary

1. The client will obtain an client ID and secret.
2. The client will include either the API Key on the request or exchange the client ID and secret for a JWT and include it the request to the Microgateway.
3. The `oauth` plugin will validate either the API Key or the JWT.
4. If the API Key or JWT is valid, then the request will continue to the next plugin, otherwise an error message will be returned to the client application.

## Prerequisites
Please complete the following tasks before you use this plugin.  

1. [Install MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Prerequisite)   

2. [Configure MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part1)

3. [Create entities on Apigee Edge](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part2)


## Plugin configuration properties
The following properties can be set in the `oauth` stanza in the Microgateway configuration file.

```yaml
oauth:
  # header name used to send the JWT to the Microgateway
  authorization-header: "x-custom-auth-header" # defaults to Authorization
  # header name used to send the API Key to the Microgateway
  api-key-header: "x-custom-header" # defaults to x-api-key
  # set to true if you want to send the Authorization header to the target server; set to false when you want this plugin to remove the header after it is validated.
  keep-authorization-header: true # defaults to false
  # set to true if you want to enable the Microgateway to cache the API Key with JWT.
  cacheKey: true # defaults to false, which validates the API key with Apigee Edge on each request
  # number of seconds before the token is removed from the cache.
  # if you set this to 5 (seconds) then MG will check of the difference between the expiry time and the current time [abs(expiry time - current time)] is less than or equal (<=) to the grace period.  If true, then MG will remove the token from the cache.  
  gracePeriod: 5 # defaults to 0 seconds
  ## Do not set allowOAuthOnly and allowAPIKeyOnly both to true. Only one of them should be set true.
  # set to true if you want to allow OAuth 2.0 only.  This will disable API Key validation.
  allowOAuthOnly: true # defaults to false, which allows both API Key and OAuth 2.0
  # set to true if you want to allow OAuth 2.0 only.  This will disable API Key validation.
  allowAPIKeyOnly: true # defaults to false, which allows both API Key and OAuth 2.0
  # set to true to enable the Microgateway to check against the resource paths only.  In this case it ignores the proxy name check.  
  productOnly: true # defaults to false, which enables the Microgateway to check if the proxy name is included in the product.

  ## Note that if you set the tokenCacheSize, then you should also enable it (tokenCache: true)
  # set tokenCache to true if you want to cache the access token (JWT) locally
  tokenCache: true # defaults to false, which does not cache the access token.
  # set the number of tokens allowed to be cached locally
  tokenCacheSize: 150 # defaults to 100
```

## Enable the plugin
In the EM configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - oauth
      # other plugins can be listed here
```

## Configure the plugin
In the same configuration file you also need to configure the `oauth` plugin if you want to change the default behavior.  The example below changes the `oauth` plugin to allow OAuth 2.0 access tokens only (JWTs), enables access token caching and changes the `tokenCacheSize` to 150 tokens.    

```yaml
auth:
  allowOAuthOnly: true
  tokenCache: true
  tokenCacheSize: 150
```

## API Key Validation
The Microgateway exchanges the API Key for a JWT when it submits the API key to Apigee Edge to be validated.   

## Caching
### API Keys
If you set `cacheKey` to `true`, then the Microgateway will cache the JWT token that it receives in exchange for the API Key.  This prevents the Microgateway from sending a request to Apigee Edge to validate an access token on every request.

### Cache Headers
The Microgateway observes the [`cache-control`](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching) header, but only if you set it as `cache-control: no-cache`.  If you send this header in the request, then the JWTs that are received when you validate an API key will **not** be cached in the Microgateway provided that the `cacheKey` property is set to `false` in the Microgateway configuration file.  This plugin does not allow you to set the cache expiry time in the `cache-control` header. 

### Access Tokens
Access tokens (JWTs) can also be cached in the Microgateway to avoid validating the JWT on every request.  Access tokens are only cached if you set `tokenCache` to `true` and the JWT is valid.  

## Best Practices for configuring this plugin
* The oauth plugin is typically listed first in the plugin sequence.  
* Most customers only want to allow either API Key validation or access token validation, but not both at the same time.
* If you need one set of APIs to be validated by JWTs and another set to be validated by API Keys (lower security), then consider the following:
  * create two products, one for API Key validation and one set for access token (JWT) validation, then configure two sets for Microgateway instances - one set for API key validation and one set for access token validation.
  * routing between these Microgateway instances should be handled with a HTTPS load balancer.  
