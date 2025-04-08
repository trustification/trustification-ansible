# Red Hat Trusted Profile Analyzer Ansible collection

The purpose of this Ansible collection is to automate the deployment of the Red Hat Trusted Profile Analyzer (RHTPA) service on Red Hat Enterprise Linux (RHEL).

**IMPORTANT:**
Deploying RHTAS by using Ansible is a Technology Preview feature only.
Technology Preview features are not supported with Red Hat production service level agreements (SLAs), might not be functionally complete, and Red Hat does not recommend to use them for production.
These features give early access to upcoming product features, enabling customers to test functionality and give feedback during the development process.
See the support scope for [Red Hat Technology Preview](https://access.redhat.com/support/offerings/techpreview/) features for more details.

## Description

The RHTPA service is the downstream redistribution of the [Trustification](https://github.com/trustification/trustification) project.
The automation contained within this Git repository installs and configures the components of RHTPA to run on a single RHEL server by using a standalone containerized deployment. A Kubernetes-based manifest creates containers that uses [`podman kube play`](https://docs.podman.io/en/latest/markdown/podman-kube-play.1.html).

The RHTPA Ansible collection deploys the following RHTPA components:

- [Trustification](https://github.com/trustification/trustification)
- [Guac](https://github.com/trustification/guac)

An [NGINX](https://www.nginx.com) front end places an entrypoint to the RHTPA UI.

## Prerequisites

* Red Hat Enterprise Linux server version 9.3 or higher server.
* Install and configure Ansible on a control node before performing the automated deployment.

## Minimum hardware requirements 

* 24 vCPU, 
* 48 GB Ram, 
* 100 GB Disk space 

## Requirements

* Ansible 2.16.0 or greater
* Python 3.10.0 or greater
* RHEL x86\_64 9.3 or greater.
* Installation and configuration of Ansible on a control node to perform the automation.

You must supply the following external services:

* An OpenID Connect (OIDC) provider, such [RedHat Single Sign On](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/sso/) or Amazon Web Services (AWS) Cognito.
* Simple Queue Service (SQS), for example, [Red Hat AMQ Streams](https://console.redhat.com/ansible/automation-hub/repo/published/redhat/amq_streams/)
* A new PostgreSQL database.
* AWS Simple Storage Service (S3) or an S3-compatible service, for example, MinIO.

## External Services Configurations

### RedHat Single Sign On

* [Trustification Keycloak](https://github.com/trustification/trustification/blob/release/1.2.z/docs/modules/admin/pages/cluster-preparing.adoc#keycloak)


### RedHat Kafka streams  
  With the following topic names created:
```
  bombastic-failed-default
  bombastic-indexed-default
  bombastic-stored-default
  vexination-failed-default
  vexination-indexed-default
  vexination-stored-default
  v11y-failed-default
  v11y-indexed-default
  v11y-stored-default
```

Configure these topic names in the `roles/tpa_single_node/vars/main.yml` file.

* [Trustification event queues](https://github.com/trustification/trustification/blob/release/1.2.z/docs/modules/admin/pages/cluster-preparing.adoc#event-queues)

### PostgreSQL

Create a PostgreSQL database and configure your database credentials in the environment variables.
See 'Verifying the deployment section' for details.
You can use other database configurations in the `roles/tpa_single_node/vars/main.yml` file.

Postgres SSL mode is enabled by default.
To disable SSL, you can change the following line in the `roles/tpa_single_node/vars/main.yml` file: `tpa_single_node_pg_ssl_mode: disable`.

* [Trustification-PostgreSQL](https://github.com/trustification/trustification/blob/release/1.2.z/docs/modules/admin/pages/cluster-preparing.adoc#rds)

### S3 or S3 compatible service
  Have the following unversioned S3 bucket names created:
  ```
  bombastic-default
  vexination-default
  v11y-default 
  ```
Configure these S3 bucket names in the `roles/tpa_single_node/vars/main.yml` file.

* [Trustification S3](https://github.com/trustification/trustification/blob/release/1.2.z/docs/modules/admin/pages/cluster-preparing.adoc#s3-storage)

## Installation

The following steps guide you on how to configure, and provision RHTPA to run on Red Hat Enterprise Linux.

### Configuring the controller node

On the controller node, export the following environment variables replacing the placeholders with your relevant information:

```
export TPA_SINGLE_NODE_REGISTRY_USERNAME=<Your Red Hat image registry username>
export TPA_SINGLE_NODE_REGISTRY_PASSWORD=<Your Red Hat image registry password>
export TPA_PG_HOST=<POSTGRES HOST IP>
export TPA_PG_ADMIN=<DB ADMIN>
export TPA_PG_ADMIN_PASSWORD=<DB ADMIN PASSWORD>
export TPA_PG_USER=<DB USER>
export TPA_PG_USER_PASSWORD=<DB PASSWORD>
export TPA_STORAGE_ACCESS_KEY=<Storage Access Key>
export TPA_STORAGE_SECRET_KEY=<Storage Secret Key>
export TPA_OIDC_ISSUER_URL=<AWS Cognito or Keycloak Issuer URL. Incase of Keycloak endpoint auth/realms/chicken is needed>
export TPA_OIDC_FRONTEND_ID=<OIDC Frontend Id>
export TPA_OIDC_PROVIDER_CLIENT_ID=<OIDC Walker Id>
export TPA_OIDC_PROVIDER_CLIENT_SECRET=<OIDC Walker Secret>
export TPA_EVENT_ACCESS_KEY_ID=<Kafka Username or AWS SQS Access Key>
export TPA_EVENT_SECRET_ACCESS_KEY=<Kafka User Password or AWS SQS Secret Key>
```
   
Choose between AWS S3 or an S3-compatible service, and update the `roles/tpa_single_node/defaults/main.yml` file accordingly.

Choose between Keycloak or AWS Cognito, and update the `roles/tpa_single_node/defaults/main.yml` file accordingly.

In case of Minio, create environmental variable for storage endpoint:

```
export TPA_STORAGE_ENDPOINT=<Minio storage URL >
```

For Kafka events, create an environment variable pointing to the bootstrap server:

```
export TPA_EVENT_BOOTSTRAP_SERVER=<Kafka Bootstrap Server>
```

If you use AWS Cognito as your OIDC provider, then create an environment variable pointing to the Cognito domain:

```
export TPA_OIDC_COGNITO_DOMAIN=<AWS Cognito Domain>
```

### Updating the inventory and the playbook

To deploy RHTPA on a Red Hat Enterprise Linux version 9.3 or later do the following:

Update the content of the `inventory.ini` file in the project:

```
[trustification]
<IP_TARGET_MACHINE>

[trustification:vars]
ansible_user=<username>
ansible_ssh_pass=<ssh_password>
ansible_private_key_file=<path to private key>
```

Configure if needed the `ansible.cfg` file in the project:

```
[defaults]
inventory = ./inventory.ini
host_key_checking = 
```

Copy your certificate files into the `certs/` directory with the following names:

```
trust-cert.crt
trust-cert.key
rootCA.crt
```

Optionally, you can update the variable `tpa_single_node_certificates_dir` in the `roles/tpa_single_node/vars/main.yml` file with the `certs/` directory.
You can also give the specific certificate names for the associated variables:

```
tpa_single_node_root_ca
tpa_single_node_trust_cert_tls_crt_path
tpa_single_node_trust_cert_tls_key_path
tpa_single_node_nginx_tls_crt_path
tpa_single_node_nginx_tls_key_path
```

Update the `roles/tpa_single_node/vars/main.yml` file with the appropriate values for each service.

Storage Service:

* Update the Storage type, either `s3` or `minio`.
* Update the S3 or Minio bucket names.
* Update the AWS region for AWS S3 or keep `us-west-1` for Minio.
* In case of Minio, update the Minio storage end point `tpa_single_node_storage_endpoint`.

SQS Service:

* Update the Event bus type, either `kafka` or `sqs`.
* Update the topics for each events.
* In case of Kafka, update the fields `tpa_single_node_kafka_security_protocol` and `tpa_single_node_kafka_auth_mechanism`.
* In case of AWS SQS, update the AWS SQS region `tpa_single_node_sqs_region`.

Refer to `roles/tpa_single_node/vars/main_example_aws.yml` and `roles/tpa_single_node/vars/main_example_nonaws.yml` for more details.

### Collection installation

Option 1 : Install RHTPA by using the Ansible Galaxy command-line tool.

```
ansible-galaxy collection install redhat.trusted_profile_analyzer
```

Option 2 : Include the collection in a `requirements.yml` file by using this format:

```yaml
collections:
  - name: redhat.trusted_profile_analyzer
```

Then install RHTPA with the `ansible-galaxy collection install -r requirements.yml` command.

Option 3 : Run the Ansible playbook.

```
export ANSIBLE_ROLES_PATH="roles/" ; 
ansible-playbook -i inventory.ini play.yml -vv
```

**NOTE:** If you install any collection from Ansible Galaxy, upgrading the Ansible package is not automatically done.

To upgrade the collection to the latest available version, run the following command:

```
ansible-galaxy collection install redhat.trusted_profile_analyzer --upgrade
```

You can also install a specific version of the collection.
For example, if you need to rollback to an earlier version.

```
ansible-galaxy collection install redhat.trusted_profile_analyzer:==1.2.2
```

### Dependencies

Install the required Ansible collections by executing the following

```
ansible-galaxy collection install -r requirements.yml
```

## Contributing


## Support

Support tickets for RedHat Trusted Profile Analyzer can be opened at https://access.redhat.com/support/cases/#/case/new?product=Red%20Hat%20Trusted%20Profile%20Analyzer.

## Release notes and Roadmap

You can read the latest release notes [here](https://docs.redhat.com/en/documentation/red_hat_trusted_profile_analyzer/1.3/html/release_notes/index).

## Related Information

You can find more information about Red Hat Trusted Profile Analyzer [here](https://access.redhat.com/products/red-hat-trusted-profile-analyzer).

## Feedback

Any and all feedback is welcome.
Submit an [Issue](https://github.com/trustification/trustification-ansible/issues) or [Pull Request](https://github.com/trustification/trustification-ansible/pulls) as needed.

## License Information

You can find license information within the [LICENSE](https://github.com/trustification/trustification-ansible/blob/main/LICENSE) file.
