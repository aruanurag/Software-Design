# OAuth 2.0 and OpenId

## Why do we need OAuth and OpenId
Allow applications to access data from third party apps without the users sharing their password. OAuth issues access tokens to apps, OpenID Connect issues ID tokens to apps.
And the ID Token is a statement about the user.
OAuth was created as a delegated authorization protocol. It has been extended to be used as a single-sign-on protocol through things like OpenID Connect

### Issues it tries to solve
1. It's the single sign on case that really gets to the core of why OAuth was created, especially once you consider that you may want multiple apps by different companies to be 
able to share the same user database.
2. On the user Side it is difficult to trust a third party app requesting your google (or any identity provider) passwords for authentication.
3. For API developers using inbuild username and password in exchange of authentication mechanism . It becomes very difficult if we want to use any form of MFA . As any MFA implementation needs to be worked upon even on the API side. Which is a waste of developement effort.

### How OAuth solves the problem
1. What OAuth does at a high level is requires that every application sends the user out to the OAuth server to log in there, and then redirects them back to the app so the app can get tokens. And the key thing here is this redirect step. It means the user actually leaves the application and they go type in their password at the OAuth server instead of ever giving their password to the application. So as soon as we avoid the application ever seeing the user's password, it solves all of these worries and uncertainties we have before. It provides security against untrusted third party apps and also makes first party apps much more flexible. 
2. This way, if you wanted to add MFA, you don't need to make any changes to the apps at all, since you just turn it on at your OAuth server and it would immediately be enabled across all of your applications.

## OAuth 2.0
OAuth 2.0 defines two client types, 
1. **Confidential** - Confidential clients have the ability to be deployed with a client secret where that secret won't be visible to anybody using the app. This is a pretty normal thing for apps running on a server, say apps written in a server-side language like Java or .Net or PHP.
2. **Public clients** - If you're writing a mobile app or a single page app, then you don't have the ability to include secrets in the app, because users of the app would be able to see the secrets. It's most obvious in the case of single-page apps, since users of the app can always click on View Source in the browser and then start poking around the source code, looking at all the code and finding the secrets, there's just no way to ship a secret in that source code and have it remain secret.

### How does client type matter to Oauth -
So the reason this distinction matters is that the authorization server might have different policies that make it act differently depending on the type of client making
the request. For example, a confidential client that is also a first party app might have the consent screen skipped when it starts a flow because the authorization server can be sure that only the real application can actually complete that flow and end up with an access token. However, for a first party public client, an attacker could mimic that application by copying its client ID and starting a flow. And if they can control the redirect URL they could end up with access tokens of the authorization server thought were being delivered to the real application. So in that case, you may want to still include the content screen to get the user involved in that flow. Some of the other things the authorization server might do differently, depending on the client type, is things like whether to include refresh tokens or changing the token lifetimes to mitigate risk. All of these things are reasons to use client authentication when possible.

## OAUth for Server Side Apps

- First Step is to register an application with the OAuth server and have a client ID and secret.
- Flow :
  1. The flow starts out with the user clicking the login button. That's like the user saying, I would like to use this application.
  2. Before the app redirects the user, it makes up a new secret for this particular flow. This is not the client secret. This is a random string the app      generates and it's different every time it starts to flow. This is called the PKCE Code Verifier.
  3. So the app holds onto that code verifier in the app's server and then calculates a hash of it called the Code Challenge. And a hash, of course, is a one way operation. So somebody knows the hashed value. They can't reverse engineer it and figure out the secret. So it takes that hash and includes it in that URL, which it builds to send the browser over to the server.
  4. It'll redirect the user to the server with a bunch of stuff in the query string, including that hash, the client ID, redirect URL, and scope. So the user ends up at the server delivering the message the app sent.
  5. So the user is now at the OAuth server and the server asks them to log in, they log in,they do any sort of multifactor auth necessary, and then that server asks them to confirm if they really are trying to log into the app. If they say, yes, the server needs to send the user back to the app and also deliver this temporary one time use authorization code.
  6. So it takes the app's redirect URI, adds the authorization code in the query string and then sends the user's browser there to deliver that back to the app.
  7. The backedn App can now ue the authorization code with client id and secret to get the access token from the auth server.

## OAUth for Native/Mobile/SPA Apps

Since Native/Mobile Apps have public clients. When you're building a native app, remember that you don't have a way to deploy any credentials if you're shipping it in an app store. So when we register a app with a auth server and a option is given to choose whether it is a public or confidential client, for Public client a Client Secret is not generated.
FLow starts like this :

