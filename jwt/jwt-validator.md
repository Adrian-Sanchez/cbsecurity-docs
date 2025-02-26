# JWT Validator

Now that we have all the pieces in place for JWT, we can now register the JWT validator as our validator of choice for requests which in our case it is the same JWT Service that will take care of the validation: `JWTService@cbsecurity`. 

The validator will inspect the incoming requests for valid jwt authorization headers. It will be in charge of verifying their expiration, their required claims, and the user it represents. Once that is done, it goes in the same rule/annotation security flow that **cbsecurity** leverages.

{% code title="config/Coldbox.cfc" %}
```javascript
cbsecurity = {
    validator = "JWTService@cbsecurity"
}
```
{% endcode %}

## Module Override

Each module can also override their validator via it's configuration setting `cbsecurity.validator`. So if the global validator is something other than jwt but your module REQUIRES JWT validation, then just add it in your `ModuleConfig.cfc`

```javascript
settings = {
    cbsecurity = {
         validator = "JWTService@cbsecurity"
    }
}
```

## JWT Token Discovery

The JWT validator will discover the incoming JWT token from 3 sources:

1. `authorization` header using the bearer token approach
2. Custom header configured in your settings: `cbsecurity.customAuthHeader`
3. Incoming `rc` variable with the same name as `cbsecurity.customAuthHeader`

## Token Scopes & Permissions

If your rules have the `permissions` element or your `secure` annotations have context, then we will treat those as the scopes/permissions to check the user/token must have at validation.

{% embed url="https://auth0.com/docs/scopes/current" %}

## Validator Process

The validator will have the following validation process:

* Verify the jwt token exists via the `authorization` header or custom header `x-auth-token` or incoming `rc[ 'x-auth-token' ]`
* Verify we can decode it
* Verify if it has not expired from the token itself
  * If you have enabled auto refresh tokens, check out the [refresh tokens process](refresh-tokens.md#enableautorefreshvalidator).
* Verify it has the required claims
* If token storage is enabled, verify the token in the permanent storage
* Verify the subject \(`sub`\) claim and try to retrieve the user it represents
* Try to authenticate the user for the request
* Verify the subject has the right permissions or the token has the right scopes attached to it.
* If all is valid then place the token in `prc.jwt_token` and the payload in `prc.jwt_payload`
* If all is valid then place the user object in `prc.oCurrentUser` or the variable of your choice via the `cbsecurity.prcUserVariable` setting.
* Continue or block

That's it!  You can create your rules and annotations just like your used to, but now the validator will make sure valid JWT tokens are passed for those requests.

