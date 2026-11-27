---
title: Publish the platforms over HTTPS
description: Publish OpenCTI, OpenAEV and XTM One over HTTPS behind a reverse proxy, with HTTP/2, WebSockets, streamed responses, forwarded headers, secure cookies and the right public URLs.
doc_type: how-to
products:
  - suite
personas: [platform-engineer]
owner: docs-platform
last_reviewed: 2026-10-31
migrated_from:
  - opencti:/deployment/installation/
---

This page puts a reverse proxy in front of the XTM stack, so that users reach each platform on its own host name over HTTPS. The proxy terminates TLS and forwards to the platforms, which keep calling each other inside Docker. The example uses Nginx; any proxy that serves HTTP/2, forwards WebSockets and does not buffer streamed responses works the same way.

::diagram{id=deploy-install-reverse-proxy}

## Before you start

- The stack installed as [Install with Docker Compose](/deploy/install/docker-compose/) describes, with the public URLs set (step 6) and the platforms published on local ports (step 7): OpenCTI on `127.0.0.1:8080`, OpenAEV on `127.0.0.1:8081`, XTM One on `127.0.0.1:8090`.
- A DNS record for each host name, such as `opencti.example.com`, `openaev.example.com` and `xtm-one.example.com`, pointing to the proxy.
- A TLS certificate and its key for each host name, or one certificate that covers them all.
- Nginx on the Docker host.

## What the platforms expect

