https://asgi.readthedocs.io/en/latest/

ASGI (_Asynchronous Server Gateway Interface_) is a spiritual successor to WSGI (Webs Server Gateway Interface), intended to provide a standard interface between async-capable Python web servers, frameworks, and applications.

WSGI applications are a single, synchronous callable that takes a request and returns a response; this doesn’t allow for long-lived connections, like you get with long-poll HTTP or WebSocket connections.

ASGI is structured as a single, asynchronous callable: 
- `scope`= `dict` containing details about the specific connection
- `send` = asynchronous callable that lets the application send event messages to the client
- `receive` = an asynchronous callable which lets the application receive event messages from the client.
```python
async def application(scope, receive, send):
    event = await receive()
    ...
    await send({"type": "websocket.send", ...})
```
Every _event_ that you send or receive is a Python `dict`, with a predefined format. It’s these event formats that form the basis of the standard. These events all have a type key, for ex:
 `"type": "http.request"`,  `"type": "websocket.send"`
 
ASGI is a superset of WSGI: translation layer provided by asgiref library

2 components:
- the protocol server acts as the interface between the actual network (sockets) and the ASGI application, converting low-level socket events into high-level ASGI event messages.
- An _application_, which lives inside a _protocol server_, is called once per connection, and handles event messages as they happen, emitting its own event messages back when necessary
### Example Flow (HTTP Request):

1. A user sends an HTTP request.
2. The **protocol server** accepts the socket connection, translates the HTTP request into an ASGI `"http.request"` event, and sends it to the application.
3. The **ASGI application** processes this event and sends back a `"http.response.start"` and `"http.response.body"` event to the protocol server.
4. The **protocol server** translates these responses back into a format the client understands and sends the HTTP response back to the user.

### Example Flow (WebSocket Connection):

1. A client initiates a WebSocket connection.
2. The **protocol server** terminates the WebSocket socket and sends an ASGI `"websocket.connect"` event to the application.
3. The **ASGI application** processes the connection and continues to handle any incoming events like `"websocket.receive"` (when the client sends a message).
4. The **application** can send back events like `"websocket.send"` to respond to the client.


Unlike WSGI, there are two separate parts to an ASGI connection:

- A _connection scope_, which represents a protocol connection to a user and survives until the connection closes.
    
- _Events_, which are messages sent to the application as things happen on the connection, and messages sent back by the application to be received by the server, including data to be transmitted to the client.

### Summary of the Flow:

1. **Client** → **ASGI Server**: Sends request.
2. **ASGI Server** → **ASGI Application**: Translates request into ASGI events and forwards them.
3. **ASGI Application** → **ASGI Server**: Processes events and sends back ASGI events with the response.
4. **ASGI Server** → **Client**: Translates response events back into network format and sends the response.