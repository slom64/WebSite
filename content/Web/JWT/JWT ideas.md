- **Accepting arbitrary signatures**: The server doesn't validate the signature.
- **Accepting tokens with no signature**: Put the alg: "none"
- **Brute-forcing secret keys**: 
- **JWT header parameter injections**
	- **Injecting self-signed JWTs via the jwk parameter**: Add custom public key on `jwk` header, servers may append jwk, which may reused to check the signature.
	- **Injecting self-signed JWTs via the jku parameter**: Add custom public key in `jku` paramater, which you should host to be publicly available to victim server. refere to [[Web/SSRF/1. Ideas|SSRF bypass]].
	- **Injecting self-signed JWTs via the kid parameter**: `kid` maybe a file or database entry id. so you may able to write on any of those to generate your public-private key pair.
- **Algorithm confusion attacks**:
	- **Classic cofusion attack**: Change the alg from RSA to HS256, and sign the token based on that. 
	- **Deriving public keys from existing tokens**: If the server doesn't show its public keys, we can derive them from existing tokens to preform the same attack.


---

### Accepting arbitrary signatures
JWT libraries typically provide one method for verifying tokens and another that just decodes them. For example, the Node.js library `jsonwebtoken` has `verify()` and `decode()`.

Occasionally, developers confuse these two methods and only pass incoming tokens to the `decode()` method. This effectively means that the application doesn't verify the signature at all.

---

### Accepting tokens with no signature
Among other things, the JWT header contains an `alg` parameter. This tells the server which algorithm was used to sign the token and, therefore, which algorithm it needs to use when verifying the signature.

```json
{ 
	"alg": "HS256", 
	"typ": "JWT" 
}
```

This is inherently flawed because the server has no option but to implicitly trust user-controllable input from the token which, at this point, hasn't been verified at all. In other words, an attacker can directly influence how the server checks whether the token is trustworthy.

JWTs can be signed using a range of different algorithms, but can also be left unsigned. In this case, the `alg` parameter is set to `none`, which indicates a so-called "unsecured JWT". Due to the obvious dangers of this, servers usually reject tokens with no signature. However, as this kind of filtering relies on string parsing, you can sometimes bypass these filters using classic obfuscation techniques, such as mixed capitalization and unexpected encodings.

> [!attention] 
> Even if the token is unsigned, the payload part must still be terminated with a trailing dot.

---
### Brute-forcing secret keys

Some signing algorithms, such as HS256 (HMAC + SHA-256), use an arbitrary, standalone string as the secret key. Just like a password, it's crucial that this secret can't be easily guessed or brute-forced by an attacker. Otherwise, they may be able to create JWTs with any header and payload values they like, then use the key to re-sign the token with a valid signature.

When implementing JWT applications, developers sometimes make mistakes like forgetting to change default or placeholder secrets. They may even copy and paste code snippets they find online, then forget to change a hardcoded secret that's provided as an example. In this case, it can be trivial for an attacker to brute-force a server's secret using a [wordlist of well-known secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list).

We recommend using hashcat to brute-force secret keys. You can [install hashcat manually](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_do_i_install_hashcat), but it also comes pre-installed and ready to use on Kali Linux.

You just need a valid, signed JWT from the target server and a [wordlist of well-known secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list). You can then run the following command, passing in the JWT and wordlist as arguments:

```sh
hashcat -a 0 -m 16500 <jwt> <wordlist>
```

Hashcat signs the header and payload from the JWT using each secret in the wordlist, then compares the resulting signature with the original one from the server. If any of the signatures match, hashcat outputs the identified secret in the following format, along with various other details:

As hashcat runs locally on your machine and doesn't rely on sending requests to the server, this process is extremely quick, even when using a huge wordlist.

