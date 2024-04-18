[<-- BACK](https://github.com/bkieselEducational/OAuth-Concepts-and-Implementation)
# OAuth 1.0A from-Scratch

## OAuth 1.0A Flow Diagram
<img width="1399" alt="oauth_one_flow_2" src="https://github.com/bkieselEducational/OAuth-1.0a-from-Scratch/assets/131717897/e90a6e56-edcc-4b51-be7c-7895eb4c065a">


## OAuth 1.0a Flow Walkthrough

### Step 1:
**The user clicks the OAuth Login button that we have supplied in the client side code. For security reasons we do not want to generate the OAuth flow initializing URL on our frontend as this would expose sensitive credentials.**<br>
<br>

### Step 2:
**Our backend endpoint for initializing the flow will now grab our credentials from the environment (consumer_key, consumer_secret) and instantiate an instance of the OAuth1Session class from the authlib library.**

### Step 3:
**Inside of our endpoint, we call the method fetch_request_token() which will generate the request to be sent to the OAuth 1.0a server. The construction of this request is non-trivial and will be discussed in more detail in the section below titled "The Cryptography of OAuth 1.0a". Note that a successful response to this request will return an oauth_token that we can use to obtain an Access Token if desired / needed.

The request will be constructed as shown below. Note the use of the Authorization Header and that the values assigned to the keys must be quoted.**<br>

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

Interestingly, OAuth 1.0a is actually considered to be more cryptographically involved than OAuth 2.0. It was actually an important design decision of the OAuth 2.0 authors to make this simpler for developers. The OAuth 1.0a Signing Algorithm, similar to the AWS Signature v4 algorithm (Canonical Request), requires the construction of a "Base String". The Base String is essentially a URL encoded version of what will be included in the Authorization Header of the request with a couple of additions. It is this Base String which is then hashed using a SHA1-HMAC algorithm. It is this hash that is used as the Signature for the request. On the server side, the Base String will be constructed again and the generated hash will be compared to what we sent as the signature.