| Need | Why |
| --- | --- |
| The public URL in `.env` | Each platform builds its links and signs its requests to the others with `*_EXTERNAL_SCHEME`, `*_HOST` and `*_PORT`. With TLS, set the scheme to `https` and the port to `443`. |
| WebSockets forwarded | OpenCTI serves its GraphQL subscriptions over a WebSocket on `/graphql`, and OpenAEV uses WebSockets too. In XTM One, the XTM One Desktop app, the runners and the relays connect over a WebSocket, and fall back to slower HTTP long-polling without it; the chat does not use one. The proxy must pass the `Upgrade` and `Connection` headers. |
| HTTP/2 to browsers, for XTM One | The XTM One chat streams its answers as server-sent events. Every open tab of XTM One holds its activity stream, and a chat tab also the stream of its conversation and, while an answer is written, the stream of that message. A browser opens at most 6 HTTP/1.1 connections per host: over HTTP/1.1, a few open tabs use them all and XTM One stops loading. HTTP/2 carries every stream on one connection. The proxy still reaches XTM One over HTTP/1.1. |
| No buffering | Responses that stream, such as subscriptions and the answers of AI agents, must reach the browser as they come. A proxy that buffers the XTM One chat shows each answer only once it is complete. |
| A read timeout above 15 seconds | An XTM One stream with nothing to send writes a keep-alive every 15 seconds: a shorter read timeout cuts it. Nginx waits 60 seconds by default; [Finish the setup of XTM One](/deploy/configure/xtm-one/#finish-the-setup-after-the-install) raises it for the long answers of AI agents. |
| Forwarded headers | The platforms read the scheme and the client address from `X-Forwarded-Proto` and `X-Forwarded-For`. OpenCTI takes the client address only from a proxy on a loopback, link-local or private address; OpenAEV reads them (`server.forward-headers-strategy=framework`). |
| Large uploads | Nginx refuses request bodies over 1 MB by default. OpenAEV accepts files up to 5 GB. |

## Configure the proxy

:::steps
1. Write one server block per platform in `/etc/nginx/conf.d/xtm.conf`. Replace the host names and the certificate paths with yours.

   ```nginx
   server {
       listen 443 ssl;
       server_name opencti.example.com;
       ssl_certificate     /etc/nginx/tls/opencti.example.com.crt;
       ssl_certificate_key /etc/nginx/tls/opencti.example.com.key;
       client_max_body_size 5g;

       location / {
           proxy_pass                  http://127.0.0.1:8080;
           proxy_http_version          1.1;
           proxy_set_header Upgrade    $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host       $host;
           proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_cache                 off;
           proxy_buffering             off;
           chunked_transfer_encoding   off;
       }
   }

   server {
       listen 443 ssl;
       server_name openaev.example.com;
       ssl_certificate     /etc/nginx/tls/openaev.example.com.crt;
       ssl_certificate_key /etc/nginx/tls/openaev.example.com.key;
       client_max_body_size 5g;

       location / {
           proxy_pass                  http://127.0.0.1:8081;
           proxy_http_version          1.1;
           proxy_set_header Upgrade    $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host       $host;
           proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_cache                 off;
           proxy_buffering             off;
           chunked_transfer_encoding   off;
       }
   }

   server {
       listen 443 ssl;
       http2 on;
       server_name xtm-one.example.com;
       ssl_certificate     /etc/nginx/tls/xtm-one.example.com.crt;
       ssl_certificate_key /etc/nginx/tls/xtm-one.example.com.key;
       client_max_body_size 5g;

       location / {
           proxy_pass                  http://127.0.0.1:8090;
           proxy_http_version          1.1;
           proxy_set_header Upgrade    $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host       $host;
           proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_cache                 off;
           proxy_buffering             off;
           chunked_transfer_encoding   off;
       }
   }
   ```

   `http2 on;` serves XTM One over HTTP/2 to browsers, which its chat needs. The directive exists from Nginx 1.25.1; on an older Nginx, write `listen 443 ssl http2;` instead of the two lines.

2. Check the configuration and load it.

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

   :::verify
   `nginx -t` ends with `test is successful`. Each platform answers its health check through the proxy:

   ```bash
   set -a; . ./.env; set +a
   curl -fsS "https://$OPENCTI_HOST/health?health_access_key=$OPENCTI_HEALTHCHECK_ACCESS_KEY" >/dev/null && echo "OpenCTI OK"
   curl -fsS "https://$OPENAEV_HOST/api/health?health_access_key=$OPENAEV_HEALTHCHECK_KEY" >/dev/null && echo "OpenAEV OK"
   curl -fsS "https://$XTM_ONE_HOST/api/health" >/dev/null && echo "XTM One OK"
   curl -sS --http2 -o /dev/null -w 'XTM One over HTTP/%{http_version}\n' "https://$XTM_ONE_HOST/api/health"
   ```
   Run it from the `xtm-docker` folder, where `.env` is. The last line prints `XTM One over HTTP/2`; `HTTP/1.1` means the proxy does not serve HTTP/2 yet.
   :::

3. Mark the session cookies as secure, so that browsers send them over HTTPS only. In `docker-compose.override.yml`, add one variable to the `environment` of two services, as [Change the stack without editing docker-compose.yml](/deploy/install/docker-compose/#change-the-stack-without-editing-docker-compose-yml) describes:

   ```yaml
   services:
     opencti:
       environment:
         - APP__HTTPS_CERT__COOKIE_SECURE=true
     openaev:
       environment:
         - OPENAEV_COOKIE-SECURE=true
   ```

   Then apply the change: Docker Compose recreates the two containers.

   ```bash
   docker compose up -d
   ```

   :::verify
   Sign in to OpenCTI and OpenAEV at their public URL. The sign-in holds when you open another page.
   :::
:::

:::warning[Proxy on another host]
OpenCTI takes the client address from `X-Forwarded-For` only when the proxy reaches it from a loopback, link-local or private address. From any other address, OpenCTI sees every request as coming from the proxy: the activity and audit logs record the proxy's address, and the IP-based protections, such as rate limiting, apply to the proxy instead of to each user. The session cookie is not affected: OpenCTI marks it secure from `X-Forwarded-Proto` whatever the proxy's address.
:::

:::only{opencti=">=7.260811"}
If your proxy reaches OpenCTI from a public address, add that address, or its CIDR range, to `TRUST_PROXY_ADDRESSES` (`trust_proxy_addresses`), such as `TRUST_PROXY_ADDRESSES=["203.0.113.10"]`. See [Network and security](/reference/opencti/configuration/#network-and-security).
:::

:::only{opencti="<7.260811"}
No setting adds other addresses: keep the proxy on a private network with OpenCTI.
:::

## Serve a platform under a path

Give each platform its own host name in the XTM stack. Both platforms can be served under a path, such as `https://example.com/opencti`, but the other services of `docker-compose.yml` reach them at the root of their service name (`http://opencti:8080`, `http://openaev:8080`), so every internal URL would have to change too.

| Platform | Setting | Public URL |
| --- | --- | --- |
| OpenCTI | `app:base_path`, environment variable `APP__BASE_PATH=/opencti` | Keep the base URL without the path: OpenCTI adds the base path to it. The WebSocket moves to `/opencti/graphql`. |
| OpenAEV | `server.servlet.context-path=/openaev` | Put the path in `openaev.base-url`: `https://example.com/openaev`. |

## Serve HTTPS from OpenCTI itself

Without a proxy, OpenCTI can serve HTTPS on its own port: set `APP__HTTPS_CERT__KEY` and `APP__HTTPS_CERT__CRT` (`app:https_cert:key` and `app:https_cert:crt`) to the paths of the key and the certificate inside the container, or to their PEM contents, and `APP__HTTPS_CERT__PASSPHRASE` if the key is encrypted. A reverse proxy remains the simpler choice for the whole stack, since OpenAEV and XTM One need TLS too. To trust the certificates of the services the platforms call, see [Trust your certificate authority and proxy](/deploy/configure/certificates/).

## What's next

- [Connect the modules to each other](/deploy/configure/connect-modules/), to check that the platforms still reach each other with their public URLs.
- [Harden the deployment](/deploy/operate/harden/), for security headers, sessions and rate limiting.
- [Check that the platforms are healthy](/deploy/operate/health-probes/), to monitor the platforms through the proxy.
