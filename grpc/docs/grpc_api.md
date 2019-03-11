rideOS gRPC API Getting Started
===============================

The most efficient way to call the rideOS API is through gRPC. gRPC is an RPC protocol used by major tech companies that
has first-class support for advanced RPC interactions, for example asynchronous callbacks and bidirectional streaming.
Some features of the rideOS API may only be available through gRPC. For more general information about gRPC, please
refer to the official website:

https://grpc.io/

Endpoint and Authentication
---------------------------

All gRPC rideOS APIs are available at the endpoint `gapi.rideos.ai:443` over SSL. In order to call our gRPC API, you
must provide authentication information in the metadata of each request. In particular, use the following metadata:

| Metadata key    | Value                        |
| --------------- | ---------------------------- |
| `Authorization` | `rideOSToken (your_api_key)` |

Note that the value should be `rideOSToken`, a space, then your API key. rideOS API keys are typically around 97
characters long and contain a colon (`:`). If your API key does not look like this, please contact us directly for
instructions.
