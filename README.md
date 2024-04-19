[<-- BACK](https://github.com/bkieselEducational/OAuth-Concepts-and-Implementation)
# OAuth 1.0A from Scratch

## OAuth 1.0A Flow Diagram
<img width="1399" alt="oauth_one_flow_2" src="https://github.com/bkieselEducational/OAuth-1.0a-from-Scratch/assets/131717897/e90a6e56-edcc-4b51-be7c-7895eb4c065a">


## OAuth 1.0a Flow Walkthrough

### Step 1:
**The user clicks the OAuth Login button that we have supplied in the client side code. For security reasons we do not want to generate the OAuth flow initializing URL on our frontend as this would expose sensitive credentials.**<br>
<br>

### Step 2:
**Our backend endpoint for initializing the flow will now grab our credentials from the environment (consumer_key, consumer_secret) and instantiate an instance of the OAuth1Session class from the authlib library.**

### Step 3:
**Inside of our endpoint, we call the method fetch_request_token() which will generate the request to be sent to the OAuth 1.0a server. The construction of this request is non-trivial and will be discussed in more detail in the section below titled "The Cryptography of OAuth 1.0a". Note that a successful response to this request will return an oauth_token that we can use to obtain an Access Token if desired / needed.**

**The request will be constructed as shown below. Note the use of the Authorization Header and that the values assigned to the keys must be quoted.**<br>

<img width="795" alt="oauth_one_temp_2" src="https://github.com/bkieselEducational/OAuth-1.0a-from-Scratch/assets/131717897/857c38e7-f1a6-4b18-af36-f7721d1810e5">



### Step 4:
**Here we redirect the client's browser to the login page for the vendor in question. Upon successful login, the vendor will send a GET request to our callback endpoint, which unlike in OAuth 2.0 we did not have to register with the provider of the OAuth API.**

### Step 5:
**Having received our OAuth Token and now the oauth_verifier as a result of the successful login, we can now send another request to the OAuth API to request an Access Token by sending the oauth_verifier in that request.**<br>

<img width="1257" alt="oauth_one_access" src="https://github.com/bkieselEducational/OAuth-1.0a-from-Scratch/assets/131717897/7342e70a-0e61-4ca7-a1aa-8cfde1acfa90"><br>


### Step 6:
**Assuming we have recieved a successful response from the vendor, we should now have an Access Token for our user. We can now finish the process of logging the user into our application.**

### Step 7:
**Finally we redirect the user back to our app, having logged them in.**


## The Cryptography of OAuth 1.0a

**Interestingly, OAuth 1.0a is actually considered to be more cryptographically involved than OAuth 2.0. It was actually an important design decision of the OAuth 2.0 authors to make this simpler for developers. The OAuth 1.0a Signing Algorithm, similar to the AWS Signature v4 algorithm (Canonical Request), requires the construction of a "Base String". The Base String is essentially a URL encoded version of what will be included in the Authorization Header of the request with a couple of additions. It is this Base String which is then hashed using a SHA1-HMAC algorithm. It is this hash that is used as the Signature for the request. On the server side, the Base String will be constructed again and the generated hash will be compared to what we sent as the signature.**

```python
# Looking at the file authlib/oauth1/rfc5849/signature.py from the authlib library we can see how the base string is being constructed

def construct_base_string(method, uri, params, host=None):
    """Generate signature base string from request, per `Section 3.4.1`_.

    For example, the HTTP request::

        POST /request?b5=%3D%253D&a3=a&c%40=&a2=r%20b HTTP/1.1
        Host: example.com
        Content-Type: application/x-www-form-urlencoded
        Authorization: OAuth realm="Example",
            oauth_consumer_key="9djdj82h48djs9d2",
            oauth_token="kkk9d7dh3k39sjv7",
            oauth_signature_method="HMAC-SHA1",
            oauth_timestamp="137131201",
            oauth_nonce="7d8f3e4a",
            oauth_signature="bYT5CMsGcbgUdFHObYMEfcx6bsw%3D"

        c2&a3=2+q

    is represented by the following signature base string (line breaks
    are for display purposes only)::

        POST&http%3A%2F%2Fexample.com%2Frequest&a2%3Dr%2520b%26a3%3D2%2520q
        %26a3%3Da%26b5%3D%253D%25253D%26c%2540%3D%26c2%3D%26oauth_consumer_
        key%3D9djdj82h48djs9d2%26oauth_nonce%3D7d8f3e4a%26oauth_signature_m
        ethod%3DHMAC-SHA1%26oauth_timestamp%3D137131201%26oauth_token%3Dkkk
        9d7dh3k39sjv7

    .. _`Section 3.4.1`: https://tools.ietf.org/html/rfc5849#section-3.4.1
    """

    # Create base string URI per Section 3.4.1.2
    base_string_uri = normalize_base_string_uri(uri, host)

    # Cleanup parameter sources per Section 3.4.1.3.1
    unescaped_params = []
    for k, v in params:
        # The "oauth_signature" parameter MUST be excluded from the signature
        if k in ('oauth_signature', 'realm'):
            continue

        # ensure oauth params are unescaped
        if k.startswith('oauth_'):
            v = unescape(v)
        unescaped_params.append((k, v))

    # Normalize parameters per Section 3.4.1.3.2
    normalized_params = normalize_parameters(unescaped_params)

    # construct base string
    return '&'.join([
        escape(method.upper()),
        escape(base_string_uri),
        escape(normalized_params),
    ])

# The Base String is returned and ultimately hashed to generate the signature using another of the class methods hmac_sha1_signature()

def hmac_sha1_signature(base_string, client_secret, token_secret):
    """Generate signature via HMAC-SHA1 method, per `Section 3.4.2`_.

    The "HMAC-SHA1" signature method uses the HMAC-SHA1 signature
    algorithm as defined in `RFC2104`_::

        digest = HMAC-SHA1 (key, text)

    .. _`RFC2104`: https://tools.ietf.org/html/rfc2104
    .. _`Section 3.4.2`: https://tools.ietf.org/html/rfc5849#section-3.4.2
    """

    # The HMAC-SHA1 function variables are used in following way:

    # text is set to the value of the signature base string from
    # `Section 3.4.1.1`_.
    #
    # .. _`Section 3.4.1.1`: https://tools.ietf.org/html/rfc5849#section-3.4.1.1
    text = base_string

    # key is set to the concatenated values of:
    # 1.  The client shared-secret, after being encoded (`Section 3.6`_).
    #
    # .. _`Section 3.6`: https://tools.ietf.org/html/rfc5849#section-3.6
    key = escape(client_secret or '')

    # 2.  An "&" character (ASCII code 38), which MUST be included
    #     even when either secret is empty.
    key += '&'

    # 3.  The token shared-secret, after being encoded (`Section 3.6`_).
    #
    # .. _`Section 3.6`: https://tools.ietf.org/html/rfc5849#section-3.6
    key += escape(token_secret or '')

    signature = hmac.new(to_bytes(key), to_bytes(text), hashlib.sha1)

    # digest  is used to set the value of the "oauth_signature" protocol
    #         parameter, after the result octet string is base64-encoded
    #         per `RFC2045, Section 6.8`.
    #
    # .. _`RFC2045, Section 6.8`: https://tools.ietf.org/html/rfc2045#section-6.8
    sig = binascii.b2a_base64(signature.digest())[:-1]
    return to_unicode(sig)
```

**Ultimately, the signature will be included with the request by being included in the Authorization Header which is defined as being of type 'OAuth'.**

## Additional Resources
1. [OAuth 1.0a Official](https://oauth.net/core/1.0a/)
2. [OAuth 1.0a Parameters](https://github.com/bkieselEducational/OAuth-1.0-Parameters)
3. [Evernote OAuth 1.0A API Reference (Good Luck!)](https://dev.evernote.com/doc/articles/authentication.php)
