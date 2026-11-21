---
title: Use your own domain for phishing links
description: Register a hostname you own for phishing links, prove it with a DNS TXT record, give it a certificate at your edge, and serve a landing page on it.
doc_type: how-to
products:
  - openaev
personas: [admin, platform-engineer]
owner: docs-openaev
last_reviewed: 2026-11-07
migrated_from:
  - openaev:/usage/build/components/phishing/custom-domains/
---

By default, the link of a phishing email points at the address of your OpenAEV platform. A custom domain lets the link and the address bar show a hostname you own instead, such as `reservations.grand-meridian.example`, so the lure looks real and the testing platform stays out of sight. This page registers the hostname, proves you own it in DNS, sets up its certificate, and links it to a landing page. The domain changes the link only: the sender of the email is set on the [email template](/openaev/use/phishing/email-templates/).

:::warning
Register only hostnames your organization owns and has authorized for awareness exercises.
:::

## Before you start

- Your role has **Manage tenant settings**: custom domains are a tenant setting. It covers adding, verifying and deleting them.
- You can publish DNS records in the zone of the hostname.
- An edge in front of the platform, such as a reverse proxy, can serve HTTPS for the new hostname: the links always use `https://`.

## Register the domain

:::steps

1. Open :menu[Settings > Customization > Custom domains].
2. Click **Add a custom domain**, enter the hostname in **Domain**, and click **Create**. Enter the bare hostname: no `https://`, path, port or wildcard.
3. The **Configure DNS** panel opens with the two records to publish. Copy them.
4. At your DNS provider, publish both records (see below).
5. Back in the panel, click **Verify domain**. If the records have not reached DNS yet, wait a few minutes and click again.

:::

::screenshot{id=openaev-phishing-custom-domains-list alt="Settings, Customization, Custom domains: reservations.grand-meridian.example, Pending verification, not verified yet, with the Add a custom domain button."}

The domain is **Verified** when the panel shows **This domain is verified and can be linked to phishing landing pages.**, with the date. The domain list shows its **Status** and **Verified on**; its actions menu reopens **DNS setup & verification**, runs **Verify now**, or deletes it.

::diagram{id=openaev-phishing-custom-domains-dns}

## The two DNS records

::screenshot{id=openaev-phishing-custom-domains-dns alt="The DNS setup & verification panel of reservations.grand-meridian.example, Pending verification, with the Verify domain button: the CNAME record of the hostname pointing to the platform, and the TXT record _openaev-challenge.reservations.grand-meridian.example holding the openaev-domain-verification token."}

| Record | **Record type** | **Name / host** | **Value** |
| --- | --- | --- | --- |
| **Point your domain to the platform (CNAME record)** | `CNAME` | The hostname | The host of the platform's address, `openaev.base-url` |
| **Prove ownership (TXT record)** | `TXT` | `_openaev-challenge.` followed by the hostname | A token that starts with `openaev-domain-verification=` |

The `TXT` record is what proves you control the zone: anyone can point a name at the platform, but only the zone's owner can publish the token. It is required, and a hostname belongs to one tenant only: another tenant that tries to add it gets **This domain is already registered**.

When your provider cannot create a `CNAME` at that name, at the apex of a zone for instance, publish an `A` record to the address of your edge instead. The `CNAME` value is the host of `openaev.base-url`: if that address only resolves inside your network, point the record at the public address of your edge.

## When the verification fails

The platform looks the `TXT` record up with the DNS resolvers of the machine it runs on, and stops after a few seconds. The panel says why the last attempt failed and when it ran (**Last checked**):

| Message | What to do |
| --- | --- |
| No TXT record found at `_openaev-challenge.…` | The record is missing, or has not reached the platform's resolvers yet: check its name, then try again |
| TXT record at `_openaev-challenge.…` does not match the expected value yet | Copy the value again from the panel, without spaces or extra quotes |
| DNS lookup failed | The platform could not reach a resolver: check the DNS setup of its host |

The status moves from **Pending verification** to **Verified**, or to **Verification failed** after a failed attempt. Verification runs only when you click: nothing checks the domain again on its own. A verified domain stays verified even if a later check fails, so a DNS hiccup never breaks a running exercise.

## Give the domain a certificate

The platform does not serve HTTPS for your hostnames itself: your edge does, and forwards the requests to the platform. To let the edge obtain a certificate only for your verified domains, OpenAEV answers an ownership check:

```http
GET /api/hosted/domain-check?domain=reservations.grand-meridian.example
```

It answers `200` for a verified custom domain, and `404` for any other hostname. It needs no authentication. An edge that issues certificates on demand, such as Caddy with `on_demand_tls`, calls it before each new certificate, so a hostname nobody verified never gets one.

Let the edge pass the `X-Forwarded-For` header to the platform: the results show the first address it holds as the origin of a click.

## Serve a landing page on the domain

:::steps

1. Open the landing page from :menu[Components > Phishing], then **Update** in its actions menu.
2. In **Serve on domain**, choose the hostname. Only verified domains are listed.
3. Click **Update**.

:::

The emails sent from then on carry links such as `https://reservations.grand-meridian.example/auth/` followed by each person's token. Emails already sent keep the links they were sent with.

Deleting a custom domain does not delete its landing pages: they go back to the **Platform default domain**. Delete a domain once the exercises that use it are over, since the links already sent still point at it.

:::example[Serve the summit booking portal on a Grand Meridian hostname]
Grand Meridian Hotels serves its phishing pages on `reservations.grand-meridian.example`, behind a Caddy edge. Its OpenAEV platform answers at `https://aev.grand-meridian.example` (`openaev.base-url`), and the Caddy edge reaches it at `openaev:8080`.

The administrator adds the domain, then publishes the records the **Configure DNS** panel gives:

```text
reservations.grand-meridian.example.                     CNAME  aev.grand-meridian.example.
_openaev-challenge.reservations.grand-meridian.example.  TXT    "openaev-domain-verification=…"
```

The value of the `TXT` record is the whole token the panel shows, cut short here. After **Verify domain**, the domain is **Verified**. The platform engineer adds the hostname to the Caddy edge, which asks OpenAEV before each certificate:

```caddyfile
{
	on_demand_tls {
		ask http://openaev:8080/api/hosted/domain-check
	}
}

https:// {
	tls {
		on_demand
	}
	reverse_proxy openaev:8080
}
```

The awareness manager sets **Serve on domain** to `reservations.grand-meridian.example` on the landing page **EHS 2026 booking portal**. The next exercise's emails link to `https://reservations.grand-meridian.example/auth/…`, and the first visit gets its certificate from the edge.
:::

## What's next

- [Build the landing page of a phishing exercise](/openaev/use/phishing/landing-pages/) and link it to the domain.
- [Send a phishing exercise to your teams](/openaev/use/phishing/run/).
- The [`openaev` block of the configuration reference](/reference/openaev/configuration/#openaev), for `openaev.base-url`.
