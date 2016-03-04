<!--

---
lang: american
---
-->

# Let's Encrypt

[Ansible](http://ansible.com) role to generate
[Let's Encrypt](https://letsencrypt.org/) SSL certificates.

## Usage

Include the `letsencrypt` module to your playbook.

## Description

This module:

* Generate server administrator private and public key pair and register it
  to Let's Encrypt servers.

* Generate domain private key and CSR, register it to Let's Encrypt servers
  and ask for a domain validation token.

* Deploy the domain validation token in the well-known URI to the running
  web server. The tokens are deployed to one and only one server (see proxy
  setting below if you are using load balanced architecture).

* Validate the signature and issue a signed certificate (PEM format).

* Cleanup all temporary files (validation token).

This module does *not*:

* Require a service shutdown, it deploys token into the webserver document
  root.

* Deploy generated certificates to the webserver, you need to deploy them
  manually or write an Ansible task for that. The main reason is that you
  may have an appliance that does not support automation.


To ensure idempotent runs, a certificate won't be reissued unless the
minimal number of days before expiration is reached or the certificate is
expired.

## Configuration

See the [defaults](defaults/main.yml) file for more details.

Please note that due to
[Let's Encrypt rate limiting](https://community.letsencrypt.org/t/rate-limits-for-lets-encrypt/6769)
by default the module runs against the staging environment. This you have to
explicitly specify to not use the staging.

## Requirements

You only need Ansible and OpenSSL command line utility. No Python specific
module is required.

## How this works

All keys and certificates are generated locally (on the machine that runs
Ansible) and stored in a storage directory. The role runs 2 stages:

* stage 1: generate all keys and certificates and ask for tokens.
* stage 1.5: Deploy tokens to the websevers (this phase is a bunch of pure
  Ansible tasks).
* stage 2: Validate tokens and get the signed certificate.

For further details on the ACME protocol see Let's Encrypt
[details](https://letsencrypt.org/how-it-works/).

## Proxy setting

This module does not deploy the Domain Validation challenges on more than
one host, thus the validation may fail. You still can configure a proxy to
redirect the queries to the server on which the challenges are deployed.

### NginX


Use the following proxy command on each virtual server that need to be
secured:

```
location /.well-known/acme-challenge/ {
   proxy_pass    http://letsencrypt.example.com;
   proxy_set_header X-Forwarded-For $remote_addr;
   proxy_redirect   off;
}
```

### Brocade ZXTM load balancers

A simple `LetsEncrypt` rule can be:

```
if( string.startsWith( http.getPath(), "/.well-known/acme-challenge/" ) ) {
    pool.use("LETSENCRYPT HTTP");
}
```

The `LETSENCRYPT HTTP` pool should redirect to
*letsencrypt.example.com:80* is your `host` is `letsencrypt.example.com`.

Now use the `LetsEncrypt` rule at the very beginning of you virtual servers.


## Source

This role is heavily inspired from [letsencrypt-nosudo]
(https://github.com/diafygi/letsencrypt-nosudo).



## Copyright

Author: Sébastien Gross `<seb•ɑƬ•chezwam•ɖɵʈ•org>` [@renard_0](https://twitter.com/renard_0)

License: WTFPL, grab your copy here: http://www.wtfpl.net
