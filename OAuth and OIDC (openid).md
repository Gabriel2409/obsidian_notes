---
id: OAuth and OIDC (openid)
aliases: []
tags: []
---
, #sd #todo
<https://youtube.com/watch?v=t18YB3xDfXI&si=JdxHyjoZrnRHuKuK>

Blog version
<https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc>

- Recommendations of PKCE even in server side apps
- Todo: authorization code injection attack

## Key Actors in OAuth 2.0

- **User = resource owner:** The individual who owns the resources. Example: A person using a social media platform who consents to share their profile data with a third-party application.

- **User-Agent:** device or application that acts on behalf of the user to interact with the client and authorization server. Example: A web browser used by the user to log into an application and authorize access.

- **Resource server**: The server that hosts the user's protected resources. Example: An API server that hosts user data, such as a social media API

- **Authorization server:** the server responsible for issuing access tokens to the client after authenticating the user and obtaining their consent. Example: The server operated by a service provider (like Google or Facebook) that handles user authentication and authorization.

- **Client:** The application that requests access to the resources on the resource server on behalf of the user. Example: A third-party application (like a calendar app) that requests access to a user's social media profile data.

## Client registration

First step is to register the client on the authorization server. Typically you sign up with your developer account on Google, Facebook, etc and register your application, which gives you credentials to use in the OAuth flow.

After that you get:

- A **client id** that uniquely identifies the app
- A **client secret** (app password)

Note that you don't always get a client secret depending on your type of app. If it is a native app, you can't hide the secret so no point in generating one. For server side app on the other hand, secret is useful.

During registration, you must also enter one or several **redirect urls**. This prevents attacker to start a flow with your app id and then redirect users to their site. Note that you should not use a wildcard url because it opens you to redirect attacks.

## OAuth for server side applications

- User clicks the button on the client app to do something
- If PKCE (Proof of Key Code Exchange) extension is enabled (preferred way), client generates a new secret on the fly for this particular flow and hashes it . Note: the generated secret is not the client secret: it is a random string called the **PKCE Code Verifier**, and the hash is called the **Code Challenge** (it is encoded in base64 url)
- Client redirects user to the OAuth server and includes the hash, the client id, the scope and the redirect url in the url query parameters. This is the first message sent in the front channel
- Now the user is on the oauth server and must log in. Then the user is shown a consent screen to make sure they want to grant access to the client.
- If the user consents, the oauth server sends the user back to the specified redirect url with the authorization code. This is another message sent on the front channel.
- Then the client can send a back channel message directly to the oauth server with the authorization code as well as the PKCE Code Verifier and the client secret to get an access token
- Now the client can make API request to the resource server with the access token.

Note: secrets are only sent through the back channel as front channel requests can be tampered with.

Note: PKCE adds an extra layer of security. Indeed, the client secret is enough to identify the client but the PKCE also ensures that a request is tied to a specific flow. At first it was only for native apps, not server side app, but PKCE solves Authorization Code injection attack

Typically, the first redirect from the client to the oauth server will look like this

```bash
https://myoauthserver.com/authorize?
  response_type=code&
  scope={SCOPE}&
  client_id={CLIENT_ID}&
  state={RANDOM_STRING}&
  redirect_uri=https://example-app.com/redirect&
  code_challenge={CODE_CHALLENGE}&
  code_challenge_method=S256
```

Note that if you use PKCE, state does not have to be a random string

Then the oauth server will redirect the user to

```bash
https://example-app.com/redirect?
  code={AUTHORIZATION_CODE}&
  state={RANDOM_STRING}
```

We have to make sure the state is the same

After that, the client can contact the server directly. Here is an example in the POST body but some servers will expect the info in the header

```bash
curl -X POST https://myoauthserver.com/oauth/token \
  -d grant_type=authorization_code \
  -d redirect_uri=https://example-app.com/redirect \
  -d client_id={CLIENT_ID} \
  -d client_secret={CLIENT_SECRET} \
  -d code_verifier={PLAIN_TEXT_PKCE_STRING} \
  -d code={AUTHORIZATION_CODE}
```

Response will look like this
`{"access_token":"{ACCESS_TOKEN}","expires_in":86400,"token_type":"Bearer", "scope": "{SCOPE}", "refresh_token":{REFRESH_TOKEN}"}`

