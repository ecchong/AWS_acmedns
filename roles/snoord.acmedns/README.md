<h1 align="center" style="border-bottom: none;">Ansible Role: ACME (DNS challenge)</h1>

This Ansible role generates a certificate from LetsEncrypt
using the DNS challenge method.
Initially it will only support domains
that are hosted using either [Cloudflare](https://cloudflare.com) OR [AWS Route53](https://aws.amazon.com/route53/),
but it should be relatively straight-forward
to add support for other DNS providers in the future.

The motivation for creating this role
was that I needed a simple and straight-forward way
to generate LetsEncrypt certificates,
one that could also work independently of
any external services
(such as Traefik or Certbot).

By not having this process
tied to any particular service,
I can easily integrate it
with everything else
that I already use
within my environment -
namely Consul, Nomad and Vault.

## Requirements

* Linux
* `openssl`

## Usage

There are only a few key variables that this role requires
for generating the LetsEncrypt certificates:
* 1\) DNS provider variables; _and_
* 2\) LetsEncrypt certificate variables
---

### 1) DNS provider variables
To use one of the following DNS providers,
configure the relevant role variables:
* __a)__ AWS Route53: variable prefix = `acmedns_r53_`
* __b)__ Cloudflare: variable prefix = `acmedns_cf_`

#### a) AWS Route53
```yaml
acmedns_provider: r53
acmedns_r53_zone: Z123EXAMPLE      # Hosted-zone ID to use when generating the certificate
acmedns_r53_access_key: ACCESSKEY  # Access key of an account with permission to add records to the hosted zone
acmedns_r53_secret_key: SECRETKEY  # Secret key of an account with permission to add records to the hosted zone
```

#### b) Cloudflare
```yaml
acmedns_provider: cf
acmedns_cf_zone: example.com        # Cloudflare DNS zone to use when generating the certificate
acmedns_cf_email: user@example.com  # Email address of your Cloudflare account
acmedns_cf_token: EXAMPLETOKEN      # Zone-specific API Token with 'Zone:DNS:Edit' and 'Zone:Zone:Read' permissions
```
---

### 2) LetsEncrypt certificate variables
To keep configuration simple,
you only need
to specify the following variables
for the _to-be-generated_ certificate:
```yaml
acmedns_le_cn: test.example.com     # Common Name to use for the generated certificate
acmedns_le_email: user@example.com  # Email address of your LetsEncrypt account (created if non-existent)
acmedns_le_sans:                    # (optional) List of Subject Alternative Names for the generated certificate
  - 'another.test.example.com'
  - '*.test.example.com'
```
---

### Other variables
This role will look for
a LetsEncrypt account key named `letsencrypt.pem`
within the `~/.local/share/letsencrypt` directory.
This is also where all certificates
will be generated and saved.

You can specify the path
to your LetsEncrypt account key
and change the directory
where certificates are saved
by modifying the following variables:
```yaml
acmedns_le_dir: /etc/acme                  # default: ~/.local/share/letsencrypt
acmedns_le_accountkey: /etc/acme/acme.key  # default: ~/.local/share/letsencrypt/letsencrypt.pem
```

## Example Playbook

*AWS Route53:*
```yaml
---
- hosts: localhost
  roles:
    - role: snoord.acmedns
      vars:
        # LetsEncrypt
        acmedns_le_email: user@example.com
        acmedns_le_cn: example.com
        acmedns_le_sans:
          - '*.example.com'
        # AWS Route53
        acmedns_provider: r53
        acmedns_r53_zone: Z123EXAMPLE
        acmedns_r53_access_key: ACCESSKEY
        acmedns_r53_secret_key: SECRETKEY
...
```
*Cloudflare:*
```yaml
---
- hosts: localhost
  roles:
    - role: snoord.acmedns
      vars:
        # LetsEncrypt
        acmedns_le_email: user@example.com
        acmedns_le_cn: example.com
        acmedns_le_sans:
          - '*.example.com'
        # Cloudflare
        acmedns_provider: cf
        acmedns_cf_zone: example.com
        acmedns_cf_email: user@example.com
        acmedns_cf_token: EXAMPLETOKEN
...
```

## License

MIT / BSD

## Author Information

Created by Samuel Noordhuis in 2020. Inspired heavily by the Ansible roles and writings of [Jeff Geerling](https://github.com/geerlingguy).

If you see any errors or think this role could be improved in some way, you are welcome to open an issue/feature request or create a pull request :)
