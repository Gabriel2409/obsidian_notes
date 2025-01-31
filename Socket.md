#sd 
https://www.youtube.com/watch?v=XXfdzwEsxFk

A socket is an abstraction provided by the operating system to manage network communication. 
**Stream Sockets (TCP)** provide a reliable, connection-oriented communication channel. TCP sockets ensure that data is delivered in order and without duplication.

**Datagram Sockets (UDP)** provide a connectionless communication channel. UDP sockets are faster but do not guarantee delivery, order, or error checking.

When working with the socket API, the OS returns a [[File descriptor]] that represents the socket. You use this file descriptor to perform operations on the socket, such as reading, writing, or closing it.

The OS kernel manages the complex details of network communication, including routing, handling protocol specifics (like TCP or UDP), buffering data, and dealing with errors or retransmissions.


## Socket API

![[Socket_api.excalidraw]]

The client makes the connection. The server receives it. Once the connection is established, both client and server use `send` and `recv` to send and receive messages.
So basically, client is in charge of `connect` and server of `bind`, `listen` and `accept`


In the diagram, client initiate closing but it can also start from server. 

### Individual commands
#### getaddrinfo

Leverages DNS. Input is a dns hostname and output a list of potential ips to connect to/listen to. 
#### socket
Creates a file descriptor. 
But that's it! Socket is not connected to anything yet
It does not use by itself the output of `getaddrinfo` 
#### send and recv
Given a file descriptor and a collection of bytes, tell the OS to attempt to deliver / receive up to this many bytes out of the buffer 
- Works very similar to `read` and `write`
- `send` and `recv` operate on user level memory & kernel level buffers but the do NOT do the sending themselves! All it means is that the OS received this message from you and will do its best to deliver it to the other side.

By default, send and recv are blocking. 

When you call `send()`, the function blocks until the data has been copied into the OS's socket buffer. This means that once `send()` returns, the data has been handed off to the operating system, but it does **not** guarantee that the data has been delivered to or received by the remote side. It only guarantees that the OS has successfully received the data for transmission. For TCP sockets, the OS will manage retransmissions, acknowledgments, and ordering as part of the TCP protocol.

When you call `recv()`, the function blocks until there is data available to be read from the socket. It guarantees that your application has received some data that was sent by the remote side. However, it does not provide any information about how much of the data the remote side has sent has been received

#### close
Takes a file descripter and tell the kernel to terminate the connection:
- Kernel continues sending buffered bytes
- At end of buffered bytes, sends special EOF message (interpreted as len=0 result for recv)
#### connect

Given a file descriptor and an IP to connect to, create a connection in an abstracted way. OS sets a reliable connection on top of best effort delivery provided by network layer

An application programmer does not need to know how it works (TCP under the hood usually).

#### bind
Given a file descriptor, tell the kernel to associate it with a given IP and port. The port is reserved in the sense that no one else can use it. But that's all !

#### listen
Given a file descriptor that has been bound to a port, tell the os that you wish to start receiving connections. Now 

#### accept
Given a file descriptor activated via listen, start a conversation with one client. 
By default, blocks until a client shows up. 
It returns ANOTHER descriptor that will then be used for the conversation. 



### example code (python)

```python
#server.py
import socket

# Create a socket object
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Set the SO_REUSEADDR option to reuse the port
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# Bind the socket to an address and port
server_socket.bind(("localhost", 65430))

# Listen for incoming connections
server_socket.listen()

print("Server is listening on port 65430...")

# Accept a connection
conn, addr = server_socket.accept()

with conn:
    print(f"Connected by {addr}")

    # Receive data from the client
    data = conn.recv(1024)
    if data:
        print("Received from client:", data.decode())

        # Send a response back to the client
        conn.sendall(b"Hello, Client!")

# Close the server socket
server_socket.close()
```
Note: you can use the convenience function `create_server`:
```python
with create_server(('', 8000)) as server:
	while True:
		conn, addr = server.accept()
		...
```

and the client side
```python
import socket

# Create a socket object
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to the server
client_socket.connect(("localhost", 65430))

# Send a message to the server
client_socket.sendall(b"Hello, Server!")

# Receive a response from the server
data = client_socket.recv(1024)
print("Received from server:", data.decode())

# Close the client socket
client_socket.close()
```

