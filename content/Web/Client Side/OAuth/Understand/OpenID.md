## What is OpenID Connect?
OpenID Connect extends the OAuth protocol to provide a dedicated identity and authentication layer that sits on top of the [basic OAuth implementation](https://portswigger.net/web-security/oauth#how-does-oauth-2-0-work). It adds some simple functionality that enables better support for the authentication use case of OAuth.

OAuth was not initially designed with authentication in mind; it was intended to be a means of delegating authorizations for specific resources between applications. However, many websites began customizing OAuth for use as an authentication mechanism. To achieve this, they typically requested read access to some basic user data and, if they were granted this access, assumed that the user authenticated themselves on the side of the OAuth provider.

These plain [OAuth authentication](https://portswigger.net/web-security/oauth#oauth-authentication) mechanisms were far from ideal. For a start, the client application had no way of knowing when, where, or how the user was authenticated. As each of these implementations was a custom workaround of sorts, there was also no standard way of requesting user data for this purpose. To support OAuth properly, client applications would have to configure separate OAuth mechanisms for each provider, each with different endpoints, unique sets of scopes, and so on.

OpenID Connect solves a lot of these problems by adding standardized, identity-related features to make authentication via OAuth work in a more reliable and uniform way.

---
## How does OpenID Connect work?
OpenID Connect slots neatly into the normal [OAuth flows](https://portswigger.net/web-security/oauth/grant-types). From the client application's perspective, the key difference is that there is an additional, standardized set of scopes that are the same for all providers, and an extra response type: `id_token`.

### OpenID Connect roles
The roles for OpenID Connect are essentially the same as for standard OAuth. The main difference is that the specification uses slightly different terminology.
- **Relying party** - The application that is requesting authentication of a user. This is synonymous with the OAuth client application.
- **End user** - The user who is being authenticated. This is synonymous with the OAuth resource owner.
- **OpenID provider** - An OAuth service that is configured to support OpenID Connect.

### OpenID Connect claims and scopes

The term "claims" refers to the `key:value` pairs that represent information about the user on the resource server. One example of a claim could be `"family_name":"Montoya"`.

Unlike basic OAuth, whose [scopes are unique to each provider](https://portswigger.net/web-security/oauth/grant-types#oauth-scopes), all OpenID Connect services use an identical set of scopes. In order to use OpenID Connect, the client application must specify the scope `openid` in the authorization request. They can then include one or more of the other standard scopes:
- `profile`
- `email`
- `address`
- `phone`

Each of these scopes corresponds to read access for a subset of claims about the user that are defined in the OpenID specification. For example, requesting the scope `openid profile` will grant the client application read access to a series of claims related to the user's identity, such as `family_name`, `given_name`, `birth_date`, and so on.

### ID token

The other main addition provided by OpenID Connect is the `id_token` response type. This returns a JSON web token (JWT) signed with a JSON web signature (JWS). The JWT payload contains a list of claims based on the scope that was initially requested. It also contains information about how and when the user was last authenticated by the OAuth service. The client application can use this to decide whether or not the user has been sufficiently authenticated.

The main benefit of using `id_token` is the reduced number of requests that need to be sent between the client application and the OAuth service, which could provide better performance overall. Instead of having to get an access token and then request the user data separately, the ID token containing this data is sent to the client application immediately after the user has authenticated themselves.

Rather than simply relying on a trusted channel, as happens in basic OAuth, the integrity of the data transmitted in an ID token is based on a JWT cryptographic signature. For this reason, the use of ID tokens may help protect against some man-in-the-middle attacks. However, given that the cryptographic keys for signature verification are transmitted over the same network channel (normally exposed on `/.well-known/jwks.json`), some attacks are still possible.

Note that multiple response types are supported by OAuth, so it's perfectly acceptable for a client application to send an authorization request with both a basic OAuth response type and OpenID Connect's `id_token` response type:

```
response_type=id_token token
response_type=id_token code
```

In this case, both an ID token and either a code or access token will be sent to the client application at the same time.

---
## Identifying OpenID Connect

If OpenID connect is actively being used by the client application, this should be obvious from the authorization request. The most foolproof way to check is to look for the mandatory `openid` scope.

Even if the login process does not initially appear to be using OpenID Connect, it is still worth checking whether the OAuth service supports it. You can simply try adding the `openid` scope or changing the response type to `id_token` and observing whether this results in an error.

As with basic OAuth, it's also a good idea to take a look at the OAuth provider's documentation to see if there's any useful information about their OpenID Connect support. You may also be able to access the configuration file from the standard endpoint `/.well-known/openid-configuration`.