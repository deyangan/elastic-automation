# Elasticsearch Ansible Ops

Ansible automation to install, configure, and upgrade an on-prem
Elasticsearch + Kibana cluster, orchestrated through Jenkins.

- 📖 **[DEPLOYMENT.md](DEPLOYMENT.md)** — full write-up of the architecture,
  the Jenkins pipeline, what the Ansible role does step by step, and how to
  run it (with diagrams).
- 🔒 **[SECURITY.md](SECURITY.md)** — what was sanitized from the internal
  version of this repo, and what you must do before/after publishing your
  own copy.

## Quick start

```bash
pip install ansible ansible-lint
ansible-lint playbooks/ roles/

# Dry run against a single AT node
ansible-playbook -i inventories/at/hosts playbooks/deploy-elastic.yml \
  -u <your-ad-account> --become --limit es-at-node1 \
  --check --diff --ask-pass --ask-become-pass
```

In practice, this is triggered through the **`Jenkinsfile.deploy-elastic`**
pipeline (see `jenkinsfiles/`), which exposes `ENV`, `LIMIT`, `UPGRADE`,
`ES_VERSION`, and a `DRY_RUN` toggle as build parameters — no direct shell
access to the control node needed. Full details in
[DEPLOYMENT.md](DEPLOYMENT.md).

## Layout

```
playbooks/deploy-elastic.yml   # entry point
roles/elasticsearch_deploy/    # install / configure / upgrade Elasticsearch
roles/elk_validate/            # pre/post upgrade health & readiness checks
inventories/{at,production}/   # per-environment hosts + group_vars
jenkinsfiles/                  # CI lint job + the deployment pipeline
```

## Requirements

- Ansible ≥ 2.15, `ansible-lint` for CI
- Elasticsearch/Kibana 8.x (see `es_version`/`kb_version` in each
  environment's `group_vars`)
- Real certificates and secrets are **not** included — see
  [SECURITY.md](SECURITY.md) for what you need to supply.
