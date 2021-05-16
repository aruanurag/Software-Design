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