Once you have identified the secret key, you can use it to generate a valid signature for any JWT header and payload that you like. For details on how to re-sign a modified JWT in Burp Suite, see [Editing JWTs](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#editing-jwts).

---
### JWT header parameter injections
According to the JWS specification, only the `alg` header parameter is mandatory. In practice, however, JWT headers (also known as JOSE headers) often contain several other parameters. The following ones are of particular interest to attackers.
- `jwk` (JSON Web Key) - Provides an embedded JSON object representing the key.
- `jku` (JSON Web Key Set URL) - Provides a URL from which servers can fetch a set of keys containing the correct key.
- `kid` (Key ID) - Provides an ID that servers can use to identify the correct key in cases where there are multiple keys to choose from. Depending on the format of the key, this may have a matching `kid` parameter.
As you can see, these user-controllable parameters each tell the recipient server which key to use when verifying the signature.

#### Injecting self-signed JWTs via the jwk parameter

The JSON Web Signature (JWS) specification describes an optional `jwk` header parameter, which servers can use to embed their public key directly within the token itself in JWK format.

You can see an example of this in the following JWT header:

```json
{
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "typ": "JWT",
    "alg": "RS256",
    "jwk": {
        "kty": "RSA",
        "e": "AQAB",
        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
    }
}
```


Ideally, servers should only use a limited whitelist of public keys to verify JWT signatures. However, misconfigured servers sometimes use any key that's embedded in the `jwk` parameter.

You can exploit this behavior by signing a modified JWT using your own RSA private key, then embedding the matching public key in the `jwk` header.

Although you can manually add or modify the `jwk` parameter in Burp, the [JWT Editor extension](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd) provides a useful feature to help you test for this vulnerability:

1. With the extension loaded, in Burp's main tab bar, go to the **JWT Editor Keys** tab.
2. [Generate a new RSA key.](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#adding-a-jwt-signing-key)
3. Send a request containing a JWT to Burp Repeater.
4. In the message editor, switch to the extension-generated **JSON Web Token** tab and [modify](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#editing-jwts) the token's payload however you like.
5. Click **Attack**, then select **Embedded JWK**. When prompted, select your newly generated RSA key.
6. Send the request to test how the server responds.

You can also perform this attack manually by adding the `jwk` header yourself. However, you may also need to update the JWT's `kid` header parameter to match the `kid` of the embedded key. The extension's built-in attack takes care of this step for you.

#### Injecting self-signed JWTs via the jku parameter

Instead of embedding public keys directly using the `jwk` header parameter, some servers let you use the `jku` (JWK Set URL) header parameter to reference a JWK Set containing the key. When verifying the signature, the server fetches the relevant key from this URL.

##### JWK Set

A JWK Set is a JSON object containing an array of JWKs representing different keys. You can see an example of this below.

```json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```

JWK Sets like this are sometimes exposed publicly via a standard endpoint, such as `/.well-known/jwks.json`.

More secure websites will only fetch keys from trusted domains, but you can sometimes take advantage of URL parsing discrepancies to bypass this kind of filtering. We covered some [examples of these](https://portswigger.net/web-security/ssrf#ssrf-with-whitelist-based-input-filters) in our topic on SSRF.

then put in the JWT header the jku header points to exploit server:
```json
{
    "jku": "https://exploit-0a98002c04edca9480a7a7ef01580004.exploit-server.net/.well-known/jwks.json",
    "kid": "11e55fae-9ef4-4a14-83cc-de01cbcd4c09",
    "typ": "JWT",
    "alg": "RS256"
}
```

#### Injecting self-signed JWTs via the kid parameter

Servers may use several cryptographic keys for signing different kinds of data, not just JWTs. For this reason, the header of a JWT may contain a `kid` (Key ID) parameter, which helps the server identify which key to use when verifying the signature.

Verification keys are often stored as a JWK Set. In this case, the server may simply look for the JWK with the same `kid` as the token. However, the JWS specification doesn't define a concrete structure for this ID - it's just an arbitrary string of the developer's choosing. For example, they might use the `kid` parameter to point to a particular entry in a database, or even the name of a file.

If this parameter is also vulnerable to directory traversal, an attacker could potentially force the server to use an arbitrary file from its filesystem as the verification key.

```json
{
    "kid": "../../../../dev/null",
    "typ": "JWT",
    "alg": "HS256",
    "k": "asGsADas3421-dfh9DGN-AFDFDbasfd8-anfjkvc"
}
// Then press attack --> "sign with empty key". 
// If you can't use "/dev/null" file. you can try to use any file accessable for you and generate public-private key based on it.
```

This is especially dangerous if the server also supports JWTs signed using a [symmetric algorithm](https://portswigger.net/web-security/jwt/algorithm-confusion#symmetric-vs-asymmetric-algorithms). In this case, an attacker could potentially point the `kid` parameter to a predictable, static file, then sign the JWT using a secret that matches the contents of this file.

You could theoretically do this with any file, but one of the simplest methods is to use `/dev/null`, which is present on most Linux systems. As this is an empty file, reading it returns an empty string. Therefore, signing the token with a empty string will result in a valid signature.

If the server stores its verification keys in a database, the `kid` header parameter is also a potential vector for SQL injection attacks.

#### Other interesting JWT header parameters

The following header parameters may also be interesting for attackers:
- `cty` (Content Type) - Sometimes used to declare a media type for the content in the JWT payload. This is usually omitted from the header, but the underlying parsing library may support it anyway. If you have found a way to bypass signature verification, you can try injecting a `cty` header to change the content type to `text/xml` or `application/x-java-serialized-object`, which can potentially enable new vectors for [XXE](https://portswigger.net/web-security/xxe) and [deserialization](https://portswigger.net/web-security/deserialization) attacks.
- `x5c` (X.509 Certificate Chain) - Sometimes used to pass the X.509 public key certificate or certificate chain of the key used to digitally sign the JWT. This header parameter can be used to inject self-signed certificates, similar to the [`jwk` header injection](https://portswigger.net/web-security/jwt#injecting-self-signed-jwts-via-the-jwk-parameter) attacks discussed above. Due to the complexity of the X.509 format and its extensions, parsing these certificates can also introduce vulnerabilities. Details of these attacks are beyond the scope of these materials, but for more details, check out [CVE-2017-2800](https://talosintelligence.com/vulnerability_reports/TALOS-2017-0293) and [CVE-2018-2633](https://mbechler.github.io/2018/01/20/Java-CVE-2018-2633).

---
### Performing an algorithm confusion attack

An algorithm confusion attack generally involves the following high-level steps:
1. [Obtain the server's public key](https://portswigger.net/web-security/jwt/algorithm-confusion#step-1-obtain-the-server-s-public-key)
2. [Convert the public key to a suitable format](https://portswigger.net/web-security/jwt/algorithm-confusion#step-2-convert-the-public-key-to-a-suitable-format)
3. [Create a malicious JWT](https://portswigger.net/web-security/jwt/algorithm-confusion#step-3-modify-your-jwt) with a modified payload and the `alg` header set to `HS256`.
4. [Sign the token with HS256](https://portswigger.net/web-security/jwt/algorithm-confusion#step-4-sign-the-jwt-using-the-public-key), using the public key as the secret.

In this section, we'll walk through this process in more detail, demonstrating how you can perform this kind of attack using Burp Suite.
#### Step 1 - Obtain the server's public key

Servers sometimes expose their public keys as JSON Web Key (JWK) objects via a standard endpoint mapped to `/jwks.json` or `/.well-known/jwks.json`, for example. These may be stored in an array of JWKs called `keys`. This is known as a JWK Set.

```json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```

Even if the key isn't exposed publicly, you may be able to [extract it from a pair of existing JWTs](https://portswigger.net/web-security/jwt/algorithm-confusion#deriving-public-keys-from-existing-tokens).
#### Step 2 - Convert the public key to a suitable format

Although the server may expose their public key in JWK format, when verifying the signature of a token, it will use its own copy of the key from its local filesystem or database. This may be stored in a different format.

In order for the attack to work, the version of the key that you use to sign the JWT must be identical to the server's local copy. In addition to being in the same format, every single byte must match, including any non-printing characters.

For the purpose of this example, let's assume that we need the key in X.509 PEM format. You can convert a JWK to a PEM using the [JWT Editor](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd) extension in Burp as follows:

1. With the extension loaded, in Burp's main tab bar, go to the **JWT Editor Keys** tab.    
2. Click **New RSA** Key. In the dialog, paste the JWK that you obtained earlier.
3. Select the **PEM** radio button and copy the resulting PEM key.
4. Go to the **Decoder** tab and **Base64-encode** the PEM. "Encode not decode"
5. Go back to the **JWT Editor Keys** tab and click **New Symmetric Key**.
6. In the dialog, keep ID as the public key ID of the server, click **Generate** to generate a new key in JWK format.
7. Replace the generated value for the `k` parameter with a Base64-encoded PEM key that you just copied.
8. Save the key.

#### Step 3 - Modify your JWT

Once you have the public key in a suitable format, you can [modify the JWT](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#editing-jwts) however you like. Just make sure that the `alg` header is set to `HS256`.

#### Step 4 - Sign the JWT using the public key

[Sign the token](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#adding-a-jwt-signing-key) using the HS256 algorithm with the RSA public key as the secret.

### Deriving public keys from existing tokens
In cases where the public key isn't readily available, you may still be able to test for algorithm confusion by deriving the key from a pair of existing JWTs. This process is relatively simple using tools such as `jwt_forgery.py`. You can find this, along with several other useful scripts, on the [`rsa_sign2n` GitHub repository](https://github.com/silentsignal/rsa_sign2n).

We have also created a simplified version of this tool, which you can run as a single command:

```sh
docker run --rm -it portswigger/sig2n <token1> <token2>
```


You need the Docker CLI to run either version of the tool. The first time you run this command, it will automatically pull the image from Docker Hub, which may take a few minutes.

This uses the JWTs that you provide to calculate one or more potential values of `n`. Don't worry too much about what this means - all you need to know is that only one of these matches the value of `n` used by the server's key. For each potential value, our script outputs:
- A Base64-encoded PEM key in both X.509 and PKCS1 format.    
- A forged JWT signed using each of these keys.

To identify the correct key, use Burp Repeater to send a request containing each of the forged JWTs. Only one of these will be accepted by the server. You can then use the matching key to construct an algorithm confusion attack.

#### Steps
- After you get output you will find `Tampered JWT`, try to submit them without changing the user context. keep everything as it is.
- When you get a successful hit, you are able to get the key ID, and base64-encoded `x509 key` PEM of the public key. and follow the classic confusion attack. 

For more information on how this process works, and details of how to use the standard `jwt_forgery.py` tool, please refer to the documentation provided in the [repository](https://github.com/silentsignal/rsa_sign2n).