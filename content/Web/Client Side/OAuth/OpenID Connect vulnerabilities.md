- **Unprotected dynamic client registration**:
- **Allowing authorization requests by reference**:

---
The specification for OpenID Connect is much stricter than that of basic OAuth, which means there is generally less potential for quirky implementations with glaring vulnerabilities. That said, as it is just a layer that sits on top of OAuth, the client application or OAuth service may still be vulnerable to some of the OAuth-based attacks we looked at earlier. In fact, you might have noticed that all of our [OAuth authentication labs](https://portswigger.net/web-security/all-labs#oauth-authentication) also use OpenID Connect.

In this section, we'll look at some additional vulnerabilities that may be introduced by some of the extra features of OpenID Connect.

### Unprotected dynamic client registration

The OpenID specification outlines a standardized way of allowing client applications to register with the OpenID provider. If dynamic client registration is supported, the client application can register itself by sending a `POST` request to a dedicated `/registration` endpoint. The name of this endpoint is usually provided in the configuration file and documentation.

In the request body, the client application submits key information about itself in JSON format. For example, it will often be required to include an array of whitelisted redirect URIs. It can also submit a range of additional information, such as the names of the endpoints they want to expose, a name for their application, and so on. A typical registration request may look something like this:

you may find it in `/.well-known/openid-configuration`,`/openid/register`

```http
POST /openid/register HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: oauth-authorization-server.com
Authorization: Bearer ab12cd34ef56gh89

{
    "application_type": "web",
    "redirect_uris": [
        "https://client-app.com/callback",
        "https://client-app.com/callback2"
        ],
    "client_name": "My Application",
    "logo_uri": "https://client-app.com/logo.png",
    "token_endpoint_auth_method": "client_secret_basic",
    "jwks_uri": "https://client-app.com/my_public_keys.jwks",
    "userinfo_encrypted_response_alg": "RSA1_5",
    "userinfo_encrypted_response_enc": "A128CBC-HS256",
    …
}
```

The OpenID provider should require the client application to authenticate itself. In the example above, they're using an HTTP bearer token. However, some providers will allow dynamic client registration without any authentication, which enables an attacker to register their own malicious client application. This can have various consequences depending on how the values of these attacker-controllable properties are used.

For example, you may have noticed that some of these properties can be provided as URIs. If any of these are accessed by the OpenID provider, this can potentially lead to second-order SSRF vulnerabilities unless additional security measures are in place.

---
### Allowing authorization requests by reference

Up to this point, we've looked at the standard way of submitting the required parameters for the authorization request i.e. via the query string. Some OpenID providers give you the option to pass these in as a JSON web token (JWT) instead. If this feature is supported, you can send a single `request_uri` parameter pointing to a JSON web token that contains the rest of the OAuth parameters and their values. Depending on the configuration of the OAuth service, this `request_uri` parameter is another potential vector for SSRF.

You might also be able to use this feature to bypass validation of these parameter values. Some servers may effectively validate the query string in the authorization request, but may fail to adequately apply the same validation to parameters in a JWT, including the `redirect_uri`.

To check whether this option is supported, you should look for the `request_uri_parameter_supported` option in the configuration file and documentation. Alternatively, you can just try adding the `request_uri` parameter to see if it works. You will find that some servers support this feature even if they don't explicitly mention it in their documentation.