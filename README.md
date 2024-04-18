[<-- BACK](https://github.com/bkieselEducational/OAuth-Concepts-and-Implementation)
# OAuth 1.0A from-Scratch

## OAuth 1.0A Flow Diagram
<img width="1387" alt="oauth_one_flow" src="https://github.com/bkieselEducational/OAuth-1.0a-from-Scratch/assets/131717897/23e84a3e-b1a9-4050-aa6e-bf31abdf3943">


## OAuth 1.0a Flow Walkthrough

### Step 1:
**The user clicks the OAuth Login button that we have supplied in the client side code. For security reasons we do not want to generate the OAuth flow initializing URL on our frontend as this would expose sensitive credentials.**<br>
<br>

### Step 2:
Our backend endpoint for initializing the flow will now grab our credentials from the environment (consumer_key, consumer_secret) and instantiate an instance of the OAuth1Session class from the authlib library.

### Step 3:
Inside of our endpoint, we call the method fetch_request_token() which will generate the request to be sent to the OAuth 1.0a server. The construction of this request is non-trivial and will be discussed in more detail in the section below titled "The Cryptography of OAuth 1.0a". Note that a successful response to this request will return an oauth_token that we can use to obtain an Access Token if desired / needed.


## The Cryptography of OAuth 1.0a