**Message 1 (Front Channel)**
1. The user clicking the login button and that's the user saying, I would like to use this application.
2. Before the app redirects the user (to the auth server), it makes up a new secret for this particular flow. This is not the client secret. This is a random string the app generates and it's different every time it starts to flow. This is called the PKCE Code Verifier.
3. It holds on to that code verifier in the app itself. Then it calculates a hash called the code challenge. And a hash, of course, is a one way operation. So somebody knows the hashed value. They can't reverse engineer and figure out the original secret.
4. So it takes that hash and it includes it in the URL that it builds, which tells the browser to go to the OAuth server. Query Parameters will include
    1. With Client-ID, 
    2. Code-Challenge, 
    3. Response_typ=code (telling the server it is the authorization code flow)
    4. State Value - random string that should be matched when a response is retunred from the Auth Server
5. Now, instead of a redirect, the app launches a new in-app browser to get the user to the server with all that stuff in the query string, including that hash, it also includes the client ID, redirect URL and scope.
6. So the user ends up at the OAuth server, which is delivering this message the app sent.

So what's actually really happening here is the app is trying to request some stuff from the server, but instead of requesting it directly, it hands its request to the user to deliver to the OAuth server. And because this is a front channel request, that's the reason it sends only a hash of the secret rather than sending the secret itself. Now, in the case of a mobile app, it's a little bit easier to understand the front channel risks here as well. 
The backchannel is when the app makes a request from code within the app directly to the server. The front channel is when the app makes a request through the system browser to the server, which means that request leaves the little walled garden of the app and is more exposed. 

