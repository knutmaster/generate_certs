# generate_certs

Ansible role for creating, validating, renewing, and distributing TLS certificates.

## Features

| Method                      | Description                                            |
| --------------------------- | ------------------------------------------------------ |
| `selfsigned`                | Generate self-signed certificates                      |
| `letsencrypt`               | Deploy existing Let's Encrypt certificates             |
| `letsencrypt` + Hetzner DNS | Generate wildcard certificates using DNS-01 challenges |

Additional features:

* Automatic certificate validation
* Automatic renewal before expiration
* Support for Subject Alternative Names (SAN)
* Configurable certificate paths
* Wildcard certificate support
* Offline mode for Hetzner DNS challenge scripts

---

# Directory Structure

```text
roles/
└── generate_certs/
    ├── defaults/
    │   └── main.yml
    ├── files/
    │   ├── certbot-hetzner-auth.sh
    │   └── certbot-hetzner-cleanup.sh
    ├── handlers/
    │   └── main.yml
    ├── tasks/
    │   ├── main.yml
    │   ├── selfsigned.yml
    │   ├── letsencrypt.yml
    │   └── validate.yml
    ├── templates/
    ├── vars/
    ├── meta/
    └── README.md
```

---

# Requirements

## Collections

```yaml
collections:
  - community.crypto
```

## Supported Operating Systems

* Debian
* Ubuntu

## Required Packages

### General

```text
libssl-dev
python3
python3-cryptography
```

### Let's Encrypt

```text
certbot
```

The role installs all required packages automatically.

---

# Role Variables

## General Variables

| Variable                    | Default            | Description                                                   |
| --------------------------- | ------------------ | ------------------------------------------------------------- |
| `O_generate_certs_method`   | `selfsigned`       | Certificate generation method (`selfsigned` or `letsencrypt`) |
| `O_generate_certs_path`     | `/etc/ssl/private` | Destination directory for certificate files                   |
| `O_generate_certs_valid_at` | `+2W`              | Renewal threshold used for certificate validation             |

### Defaults

```yaml
O_generate_certs_method: selfsigned
O_generate_certs_path: /etc/ssl/private
O_generate_certs_valid_at: +2W
```

---

## Self-Signed Certificate Variables

| Variable                           | Default                      | Description               |
| ---------------------------------- | ---------------------------- | ------------------------- |
| `O_generate_certs_common_name`     | `default`                    | Common Name (CN)          |
| `O_generate_certs_domain`          | `example.de`                 | Domain name               |
| `O_generate_certs_key_name`        | `key.pem`                    | Private key filename      |
| `O_generate_certs_cert_name`       | `cert.pem`                   | Certificate filename      |
| `O_generate_certs_csr_name`        | `csr.csr`                    | CSR filename              |
| `O_generate_certs_valid_not_after` | `+36500d`                    | Certificate expiration    |
| `O_generate_certs_alt_names_list`  | `["localhost", "127.0.0.1"]` | Subject Alternative Names |

### Defaults

```yaml
O_generate_certs_common_name: default
O_generate_certs_domain: example.de

O_generate_certs_key_name: key.pem
O_generate_certs_cert_name: cert.pem
O_generate_certs_csr_name: csr.csr

O_generate_certs_valid_not_after: +36500d

O_generate_certs_alt_names_list:
  - localhost
  - 127.0.0.1
```

---

## Let's Encrypt Variables

| Variable                                       | Default        | Description                                  |
| ---------------------------------------------- | -------------- | -------------------------------------------- |
| `O_generate_certs_letsencrypt_domain`          | undefined      | Existing Let's Encrypt certificate to deploy |
| `O_generate_certs_letsencrypt_ordered_domains` | `[example.de]` | Domains for wildcard certificate generation  |
| `O_generate_certs_letsencrypt_script_path`     | GitHub URL     | Source location for Hetzner DNS scripts      |
| `O_generate_certs_letsencrypt_hetzner_scripts` | List           | DNS challenge helper scripts                 |
| `O_generate_certs_offline`                     | `true`         | Use bundled scripts instead of downloading   |
| `O_hetzner_token`                              | `test`         | Hetzner DNS API token                        |
| `O_admin_mail`                                 | undefined      | Email address for Certbot                    |

### Defaults

```yaml
O_hetzner_token: test

O_generate_certs_letsencrypt_ordered_domains:
  - example.de

O_generate_certs_letsencrypt_script_path: https://raw.githubusercontent.com/knutmaster/generate_certs/main/files

O_generate_certs_letsencrypt_hetzner_scripts:
  - certbot-hetzner-auth.sh
  - certbot-hetzner-cleanup.sh

O_generate_certs_offline: true
```

---

# Usage Examples

## Self-Signed Certificate

```yaml
- hosts: webservers
  become: true

  roles:
    - role: generate_certs
      vars:
        O_generate_certs_method: selfsigned

        O_generate_certs_common_name: web01.example.com
        O_generate_certs_domain: example.com

        O_generate_certs_alt_names_list:
          - web01.example.com
          - www.example.com
          - localhost
```

---

## Deploy Existing Let's Encrypt Certificate

```yaml
- hosts: webservers
  become: true

  roles:
    - role: generate_certs
      vars:
        O_generate_certs_method: letsencrypt
        O_generate_certs_letsencrypt_domain: example.com
```

---

## Generate Wildcard Certificates using Hetzner DNS

```yaml
- hosts: certserver
  become: true

  roles:
    - role: generate_certs
      vars:
        O_generate_certs_method: letsencrypt

        O_admin_mail: admin@example.com
        O_hetzner_token: "{{ vault_hetzner_dns_token }}"

        O_generate_certs_letsencrypt_ordered_domains:
          - example.com
          - example.org
```

Generated certificates:

```text
*.example.com
example.com

*.example.org
example.org
```

---

## Example Playbook

```yaml
---
- name: Generate TLS certificates
  hosts: all
  become: true

  roles:
    - role: generate_certs
      vars:
        O_generate_certs_method: selfsigned
        O_generate_certs_common_name: "{{ inventory_hostname }}"
        O_generate_certs_domain: example.com
```

---

# Certificate Validation

The role checks whether an existing certificate is present and still valid.

Example:

```yaml
O_generate_certs_valid_at: +2W
```

The certificate must remain valid for at least two more weeks.

Further examples:

```yaml
O_generate_certs_valid_at: +30d
```

Renew if less than 30 days remain.

```yaml
O_generate_certs_valid_at: +90d
```

Renew if less than 90 days remain.

---

# Generated Files

Using default values:

```text
/etc/ssl/private/
├── key.pem
├── csr.csr
└── cert.pem
```

---

# Workflow

## Self-Signed Mode

1. Create private key
2. Create CSR
3. Generate self-signed certificate
4. Validate certificate expiration

Modules used:

* `community.crypto.openssl_privatekey`
* `community.crypto.openssl_csr`
* `community.crypto.x509_certificate`

---

## Let's Encrypt Mode

1. Install Certbot
2. Prepare Hetzner DNS hooks
3. Validate existing certificate
4. Generate or renew certificate
5. Deploy certificate files

---

# Dependencies

```yaml
collections:
  - community.crypto
```

---

# License

MIT
