#sd

**Purpose**: Cross Origin Resource Sharing (CORS) is a security feature implemented by browsers to control how web pages can request resources from a different domain than the one that served the web page. It is used to relax the same-origin policy (SOP).

**How It Works**:

- When a web page makes a request to a different domain (cross-origin request), the browser checks if the requested resource allows the cross-origin request.
- The server must include appropriate CORS headers (e.g., `Access-Control-Allow-Origin`) in the response to indicate whether the request is allowed.

**CORS Headers**:

- **Access-Control-Allow-Origin**: Specifies which origins are permitted to access the resource.
- **Access-Control-Allow-Methods**: Specifies which HTTP methods are allowed (GET, POST, etc.).
- **Access-Control-Allow-Headers**: Specifies which headers can be used in the actual request.
- **Access-Control-Allow-Credentials**: Indicates whether the response to the request can be exposed when the credentials flag is true.

Note: Correctly setting the origin header can help with [[CSRF]] 