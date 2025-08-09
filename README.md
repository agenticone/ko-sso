# ko-sso
Minimal SAML SSO lab with Keycloak (IdP) + OpenLDAP (user store) + SimpleSAMLphp (SP).

## Phases
- Phase A: Local PoC on Vagrant + Hyper-V (Docker Compose)
- Phase B: Elasticity/HA on k3s + Helm
- Phase C: Cloud rollout (Terraform for GCP/AWS) + Ansible/Helm

See `vagrant/`, `compose/`, `ansible/`, `k8s/`, and `terraform/` folders.
