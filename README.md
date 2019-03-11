# rideOS Examples

## Obtaining access to rideOS APIs

To use rideOS APIs, you'll first need to create an account on [app.rideos.ai](https://app.rideos.ai/login#lockScreen=signUp). Once you have an account, you can view your API key on your [profile](https://app.rideos.ai/profile) page. The example `curl` commands in this repo assume that you've exported your API key into the `RIDEOS_API_KEY` environment variable: `export RIDEOS_API_KEY="YOUR_API_KEY"`.

## gRPC APIs

Some rideOS APIs are also offered as gRPC APIs:

* [Fleet Control APIs](grpc/docs/fleet_control.md)

Please see our [gRPC API documentation](grpc/docs/grpc_api.md) for more information on accessing our APIs through gRPC.
