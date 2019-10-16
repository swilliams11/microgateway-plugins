# AccessControl

## Summary
The `accesscontrol` plugin provides IP filtering to Edge Microgateway. With this plugin, users can whitelist and/or blacklist IP addresses.

## When to use this plugin?

Use this plugin when you want to allow or deny specific IP addresses or allow or deny IP address ranges.

## Process summary

The following steps describe how the plugin operates in an API request flow assuming that this plugin is listed first.

1. The client obtains a client ID and secret.
2. The client includes either:
   * the API Key on each API request or
   * exchanges the client ID and secret for a JWT and includes the JWT in each request to Edge Microgateway.
3. The `accesscontrol` plugin determines to either allow or deny the IP address.
4. If the IP address is in the deny list, then an error message is returned to the client.  


## Plugin configuration properties

You can set the following properties in the `accesscontrol` stanza in the Edge Microgateway configuration file.

```yaml
accesscontrol:
  # use the allow property to list the allowed IP addresses.
  # Default: none

  allow:
    - x.x.x.x
    - x.*.x.*

  # use the deny property to list the IP addresses that should not be allowed.

  deny:
    - x.x.x.*
```

## Enable the plugin
In the Edge Microgateway configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - oauth
      - accesscontrol
      # other plugins can be listed here
```

## Configure the plugin
```
accesscontrol:
	allow:
	    - 10.10.10.10
	    - 11.*.11.*
	deny:
	    - 12.12.12.*
```

## Best Practices for configuring this plugin
* The `accesscontrol` plugin can be listed before the `oauth` plugin or after it.  
* Configure a specific IP address or configure use a `*` to denote any value for the specific octet.  
* TODO
    * update this README after issue #147 is complete.  
    * What happens if your specific IP is listed in both the allow and deny list?
    * What happens if you have overlapping IP address ranges in the allow and deny list?
