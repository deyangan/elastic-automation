# Certificates (not stored in Git)

This directory is where per-node PKCS#12 keystores/truststores and the internal
CA certificate must be placed **before** running the playbook against this
environment.

Required files (named after each `inventory_hostname`):

```
<inventory_hostname>.p12        # node keystore (TLS + transport)
<cluster>-truststore.p12        # shared truststore
ca.pem                          # internal CA certificate (if using AD/LDAPS)
elasticsearch.keystore          # ES secure settings keystore
```

**Do not commit real certificates, keystores, or `.keystore` files to this
repository.** They are excluded via `.gitignore`. Distribute them via your
secrets manager (HashiCorp Vault, AWS Secrets Manager, Ansible Vault-encrypted
files stored outside this repo, etc.) and copy them onto the control node (or
have the pipeline pull them at runtime) immediately before the play runs.
