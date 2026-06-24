# Stirling PDF with Tailscale Docker Setup

This guide cover how to run both nginx and Stirling PDF, behind the Tailscale using Docker Compose, if you like, you can use buth web application and PDF app with this [Docker Compose](./combine-tailscale-stirlingpdf/) setup.


> Important: DON'T pasted a real Tailscale auth key into ChatGPT, GitHub, logs, or screenshots.  
In case you did, revoke it from the Tailscale admin console and generate a new one.


## Table tree
```
06-quickstart-vid
├── 01-nginx-basic
│   └── docker-compose.yaml
├── 02-stirlingpdf
│   ├── config
│   │   └── stirling.json
│   └── docker-compose.yaml
├── combine-tailscale-stirlingpdf
│   ├── config
│   │   └── stirling.json
│   └── docker-compose.yaml
└── stirling_tailscale_setup.md
```
## Table Content
- [Create a .env File](#1-create-a-env-file)
- [Generate a Tailscale Auth Key](#2-generate-a-tailscale-key)
- [Docker Compose Example](#3-docker-compose-example)
- [Stirling JSON Config](#4-stirling-json-config)
- [Enable MagicDNS and HTTPS Certificates](#5-enable-magicdns-and-https-certificates)
- [Verify Tailscale Connection](#6-verify-tailscale-connection)
- [Open Stirling PDF](#7-open-stirling-pdf)
- [Troubleshooting](#troubleshooting)
- [Tailscale GitHub](https://github.com/tailscale-dev/docker-guide-code-examples)
- [Srirling PDF GitHub](https://github.com/Stirling-Tools/Stirling-PDF)
- [Documents]()
---
## 1. Create a .env File

Create a `.env` file in the same directory as your `docker-compose.yaml`, to store your key:

```bash
TS_AUTHKEY=tskey-auth-kc4MhA5vzX11CNTRL-example
TS_AUTHKEY_CLIENT=tskey-client-kcRHZB3GxG11CNTRL-example?ephemeral=false
```

Do not put the real key directly inside `docker-compose.yaml`.

## 2. Generate a Tailscale Key

Go to admin console https://login.tailscale.com/admin/settings/

- Create a 'Auth keys' for the NGINX Tailscale docker.
 - Life time is up tp 90 days, [What is the diffrence between 'Auto Key' and 'OAuto key'?](#what-is-the-diffrence-between-auto-key-and-oauto-key)  
 
```text
Tailscale Admin Console -> Settings -> Keys -> Generate auth key
```

Recommended options:

```text
Reusable: enabled
Ephemeral: disabled
Pre-approved: enabled
Expiration: short, for example 1 to 7 days
```  

- Create a second key for the PDF tool.
```text
Tailscale Admin Console -> Settings -> Trust credentials -> Credential
```
 - Select OAuth (Description Optional)
 - Under "Devices" -> Select 'Core' with a 'Write' access, and add the tag 'container'.
 - Under "Keys" -> Select 'Auth Keys' with a 'Write' access, and add the tag 'container'.
---

### What is the diffrence between 'Auto Key' and 'OAuto key'?  

- Create both 'Auto Key' and 'OAuto key' for Tailscal and the pdf container.  
Here's a comparison of the key differences between OAuth clients and auth keys in Tailscale:

| Feature | Auth Keys | OAuth Clients |
|---|---|---|
| **Expiry** | Expire after 1–90 days (default 90) | Never expire |
| **Node ownership** | Node is owned by the user who generated the key | Node is owned by the tag assigned at creation |
| **Tags** | Optional | Required for created nodes |
| **API access** | Full API access | Scoped/limited access (e.g., `auth_keys:read`) |
| **Ephemeral nodes** | Not ephemeral by default | Nodes are ephemeral by default |

Docs ref: [Auth Keys vs OAuth; Docker Guide](https://tailscale.com/blog/docker-tailscale-guide#using-oauth)

### Key points to understand

**Auth keys** (`tskey-auth-...`):
- Authenticate a device as the user who generated the key.
- Have a maximum lifespan of 90 days, but existing nodes remain authorized until their *node key* expires (default 180 days).
- Do not require tags (though tags can be applied).

Docs Ref: [Auth Keys](https://tailscale.com/docs/features/access-control/auth-keys)

**OAuth clients** (`tskey-client-...`):
- Provide delegated, scoped access to the Tailscale API — you can limit what actions are permitted (e.g., only creating auth keys, not modifying ACLs or DNS).
- Never expire, making them better suited for long-running automation.
- Require tags; nodes authenticated via OAuth are owned by those tags, not a user.
- By default, nodes registered via OAuth are marked **ephemeral** (removed when the container stops). To change this, append `?ephemeral=false` to the `TS_AUTHKEY` value.

### Which should you use?

- Use **auth keys** for simple, short-lived setups or when you don't need tags.
- Use **OAuth clients** for production automation, CI/CD pipelines, or anywhere you need non-expiring credentials with fine-grained access control.

- Example of yaml file [docker-compose.yaml](./combine-tailscale-stirlingpdf/docker-compose.yaml):

Do not expose your OAuth key inside of the yaml file, only at the `.env` file:

```yaml
environment:
  - TS_AUTHKEY=${TS_AUTHKEY}
  - TS_AUTHKEY=${TS_OAUTHKEY_SECRET_CLIENT}?ephemeral=false
```

## 3. Docker Compose Example  

- Explained of the docker file, you can found at the officle Tailscal page:  [https://tailscale.com/docs/features/containers/docker/how-to/connect-docker-container](https://tailscale.com/docs/features/containers/docker/how-to/connect-docker-container#docker-compose-example-details)

 - How to set an OAuth key, you can found at: https://tailscale.com/docs/features/oauth-clients

## 4. Stirling JSON Config

- The JSON file [stirling.json](./02-stirlingpdf/config/stirling.json), include a settings that allow https traffic using Tailscale certificate.

Example:

```json
{
    "TCP": {
      "443": {
        "HTTPS": true
      }
    },
    "Web": {
      "${TS_CERT_DOMAIN}:443": {
        "Handlers": {
          "/": {
            "Proxy": "http://127.0.0.1:8080"
          }
        }
      }
    },
    "AllowFunnel": {
      "${TS_CERT_DOMAIN}:443": false
    }
  }
```
## 5. Enable MagicDNS and HTTPS Certificates
- Go to https://login.tailscale.com/admin/dns  
  * Scroll to the buttun, and click on the "Enable MagicDNS", and "Enable HTTPS".
  * Take your unique MagicDNS, and update into the `.env` file with a new line.
  ```text
  TS_CERT_DOMAIN=<YOUR-Magic-DNS>.ts.net
  ```

 - Example of full `.env` file:
 ```text
TS_AUTHKEY=tskey-auth-NEW_KEY_HERE
TS_AUTHKEY_CLIENT=tskey-client-NEW_KEY_HERE
TS_CERT_DOMAIN=<YOUR-Magic-DNS>.ts.net
```
- Wait for the certificate to apply for 3 minutes.

## 6. Verify Tailscale Connection

Check the Tailscale container logs:

```bash
docker logs stirling-ts
```

Check Tailscale VPN device status:

```bash
docker exec -it stirling-ts tailscale status
```

You should see the machine connected as:

```text
aviela@MacBook-M1 config % docker exec -it stirling-ts tailscale status

192.168.125.15  pdf                 pdf.magicDNS.ts.net  linux  -                          
192.168.132.88  banana              tailscale@           linux  -                          
10.16.86.61     iphone-15           tailscale@           iOS    offline, last seen 6d ago  
10.90.113.31     macbook-m1          tailscale@           macOS  idle, tx 340 rx 268   
```

## 7. Open Stirling PDF

Open this URL from your browser:

```text
https://pdf.YOUR-TAILNET.ts.net
```

Use the full Tailscale MagicDNS name, not `localhost`.

## Troubleshooting

### Restart the Containers

Run:

```bash
docker compose down
docker compose up -d
```

### Check if Stirling is running

```bash
docker logs stirlingpdf
```

### Check if Tailscale Serve loaded the config

```bash
docker logs stirling-ts
```

### Confirm containers are running

```bash
docker ps
```

@Write by https://github.com/Aviel-Amitay