**Message 2(Front Channel)**
1. Ok, so the user is now at the server and the server asks them to log in.
2. They log in, they do any sort of multifactor auth, and one of the neat things here about using the in-app browser (that's in Android, Chrome Custom Tabs or SFSafariViewController on iOS) is that because this browser is actually isolated from the app, it can share cookies with the rest of the system, meaning if the user's already logged into the server in their native browser when this in-app browser pops up, they might already be logged in as well, because it can share cookies with the rest of the system.
3. Ok, so then the OAuth server asks the user to confirm they're trying to log in to this application. And if they say yes, the server needs to send the user back to the app and also deliver this temporary authorization code. So it takes the redirect URI, adds the authorization code in the query string and also includes the state Value and then sends the user's browser there to deliver that back to the app. 
4. And again, because this is a front channel request, the server can't really be sure that the code was received by the application. So this authorization code is valid for only one use and it has to be redeemed within a short period of time, typically under a minute.

**Message 3(Back Channel)**
1. So now that the application has the authorization code, it can go make a back channel request to exchange that for an access token.
2. This request is made from the app's code to the OAuth server (Back Channel).
3. Now, the app still doesn't have a client secret, but it does have the plaintext secret it generated at the beginning. The server looks in that request and says, ok, I see that I just issued this code, it hasn't been used yet, and it was intended for this client. And when this request started, I saw this hash value, the code challenge. So the OAuth server calculates the hash of the code verifier in the request, compares the hashes and if they match then the server knows the thing redeeming this code is the same thing that started the flow.
4. And then the OAuth server generates the access token and returns it in the response, and then the flow is done and the app can go make API requests with that access token.
5. So this step of doing that hash is the PKCE extension.
And PKCE was originally developed for mobile apps to protect the authorization code flow because there is no client secret. And it turns out that PKCE also protects against some other specific attacks, even if you do have a client secret. So the latest recommendations from the OAuth group are for all applications to use PKCE even if you already have a client secret.


## Refresh Tokens
Refresh tokens are special tokens whose only job is to get new access tokens. The thing that makes them special is the app is able to use them without getting the user involved. Essentially, these are used to keep the user logged in while also letting access tokens last for a shorter time.
Some OAuth servers always return Refresh_tokens. There is a convention to use **scope=offline_access** to get a refresh token.
If the app receives a refresh token, it will be at the same time it gets the access token, returned in the response alongside the access token. If the app sees the access token is about to expire, it can go use the refresh token to get a new one first. Using the refresh token is actually a very simple POST request to the OAuth server's token endpoint. This is the same endpoint that the app requested the first access token from using the authorization code. The post includes **grant_type=refresh_token**, the refresh token itself and the app's client ID. There's no client secret here because native apps don't get client secrets.

## OpenId Connect

Extension of OAuth. While OAuth is always about applications accessing APIs, OpenID Connect is about applications learning information about users.
The main thing that OpenID Connect adds into the picture is an ID Token. The ID Token is the way that the authorization server communicates information about the user who just logged in to the application. So what exactly is an ID Token? Well, it turns out that ID Tokens are always JSON Web tokens. JSON Web Token is a format of token that is used for ID Tokens and sometimes OAuth servers.
Access tokens dont have a defined format they can be JWT or anything else but Id Tokens are always JWT.
JSON Web Token is broken up into three parts. There's the header, the payload and the signature. The header talks about the token. The payload actually contains the data that you care about. And the signature is how you can validate that token.

**How to get an Id Token**

Applications very often want to get both an access token and an ID token. It'll use the access token to make API requests and an ID token to identify the user. Now, if the application is already getting an access token using the authorization code flow, then by far the easiest way to get an ID token is to add the scope "openid" to the request, and you'll get back in ID token and an access token.
So what this looks like is you take the normal authorization request using the authorization code flow, that's going to include response_type=code and your normal scopes that you would need in order to get the appropriate access token. And then you just add a new scope into the list and the scope is called "openid". So what that does is it tells the OAuth server that you also want an ID token. So the user logs in like normal and then the application gets this authorization code, this temporary authorization code. When it goes and exchanges that code for an access token, it will also get back an ID token in the same response. So you'll notice that the response contains the normal OAuth response properties like access_token and expires_in. And there's a new property in the response called id_token. So you can see it's very straightforward to just get an ID token along with the access token in this way. One of the really nice things about doing it this way is that because you obtained the ID token over the back channel by directly exchanging that authorization code for the ID token, you actually can forget that it's a JSON Web Token and you can skip all the verifications, because you already know where it came from. So you don't need to validate the signature and you already know that it's valid so you don't need to check the expiration dates. You can basically just extract the parts in the middle that you care about and just use the data directly./

## API Validation

So what we're talking about here is how an API can check whether an access token is valid. So the basic idea is if your API gets a request with an access token in it, it needs to somehow know if that is valid before returning data.

**Simple/Slow way** requiring network connection to do a round trip.
The simplest way to do this is to go back and ask the authorization server if it's valid . So the API can go make a POST request to the server, passing along the token, and the server can reply back whether the token is valid or not. This happens by making a post request to the OAuth servers introspection endpoint sending along the token.
But the key takeaway here is that by using remote token introspection, you push more of the work of validating the token onto the server to tell you whether it's active. And your resource servers don't need to worry about token formats. The only downside is that this requires network traffic. So if you're doing remote introspection for every API request you get, you're going to be adding latency to every API request of whatever that time it takes for your API call to go out to the server and back. Now, if your access tokens are just a random string, then this is really the only option you have in order to validate your access tokens.

**Non-Network/faster way** for JWT tokens. No Network traffic required also called local validation.
Before we go through all these teps one thing to point out is the easiest way to do this is by using a library (>net JsontokaneValidation).

Remember that a JSON Web Token has three parts.
The header, the payload and the signature. The header and the payload are Base64 encoded JSON. So we'll first base64 decode those and see what's inside. The header talks about the token, describing which signature algorithm was used and may include other things like the identifier of the key that was used to sign the token. One thing to note here is that the JSON Web Token access token spec explicitly states that the "none" algorithm must not be used. This is good because "none" was probably the worst idea to ever include in the JSON Web Tokens spec to begin with. And it's been the source of many, many vulnerabilities over the years. But the other thing to keep in mind is that you are reading data out of this header before verifying the signature. So you have to treat this all as untrusted data at this point. The main impact of this is that you should only accept signing algorithms that you know to expect from the server.

The JSON Web Token access tokens spec recommends that OAuth servers publish their public keys and issuer identifier at a URL that can be found from the servers metadata URL. That metadata URL can be usually found either in the docs for the OAuth server that you're using or you can build it by tacking on this .well-known path onto the server's issuer URL. If you fetch that metadata, you will see a bunch of JSON data that describes the OAuth server. It's got things like the issuer identifier, the URL for the authorization endpoint, the token endpoint, other stuff we don't care about right now. But it also has this jwks_uri property, which is the URL where you can find the keys. If you fetch that URL, you'll get to the actual public key data.
There might be one or two keys here depending on the server. One of the properties of the key is "kid", and that's the identifier for the keys. You can match up that key with the one in the header of the access token and that way you know which key to use to verify the signature.
The rest of this is the actual public key itself.

So the next step is to actually validate the signature. Now for this, you'll definitely want to find a JSON Web Token library because you should never write cryptography code by hand. So you take your JSON Web Token access token and the key, and hand it to this library and it will tell you if the signature is valid.
