# oauth plugin

## Summary
The OAuth plugin allows customers to secure API requests to Apigee Edge Microgateway with both API Key and OAuth 2.0 (JWT validation). Note that, buy default, when you include this plugin in the plugins sequence, you can use both API Key validation and OAuth 2.0 validation on API requests. However, you can change the default behavior to allow either API key validation **or** OAuth 2.0 (JWT) validation. 

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

1. The client will obtain a client ID and secret.
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
  authorization-header: "x-custom-auth-header" # defaults to Authorization: Bearer
  # header name used to send the API Key to the Microgateway
  api-key-header: "x-custom-header" # defaults to x-api-key
  # set to true if you want to send the Authorization header to the target server; set to false when you want this plugin to remove the header after it is validated.
  keep-authorization-header: true # defaults to false
  # set to true if you want to enable the Microgateway to cache the JWT that is received when the API Key is validated.
  cacheKey: true # defaults to false, which validates the API key with Apigee Edge on each request
  # number of seconds before the token is removed from the cache.
  # if you set this to 5 (seconds) then the Microgateway will check if the difference between the expiry time and the current time [abs(expiry time - current time)] is less than or equal (<=) to the grace period.  If true, then the Microgateway will remove the token from the cache.  
  gracePeriod: 5 # defaults to 0 seconds
  ## Do not set allowOAuthOnly and allowAPIKeyOnly both to true. Only one of them should be set true.
  # set to true if you want to allow OAuth 2.0 only.  This will disable API Key validation.
  allowOAuthOnly: true # defaults to false, which allows both API Key and OAuth 2.0
  # set to true if you want to allow OAuth 2.0 only.  This will disable API Key validation.
  allowAPIKeyOnly: true # defaults to false, which allows both API Key and OAuth 2.0
  # set to true to enable the Microgateway to check against the resource paths only.  In this case it ignores the proxy name check.  
  productOnly: true # defaults to false, which enables the Microgateway to check if the proxy name is included in the product.

  ## Note that if you set the tokenCacheSize, then you should also enable it (tokenCache: true)
  # set tokenCache to true if you want to cache the access token (JWT) that is received after the API key is validated.
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
In the same configuration file you also need to configure the `oauth` plugin if you want to change the default behavior.  The example below changes the `oauth` plugin to allow OAuth 2.0 access tokens (JWTs) only, enables access token caching and changes the `tokenCacheSize` to 150 tokens.    

```yaml
oauth:
  allowOAuthOnly: true
  tokenCache: true
  tokenCacheSize: 150
```

## API Key Validation
The Microgateway exchanges the API Key for a JWT when it validates the API key with Apigee Edge.   

## Caching
### API Keys
If you set `cacheKey` to `true`, then the Microgateway will cache the JWT token that it receives in exchange for the API Key.  This prevents the Microgateway from sending a request to Apigee Edge to validate the API Key on every request.

### Cache Headers
The Microgateway observes the [`cache-control`](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching) header, but in a limited fashion.  If a client application wants to cache the JWT that is received after the API Key is validated, then it should set the `cache-control` header to any value except `no-cache`.  If you send this header in the request (`cache-control: max-age=120`), then the JWTs will be cached in the Microgateway even if the `cacheKey` is set to `false`.  If you set `cacheKey` to `true` and you send a request to the Microgateway with `cache-control: no-cache`, the JWT **will** be cached anyway.  Client applications are not allowed to override the `cacheKey` setting with the `cache-control` header.  This plugin does not allow you to set the cache expiry time in the `cache-control` header; it uses the expiry time in the JWT to determine the TTL of the cache.  

### Access Tokens
Access tokens (JWTs) can also be cached in the Microgateway to avoid validating the JWT on every request.  Access tokens are only cached if you set `tokenCache` to `true` and the JWT is valid.  

## Best Practices for configuring this plugin
* The oauth plugin is typically listed first in the plugin sequence.  
* Most customers only want to allow either API Key validation or access token validation, but not both at the same time.
* Do not set `allowOAuthOnly` and `allowAPIKeyOnly` both to true. Only one of them should be set true.
* Consider using the `oauthv2` or `apikeys` plugins instead of this plugin.  
* If you need one set of APIs to be validated by JWTs and another set to be validated by API Keys (lower security), then consider the following:
  * create two products, one for API Key validation and one set for access token (JWT) validation, then configure two sets for Microgateway instances - one set for API key validation and one set for access token validation.
  * routing between these Microgateway instances should be handled with a HTTPS load balancer.  
