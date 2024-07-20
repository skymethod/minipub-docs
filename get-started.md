---
title: Get started
summary: Get started with Minipub
order: 2
---

# Get Started

### Host Minipub on your own server
1. Ensure the latest [deno](https://deno.land/#installation) runtime is installed
2. Install the `minipub` cli

```sh
deno install --name minipub \
  --allow-net --allow-env \
  --allow-read --allow-write \
  https://raw.githubusercontent.com/skymethod/minipub/v0.1.7/src/cli.ts
```
3. Generate an admin user rsa keypair, save to two separate files: `admin.server.public.pem` and `admin.server.private.pem`

```sh
minipub generate-keypair
```
4. Configure public internet traffic for your firewall/load-balancer to forward `https://comments.yourapp.com` to this machine, port 2022
5. Start the server

```sh
minipub server \
  --db path/to/storage.db \
  --origin https://comments.yourapp.com \
  --admin-ip 1.2.3.4 \
  --admin-public-key-pem /path/to/admin.server.public.pem 
```

### Host Minipub inside a Cloudflare Worker
1. Ensure the latest [deno](https://deno.land/#installation) runtime is installed
2. Ensure you have the latest version of [Denoflare](https://denoflare.dev/) installed
3. Define your script in the Denoflare configuration file (e.g. `~/.denoflare.jsonc`)

```jsonc
{
  // This file supports comments!
  "$schema": "https://raw.githubusercontent.com/skymethod/denoflare/2ddef4cce293d56ab7137eacaf7d79db055b849b/common/config.schema.json",

  "scripts": {
    "comments-yourapp-prod": {
      "path": "https://raw.githubusercontent.com/skymethod/minipub/v0.1.7/src/worker.ts",
      "bindings": {
        "origin" : { "value": "https://comments.yourapp.com" },
        "backendNamespace": { "doNamespace": "comments-yourapp-prod-backend:BackendDO" },
        "backendName": { "value": "backend1" },
        "adminIp": { "value": "1.2.3.4" },
        "adminPublicKeyPem": { "value": "<text contents from your admin.server.public.pem escaped as a json string>" },
      }
    }
  }
}
```
4. In the Cloudflare Dashboard:
- Ensure you have the [Workers Paid](https://www.cloudflare.com/plans/developer-platform/#overview) plan enabled on your account (5.00 USD/month) 
- Ensure you have [Durable Objects](https://developers.cloudflare.com/workers/runtime-apis/durable-objects) enabled on your account, used as the backend data storage

5. Upload to your Cloudflare account

```sh
denoflare push comments-yourapp-prod
```
6. In the Cloudflare Dashboard:
- Set up an A record for your zone `yourapp.com` named `comments` pointing to `192.0.2.1`
- In the `yourapp.com` zone, select _Workers_ -> _Add Route_, set _Route_ to `comments.yourapp.com/*`, and _Service_ to `comments-yourapp-prod`

### Administrate your Minipub server
In either hosting option, you will now have a comments service running at `https://comments.yourapp.com`

You can define users, create comments, federate activities using the Minipub [Admin RPC API](/admin-rpc), or the `minipub` cli which wraps it.

You can use the cli help (e.g. `minipub --help` or `minipub create-note --help`), or take a look
at the [Admin RPC API](/admin-rpc) docs for details of how to orchestrate your Minipub service from code.

### Ideas and Feedback
Start a [discussion](https://github.com/skymethod/minipub/discussions) over in the project's Github repo

### Funding this project
This is an independent open-source project by [John Spurlock](https://github.com/johnspurlock-skymethod) with no financial backing, just an interest in helping open podcasting comments get off the ground.

If you support the goals of this project and want to help fund its ongoing development, you can send a donation our way using the button below, thanks!

<Button type="primary" href="https://buy.stripe.com/6oE2aIduL5rzfEQfYZ">Donate →</Button>
