---
title: Admin RPC API
summary: Administrate Minipub using its admin RPC API
order: 3
---

# Admin RPC API

Integrating Minipub into your app means being able to make programmatic calls to your Minipub service
from your app's backend server in response to actions taken by your app's users in your app.

Use the secure admin rpc API to orchestrate your Minipub service from your backend server.

### Overview

All admin api calls are made by making http `POST` calls to the `/rpc` endpoint of your Minipub service.

```
POST https://comments.yourapp.com/rpc
content-type: application/json; charset=utf-8
host: comments.yourapp.com
date: Sun, 13 Feb 2022 00:51:01 GMT
signature: keyId="admin",algorithm="rsa-sha256",headers="(request-target) host date digest",signature="<base64>"
digest: SHA-256=<base64>

{
  "kind": "delete-note",
  "objectUuid": "a04763cbc94d4d06b38af8e81de63e5e"
}
```
The request body must be UTF-8 encoded JSON, and correspond to a known `RpcRequest` type defined in [rpc_model.ts](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts).
Each rpc request type will have an identifying `kind` string property, and return a JSON response with the same `kind`.

```
200 OK
content-type: application/json; charset=utf-8
date: Sun, 13 Feb 2022 00:51:01 GMT

{
  "kind": "delete-note",
  "objectUuid": "a04763cbc94d4d06b38af8e81de63e5e"
  "activityUuid": "7c2d2012c29d4fecb8b0b6cb9ba0672f"
}
```

### Authentication

Admin rpc calls should use [HTTP Signatures](https://technospace.medium.com/ensuring-message-integrity-with-http-signatures-86f121ac9823) for authentication.
 - Send the `host`, `date`, `digest`, and `signature` headers
 - The `digest` is computed over the request body, ensuring the message cannot be tampered with
 - The `date` helps guard against [replay attacks](https://en.wikipedia.org/wiki/Replay_attack)
 - The `keyId` signature parameter should be `admin`, as shown above
 - Use the admin private key pem file generated when you first installed the service to compute the signature
 - This admin private key pem file should never leave your backend server, other than perhaps to back it up
 - An example of computing these headers in TypeScript can be found in [minipub cli source](https://github.com/skymethod/minipub/blob/v0.1.6/src/cli.ts#L36). All calls made via the `minipub` cli use HTTP Signatures for authentication.

### Authentication (Bearer token)

Alternatively, if you cannot use HTTP Signatures or would rather use a simpler method, you can generate an admin bearer token to use instead.

Generate (or regenerate) the admin bearer token using the `minipub` cli:
```sh
minipub generate-admin-token \
  --origin https://comments.yourapp.com \
  --pem /path/to/admin.private.pem
```
```
{
  "kind": "generate-admin-token",
  "token": "t&l8p2=p*]UN-Q-IYTKzdLYZ[-y(@b5ej<kf^nn("
}
```

Now you have the option of using the bearer token in the `authorization` header:
```
POST https://comments.yourapp.com/rpc
content-type: application/json; charset=utf-8
host: comments.yourapp.com
authorization: Bearer t&l8p2=p*]UN-Q-IYTKzdLYZ[-y(@b5ej<kf^nn(

{
  "kind": "delete-note",
  "objectUuid": "a04763cbc94d4d06b38af8e81de63e5e"
}
```

To revoke the admin bearer token using the `minipub` cli:
```sh
minipub revoke-admin-token \
  --origin https://comments.yourapp.com \
  --pem /path/to/admin.private.pem
```

### Admin IP filter

In addition to the authorization mentioned above, admin rpc calls to your Minipub service will only be accepted from the IP address specified when launching the service.

i.e. the `--admin-ip` option to `minipub server` or the `adminIp` variable binding in your Cloudflare Worker.

### Available RPC kinds

| Kind          | Description   |
| ------------- | ------------- |
| [create-user](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L106) | Creates a new user (Actor) |
| [update-user](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L144) | Updates a existing user (Actor) |
| [create-note](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L197) | Creates a Note object and associated Activity |
| [update-note](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L225) | Updates the content for an existing note object, and generates an Update activity if modified |
| [delete-note](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L248) | Deletes an existing note object, and generates a Delete activity |
| [federate-activity](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L268) | Federates an existing activity to its remote recipients, if any |
| [delete-from-storage](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L291) | Deletes a value from backend storage |
| [like-object](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L312) | Creates a local Like activity for a given remote object id |
| [undo-like](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L333) | Creates a Undo activity for a given local Like activity |
| [generate-admin-token](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L352) | Generates or regenerates a bearer token the admin can use to make rpc calls without http signing |
| [revoke-admin-token](https://github.com/skymethod/minipub/blob/v0.1.6/src/rpc_model.ts#L369) | Revokes the existing admin bearer token, allowing access only via http-signed calls |
