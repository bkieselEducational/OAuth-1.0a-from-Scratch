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

The request will be constructed as shown below. Note the use of the Authorization Header and that the values assigned to the keys must be quoted.<br>

<img width="674" alt="oauth_one_1" src="https://github.com/bkieselEducational/OAuth-1.0a-from-Scratch/assets/131717897/ed89591e-135e-4c19-a984-7da62e25d5d4"><br>


### Step 4:
Here we redirect the client's browser to the login page for the vendor in question. Upon successful login, the vendor will send a GET request to our callback endpoint, which unlike in OAuth 2.0 we did not have to register with the provider of the OAuth API.

### Step 5:
Having received our OAuth Token and now the oauth_verifier as a result of the successful login, we can now send another request to the OAuth API to request an Access Token by sending the oauth_verifier in that request.

### Step 6:



## The Cryptography of OAuth 1.0a
