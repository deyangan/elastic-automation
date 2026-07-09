# Security & Sanitization Notes

This repository was sanitized from an internal copy before being made
public. This file documents exactly what was changed/removed, and what you
still need to do yourself.

## What was removed entirely (not just renamed)

- **All `.p12` keystore/truststore files** and **`elasticsearch.keystore`**
  files under `roles/elasticsearch_deploy/files/*/certs/`. These contained
  real private keys and certificates and can never be safely published,
  sanitized or not. A `README.md` placeholder now lives in each `certs/`
  folder explaining what belongs there and that it must come from your
  secrets manager, not from Git.
- **The internal CA certificate** (`apb_ca.pem`) used for LDAPS — also
  removed for the same reason.
- **Git history.** This repo was re-initialized with no history. The
  original `.git` history contained the real certificates/keystores in
  earlier commits; simply deleting files in a new commit does **not**
  remove them from history. If you ever push the *original* repo's history
  publicly, treat every secret in it as compromised (see below).

## What was replaced with placeholders

- Two `ansible-vault`-encrypted secrets (a Kibana service-account password
  and a saved-objects encryption key) — replaced with
  `{{ vault_kibana_es_password }}` / `{{ vault_kibana_encryption_key }}`
  variable references. Keep the real encrypted values in a private,
  gitignored vars file or your secrets manager.
- A hardcoded Elasticsearch API key (base64) in the AT environment —
  replaced with `{{ vault_es_health_apikey }}`.
- Two plaintext application passwords in `inventories/dev-sit/group_vars/`
  — replaced with `{{ vault_apps_paswd }}` / `{{ vault_weblogic_paswd }}`.
- `roles/elasticsearch_deploy/files/*/roles/role_mapping.yml` — the real
  files mapped Elasticsearch roles to **named employees' and consultants'**
  Active Directory distinguished names (who has `superuser` access, etc.).
  These were replaced with a generic example structure. **Do not
  regenerate this file with real names/groups in a public repo** — keep
  your real mapping private and pull it in at deploy time.

## What was genericized

- The internal domain (`*.apoteket.se` and the AD domain `apb.apoteket.se`)
  was replaced with `example.com` / `ad.example.com` throughout.
- Real server hostnames (e.g. `apo25010228.apoteket.se`,
  `apov2prelaapp05.apl.apoteket.se`) were replaced with descriptive
  placeholders (`es-prod-node4.example.com`, etc.) that preserve the
  cluster topology so the example stays readable.
- Jenkins agent labels, a real employee's support email address, and an
  internal wiki link were replaced with generic equivalents.

## Your responsibility before/after publishing

1. **Rotate every real secret** that ever existed in this repo or its
   original Git history — the Elasticsearch API key, the two vault-
   encrypted values, and the two plaintext dev-sit passwords — even though
   they're now redacted here. If they were ever committed anywhere, treat
   them as burned.
2. **Do not push the original repository's Git history publicly.** Publish
   this sanitized tree as a fresh repository (`git init`), not as a filtered
   version of the old one — `git filter-repo`/BFG can miss things, and a
   clean start is safer.
3. **Run a secret scanner** (e.g. `gitleaks detect` or `trufflehog
   filesystem .`) over the final tree — and over your full commit history if
   you ever do decide to rewrite rather than restart — before pushing.
4. **Keep real certs, vault passwords, and role mappings out of Git
   entirely**, public or private. Use Ansible Vault with a password stored
   outside the repo, or a secrets manager (HashiCorp Vault, AWS Secrets
   Manager, Azure Key Vault, etc.) and have Jenkins inject them at runtime,
   as the pipeline already does for `ANSIBLE_PASSWORD`/`BECOME_PASSWORD`.
