#sd

**In software architecture, authentication** refers to verifying the identity of our service's users. It should not be confused with **authorization**, which is the related but separate concept of determining which users have permission to take which actions.

### Storing passwords

The most typical, classic way to do authentication is with a username and password combination.

#### Plain text
The most simple approach would be to store the password text in a data store alongside the username, and then you could compare it directly to the user's password submission. DO NOT DO THIS. The critical flaw of this approach is that if your database is ever compromised, the attacker will have access to all of your users' passwords


#### Hashing

The first step we can take toward secure password storage is the use of a cryptographic [[Hashing|hash]]. **A hash function** is a function that accepts an input and maps it to a smaller value. **Cryptographic hash functions** in particular are designed to make it as mathematically difficult as possible to derive the input value from the output value.
Now we pass the password through the hash function before storing it and on authentication phase, compare the hash of the password to what is stored

#### Salting

The process involves adding a random value, known as a "salt," to a password before hashing it. The salt can be different for each user. Typically they are stored with the user info

Note: Pepper is another security technique used in conjunction with salting and hashing. Unlike a salt, which is unique and stored with each individual password, a pepper is a secret value that is shared across multiple passwords but is not stored in the database.


### Web sign in

Login handlers should always use [[HTTPS]] to ensure that credentials don't pass in plain text over the network where they could be intercepted.

Another pitfall to avoid with password submissions is logging code. It's all too easy while setting up a web project to write automatic logging code that captures the raw contents of all requests. A good rule of thumb is to avoid automatically logging POST bodies and GET parameters.

After your user has submitted their password to log in, assuming they pass verification, you need to give them a way to prove their authentication on subsequent requests.

#### Session tokens
A classic, simple way to track authentication is to generate a token the user can submit with subsequent requests to track that they are, in fact, signed in. This token should be randomly generated and long enough to be infeasible to brute force. This token will represent this particular sign-in session for this user.
Crucially, **the session token is equivalent to a password** and should be treated accordingly. Don't store the token in your database in plain text, but rather salt and hash it the same way you would a password.
Rather than plain session tokens, you may also opt to use [[JWT Token]]

#### Cookies
Whether you use session tokens or JWTs, you'll need a mechanism to store the token on the client side and send it back to the server with each request. The standard way to do this in web applications is to use browser cookies.

A cookie is simply a text string that the browser associates with a given key and sends back to the server whenever it makes a request. The server may include, in any response, one or more “Set-Cookie” headers. The header indicates the key and value to store, along with some metadata regarding expiration and security.

By storing a session token or JWT in a cookie, we can ensure that all subsequent requests will include it and allow the server to validate the current user session.

Setting the "Secure" flag on a cookie tells the browser only to include it in HTTPS requests, keeping it out of unencrypted traffic that could be intercepted. The “HttpOnly” flag tells the browser not to allow access to the cookie through JavaScript.

The "SameSite" flag allows you to specify the degree of caution the browser should take when sending requests that originate from a different site, to help prevent [[CSRF]] attacks.


### OAuth
[[OAuth and OIDC (openid)]]


