#sd #todo
https://youtube.com/watch?v=t18YB3xDfXI&si=JdxHyjoZrnRHuKuK

Blog version
https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc

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
- If PKCE (Proof of Key Code Exchange) extension is enabled (preferred way), client generates a new secret on the fly for this particular flow and hashes it . Note: the generated secret is not the client secret: it is a random string called the **PKCE Code Verifier**, and the hash is called the **Code Challenge**
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

Then the oauth server will redirect the user to `
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
  -d refrsh_token={REFRESH_TOKEN} \
  -d client_id={CLIENT_ID} \
  -d client_secret={CLIENT_SECRET} 
```
