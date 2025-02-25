---
title: Rate Limits
slug: rate-limits
top_graphic: 1
date: 2018-01-04
lastmod: 2021-10-05
show_lastmod: 1
---


Let's Encrypt provides rate limits to ensure fair usage by as
many people as possible. We believe these rate limits are high enough to work
for most people by default. We've also designed them so renewing a
certificate almost never hits a rate limit, and so that large
organizations can gradually increase the number of certificates they can issue
without requiring intervention from Let's Encrypt.

If you're actively developing or testing a Let's Encrypt client, please utilize
our [staging environment](/docs/staging-environment) instead of the production API.
If you're working on integrating Let's Encrypt as a provider or with a large
website please [review our Integration Guide](/docs/integration-guide).

The main limit is <a id="certificates-per-registered-domain"></a>**Certificates per Registered Domain** (50 per week). A
registered domain is, generally speaking, the part of the domain you purchased
from your domain name registrar. For instance, in the name `www.example.com`,
the registered domain is `example.com`. In `new.blog.example.co.uk`,
the registered domain is `example.co.uk`. We use the
[Public Suffix List](https://publicsuffix.org) to calculate the registered
domain. Exceeding the Certificates Per Registered Domain limit is reported with the
error message `too many certificates already issued`, possibly with additional
details.

You can create a maximum of 300 <a
id="new-orders"></a>**New Orders** per account per 3 hours. A new order is created
each time you request a certificate from the Boulder CA, meaning that one new order
is produced in each certificate request. Exceeding the New Orders
limit is reported with the error message `too many new orders recently`.

You can combine multiple hostnames into a single
certificate, up to a limit of 100 <a id="names-per-certificate"></a>**Names per Certificate**.
For performance and reliability reasons, it's better to use fewer names per certificate
whenever you can.  A certificate with multiple names is often called a SAN
certificate, or sometimes a UCC certificate.

Renewals are treated specially: they don't count against your **Certificates per
Registered Domain** limit, but they are subject to a **Duplicate Certificate**
limit of 5 per week. Exceeding the Duplicate Certificate limit is reported with
the error message `too many certificates already issued for exact set of domains`.

A certificate is considered a renewal (or a duplicate) of an earlier certificate if it contains
the exact same set of hostnames, ignoring capitalization and ordering of
hostnames.  For instance, if you requested a certificate for the names
[`www.example.com`, `example.com`], you could request four more certificates for
[`www.example.com`, `example.com`] during the week. If you changed the set of hostnames
by adding [`blog.example.com`], you would be able to request additional
certificates.

Renewal handling ignores the public key and extensions requested. A certificate issuance
can be considered a renewal even if you are using a new key.

**Revoking certificates does not reset rate limits**, because the resources used to
issue those certificates have already been consumed.

There is a <a id="failed-validations"></a>**Failed Validation** limit of 5 failures
per account, per hostname, per hour. This limit is higher on our
[staging environment](/docs/staging-environment), so you
can use that environment to debug connectivity problems. Exceeding the Failed
Validations limit is reported with the error message `too many failed authorizations recently`.

The "new-nonce", "new-account", "new-order", and "revoke-cert" endpoints on the API have an <a
id="overall-requests"></a>**Overall
Requests** limit of 20 per second. The "/directory" endpoint and the "/acme" 
directory & subdirectories have an Overall Requests limit of 40 requests per second.

We have two other limits that you're very unlikely to run into.

You can create a maximum of 10 <a id="accounts-per-ip-address"></a>**Accounts per IP Address** per 3 hours. You can
create a maximum of 500 **Accounts per IP Range** within an IPv6 /48 per
3 hours. Hitting either account rate limit is very rare, and we recommend that
large integrators prefer a design [using one account for many customers](/docs/integration-guide).
Exceeding these limits is reported with the error message `too many registrations for this IP`
or `too many registrations for this IP range`.

You can have a maximum of 300 <a id="pending-authorizations"></a>**Pending Authorizations** on your account. Hitting
this rate limit is rare, and happens most often when developing ACME clients. It
usually means that your client is creating authorizations and not fulfilling them.
Please utilize our [staging environment](/docs/staging-environment) if you’re
developing an ACME client. Exceeding the Pending Authorizations limit is
reported with the error message `too many currently pending authorizations`.

# <a id="overrides"></a>Overrides

If you've hit a rate limit, we don't have a way to temporarily reset it. You'll
need to wait until the rate limit expires after a week. We use a sliding window,
so if you issued 25 certificates on Monday and 25 more certificates on Friday,
you'll be able to issue again starting Monday. You can get a list of certificates
issued for your registered domain by [searching on crt.sh](https://crt.sh), which
uses the public [Certificate Transparency](https://www.certificate-transparency.org)
logs.

If you are a large hosting provider or organization working on a Let's Encrypt
integration, we have a [rate limiting
form](https://goo.gl/forms/plqRgFVnZbdGhE9n1)
that can be used to request a higher rate limit. It takes a few weeks to process
requests, so this form is not suitable if you just need to reset a rate limit
faster than it resets on its own.

# <a id="clearing-pending"></a>Clearing Pending Authorizations

If you have a large number of pending authorization objects and are getting a
Pending Authorizations rate limiting error, you can trigger a validation attempt for those
authorization objects by submitting a JWS-signed POST to one of its challenges, as
described in the
[ACME spec](https://tools.ietf.org/html/rfc8555#section-7.5.1).
The pending authorization objects are represented by URLs of the form
`https://acme-v02.api.letsencrypt.org/acme/authz/XYZ`, and should show up in your
client logs. Note that it doesn't matter whether validation succeeds or fails.
Either will take the authorization out of 'pending' state. If you do not
have logs containing the relevant authorization URLs, you need to wait for the
rate limit to expire. As described above, there is a sliding window, so this may
take less than a week depending on your pattern of issuance.

Note that having a large number of pending authorizations is generally the
result of a buggy client. If you're hitting this rate limit frequently you
should double-check your client code.