Access token has no standard structure. You may not have a refresh token.
Usually, we want the access token to be short lived so that it is easier to revoke access.
Here is what a refresh request will look like

```bash
curl -X POST https://myoauthserver.com/oauth/token \
  -d grant_type=refresh_token \
  -d refresh_token={REFRESH_TOKEN} \
  -d client_id={CLIENT_ID} \
  -d client_secret={CLIENT_SECRET}
```

## OAuth native apps (mobile)

 2 big differences with server side app:
 
 - No client secret: PKCE was invented in the first place for mobile apps.
 - Not from a browser: in modern apps, the app uses an in app browser to redirect to the authorization server. This browser is isolated from the app and shares cookies with the rest of the system. Before we only had a webview where the app could see everything but now there are APIs built in exactly for that to launch the browser in a secure way: https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller and https://developer.chrome.com/docs/android/custom-tabs

There is also an issue with the redirect: you are not exactly sure you go back to the app. Usually apps can launch when a url matching a specific scheme is used. However, there is no registry for that so multiple apps can claim a pattern. 
For more security, we have to do **deep linking**: it is where an app claims a url pattern. To do so, a developper has to prove he owns the domain name so it is more secure but still not perfect.

Finally to store the refresh token, there is no client secret so it is very important to store it securely. On the first request, we usually do a scope=offline_access to get the refresh token at the same time as the access token. Then it is stored using a secure API. To access it, the app has to prompt the user first (so for ex redo the schema on your phone).

## Single page applications

- This time we do it directly in the browser but in the frontend. 
- Like native apps, no client secrets.
- Flow is the same as server side app. The back channel final request is done from the app directly. WHICH MEANS the authorization server must implement the correct [[CORS]] headers so that the app can make requests to it

Single page apps have imperfect options to protect the token they receive in the browser. One of the biggest risk is  [[XSS]]. If a user can run malicious code on your app, pretty much everything that is accessible to javascript is accessible to the attacker:
- LocalStorage: Lets you store data that persists across sessions. Shared across tabs
- SessionStorage: Only persists as long as that particular window is open. Not shared across tabs
- Cookies (older option): not really meant for that. Normally it is used to have the browser automatically send the cookies on every request to the backend
Note: even though storage is domain dependent, with XSS, if an attacker injects code in your app, it can read its local storage

Alternatives:
- keep token in memory: hard to steal but not great user experience
- Service workers: more secure, a lot harder to build
- WebCrypto api


A more secure flow consists in using a backend as a proxy. The idea is to use the backend to do the oauth flow (backend is confidential client so it can have a client secret) then proxy all api requests through the backend.

WHen the flow start, the browser is redirected to the authorization server and receives back the authorization code (same as usual). However, now, the js app delivers the authorization code to the backend app server which then sends it to the authorization server with a secure back channel and receives the access token. Instead of giving back the access token to the frontend, the backend gives it an http only cookie (not accessible by js). 
When the frontend want to make an authenticated request, it will send that cookie, the backend will examine it and then use the access token to make an api request to the resource server before forwarding the response to the frontend. That way frontend never saw the access token

## Example openid flow with firebase

- Firebase auth is the client and Google is the Authorization and Resource server

- **Client registration:** First we register firebase auth as a client in Google API & Services. We must specify a list of authorized redirect urls as well as the scope openid. We get a client id and a client secret.

- **User initiation:** When a user clicks "Connect with Google" on your application, Firebase Authentication initiates the OAuth 2.0 flow.

- **Authorization request:** Firebase Authentication (the client) redirects the user to the Google authorization server parameters with query parameters such as the scope, the state, the redirect uri, the code challenge, the client id.

- **User authentication:** The user authenticates in Google if needed

- **Authorization Code Response**: The authorization server redirects to the redirect url if it matches one associated with the client id with the autorization code and the state as query parameters.

- **Token Exchange**: Firebase checks that the state match to protect agains [[CSRF]] and then performs a backend channel request with the pkce string in clear, the authorization code, the client id and the client secret.

- **Token response:** The Google authorization server returns an idToken (and optionally an access and a refresh token) to firebase (the client) which parses it and extracts the identity of the user (possible because the idToken is a jwt Token).

- **Authenticated requests**: Firebase gives back a token to the browser that stores it in Indexed DB (which is a more secure storage option than local storage because it's not accessible by other origins). The token can then be used in the authorization header to perform authenticated requests

