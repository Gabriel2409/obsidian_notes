#sd

https://www.jsonrpc.org/
Json RPC is very close to rest but you have only one endpoint: `POST /api/rpc` for ex

Then client sends

```json
{
	// defined by client, will be sent back by server
	// used for mapping responses to requests
	id: "rpc_req_id", // str, num or null
	// json rpc does not define format for method
	// so you can name it however you want
	method: "create_taks",
	// map of parameters to pass to the function
	params: {
		...
	}

}
```

The server will respond

```json
// success
{
	id: "rpc_req_id",
	result:{
		...
	}

}
// error
{
	id: "rpc_req_id",
	error: {
		code: 4000, // int
		message: "ERROR_MSG",
		data: ..., //optional
	}
}
```

