# Red Hat Trusted Profile Analyzer Ansible collection

The purpose of this Ansible collection is to automate the deployment of the Red Hat Trusted Profile Analyzer (RHTPA) service on Red Hat Enterprise Linux (RHEL).

> [!IMPORTANT]
Deploying RHTPA by using Ansible is a Technology Preview feature only.
Technology Preview features are not supported with Red Hat production service level agreements (SLAs), might not be functionally complete, and Red Hat does not recommend to use them for production.
These features provide early access to upcoming product features, enabling customers to test functionality and provide feedback during the development process.
See the support scope for [Red Hat Technology Preview](https://access.redhat.com/support/offerings/techpreview/) features for more details.


## Description

The RHTPA service is the downstream redistribution of the [Trustification](https://github.com/trustification/trustify) project.
The automation contained within this Git repository installs and configures the components of RHTPA to run on a single RHEL server by using a standalone containerized deployment. A Kubernetes-based manifest creates containers that uses [`podman kube play`](https://docs.podman.io/en/latest/markdown/podman-kube-play.1.html).

The RHTPA Ansible collection deploys the following RHTPA components:

- [Trustification](https://github.com/trustification/trustify)

## Prerequisites

A RHEL 9.3+ server should be used to run the Trustification components.

Install and configure Ansible on a control node before performing the automated deployment.

## Minimum hardware requirements 
* 24 vCPU, 
* 48 GB Ram, 
* 100 GB Disk space 

## Requirements

* Ansible 2.16.0 or greater
* Python 3.10.0 or greater
* RHEL x86\_64 9.3 or greater.
* Installation and configuration of Ansible on a control node to perform the automation.

You must provide the following external services:

* An OpenID Connect (OIDC) provider, such [RedHat Single Sign On](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/sso/) or Amazon Web Services (AWS) Cognito.
* A new PostgreSQL database.

## External Services Configurations

### RedHat Single Sign On

* [Trustification Keycloak](https://github.com/trustification/trustification/blob/release/1.2.z/docs/modules/admin/pages/cluster-preparing.adoc#keycloak)

### Postgresql

Create a PostgreSQL database and configure your database credentials in the  environment variables, see 'Verifying the deployment section', 
other database configurations are in the roles/tpa_single_node/vars/main.yml

* [Trustification-PostgreSQL](https://github.com/trustification/trustification/blob/release/1.2.z/docs/modules/admin/pages/cluster-preparing.adoc#rds)

Utilize the steps below to understand how to setup and execute the provisioning.

## Configurations on the controller node

On the controller node export the following environment variables:

1. Export the following environment variables, replacing the placeholders with your relevant information:

   ```shell
      export TPA_SINGLE_NODE_REGISTRY_USERNAME=<Your Red Hat image registry username>
      export TPA_SINGLE_NODE_REGISTRY_PASSWORD=<Your Red Hat image registry password>
      export TPA_PG_HOST=<POSTGRES HOST IP>
      export TPA_PG_ADMIN=<DB ADMIN>
      export TPA_PG_ADMIN_PASSWORD=<DB ADMIN PASSWORD>
      export TPA_PG_USER=<DB USER>
      export TPA_PG_USER_PASSWORD=<DB PASSWORD>
      export TPA_OIDC_ISSUER_URL=<AWS Cognito or Keycloak Issuer URL. Incase of Keycloak endpoint auth/realms/chicken is needed>
      export TPA_OIDC_FRONTEND_ID=<OIDC Frontend Id>
      export TPA_OIDC_PROVIDER_CLIENT_ID=<OIDC Walker Id>
      export TPA_OIDC_PROVIDER_CLIENT_SECRET=<OIDC Walker Secret>
   ```

2. Choose between Keycloak or AWS Cognito, and update the `roles/tpa_single_node/defaults/main.yml` file accordingly.

3If you are using AWS Cognito as your OIDC provider, then create an environment variable pointing to the Cognito domain:
```shell
export TPA_OIDC_COGNITO_DOMAIN=<AWS Cognito Domain>
```

## Provision

In order to deploy Trustification on a RHEL 9.3+ VM:

1. Update the content of the `inventory.ini` file in the project:

```
[trustification]
<IP_TARGET_MACHINE>

[trustification:vars]
ansible_user=<username>
ansible_ssh_pass=<ssh_password>
ansible_private_key_file=<path to private key>
```

2. Configure if needed the `ansible.cfg` file in the project:

```
[defaults]
inventory = ./inventory.ini
host_key_checking = 
```


3. Path for TLS certificates files:

Copy your certificate files in `certs/` directory using following names:

- trust-cert.crt
- trust-cert.key
- rootCA.crt

Optionally, the certs directory variable `tpa_single_node_certificates_dir` under `roles/tpa_single_node/vars/main.yml` file can also be updated with a directory certs for below variables:

- tpa_single_node_root_ca

- tpa_single_node_trust_cert_tls_crt_path
- tpa_single_node_trust_cert_tls_key_path

Refer `roles/tpa_single_node/vars/main_example_aws.yml` and `roles/tpa_single_node/vars/main_example_nonaws.yml`


## Installation

Run this collection

```shell
export ANSIBLE_ROLES_PATH="roles/" ; 
ansible-playbook -i inventory.ini play.yml -vv
```

## Contributing


## Support

Support tickets for RedHat Trusted Profile Analyzer can be opened at https://access.redhat.com/support/cases/#/case/new?product=Red%20Hat%20Trusted%20Profile%20Analyzer.

## Release notes and Roadmap

Release notes can be found [here](https://docs.redhat.com/en/documentation/red_hat_trusted_profile_analyzer/1.2/html/release_notes/index).

## Related Information

More information around Red Hat Trusted Profile Analyzer can be found [here](https://access.redhat.com/products/red-hat-trusted-profile-analyzer).

## Feedback

Any and all feedback is welcome. Submit an [Issue](https://github.com/trustification/trustification-ansible/issues) or [Pull Request](https://github.com/trustification/trustification-ansible/pulls) as desired.

## License Information

License Information cna be found within the [LICENSE](https://github.com/trustification/trustification-ansible/blob/main/LICENSE) file.
