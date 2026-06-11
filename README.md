# Generate Certs Ansible Role

## Overview

This role is used for creating certificates. Currently, this role only supports self-signed certificates and must be extended accordingly. However, since there is currently no use case for other types of certificates, this role is currently only used as a dependency for docker-compose.

## Requirements

This role has no requirements. All packages required for its use are already included in the installation.

## Variables

Default values are set for almost all variables, so this role can generate the corresponding certificates and keys without any configuration.

If changes are desired, the variables can be overridden within the user’s own group or host vars.

The following variables and defaults are available:

```
O_generate_certs_path: /etc/ssl/self-signed
O_generate_certs_common_name: default
O_generate_certs_key_name: key.pem
O_generate_certs_cert_name: cert.pem
O_generate_certs_csr_name: csr.csr
O_generate_certs_valid_not_after: +36500d
O_generate_certs_alt_names_list: 
  - localhost
  - 127.0.0.1
O_generate_certs_letsencrypt_ordered_domains
O_generate_certs_letsencrypt_domain
```

## Variable details

### Global variables

#### O_generate_certs_path

Base path for all generated files.

### Variables for self-signed certificates

#### O_generate_certs_common_name

CN of the certificate. This is usually the FQDN.

#### O_generate_certs_key_name

Name of the key file.

#### O_generate_certs_cert_name

Name of the certificate file.

#### O_generate_certs_csr_name

Name of the CSR.

#### O_generate_certs_valid_not_after

Validity period/expiration date of the certificate.

#### O_generate_certs_alt_names_list

Alternative names that can be included in the certificate.

### Variables for Let’s Encrypt certificates

#### O_generate_certs_letsencrypt_ordered_domains

This variable contains a list of all domains that should receive a wildcard Let’s Encrypt certificate.

#### O_generate_certs_letsencrypt_domain

This variable is required for certificate distribution. Based on this information, the correct Let’s Encrypt certificate is delivered.
