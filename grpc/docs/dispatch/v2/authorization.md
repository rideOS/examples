Authorization in Dispatch API v2
================================

The Dispatch API v2 can be called from backend-to-backend, mobile app, and web frontend contexts.
Depending on the context, the API applies various authorization checks to ensure that callers cannot access resources (tasks and vehicles) that they do not have permission for.
If a request is issued that attempts to access unauthorized resources, an HTTP 403 or gRPC `PERMISSION_DENIED` error will be issued.

The authorization context is based on the token provided in the call headers/metadata.

Full API access
---------------

Your rideOS API key provides full access to all API endpoints, including the Dispatch API.
Specifically, if `X-Api-Key` header (HTTP) or `Authorization: rideOSToken ...` metadata (gRPC) are used with a rideOS API key, then the caller may call any method in the Dispatch API without restriction.
In this context, all actions are allowed: the caller may read or modify any vehicle and any task.
Furthermore, the caller may read and modify fleet-level resources, like FleetMetadata objects.

This context is used for backend-to-backend calls (i.e. a customer backend directly calling the rideOS backend), and is also used by our web frontend to call our API.

Own-resource access
-------------------

If you use user login (e.g. in the mobile app), you will receive an OAuth2 access token.
When used to access the Dispatch API (e.g. using the `Bearer` authentication method in gRPC), this token provides "own-resource" access.
In this context, the caller may only access their own resources, identified by the caller's *subject*.
The "subject" is available as one of the properties stored in the OAuth2 token, and cryptographically verified by the OAuth2 provider.
In this context, the caller may access resources by putting their subject in the appropriate ID fields in requests.

For tasks, put the subject into the `passenger_id` field when requesting tasks.
After the request, this is stored by the backend in a field called `passenger_id` or `requestor_id` depending on the context.
Callers may only query tasks whose `passenger_id` or `requestor_id` matches their subject.

For vehicles, put the subject into the `vehicle_id` field.
Callers may only make calls with a `vehicle_id` that matches their subject.

This context is primarily used by the mobile apps to call the Dispatch API directly.

Roadmap
-------

We are working on a new iteration of our authentication service that will support more fine-grained control of permissions than the two methods presented above.

Custom authentication
---------------------

The Fleet Control mobile apps use our authentication service in order to enable the above "own-resource" authorization check.
However, if you are developing customized mobile apps, you may wish to plug in your own authentication method or provider.
To handle authorization, there are two potential options to explore:

1. Run your own backend, and call the Dispatch API in a backend-to-backend way.
   Your custom backend calls the Dispatch API in the "full access" context, allowing any action to be taken.

   Your mobile app exclusively calls your backend, which then forwards any necessary calls to the Dispatch API.
   Therefore, care must be taken in your backend to authorize the mobile app calls correctly, as the "full access" context will not perform any authorization checks.

2. If your authentication provider can act as a SAML IDP (perhaps you have your own Auth0 tenant with your own users database), we can link this with our provider.
   Your custom mobile apps call the Dispatch API directly, without going through your own backend.
   This allows you to maintain your own users database, while continuing to use the "own-resource" access context, and allowing the Dispatch API to perform authorization checks.

   As this is a relatively custom procedure, please contact us directly (support@rideos.ai) to set up this method of authentication.
