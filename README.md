# `r_o3c_wizsensor`

This Ansible role manages the deployment and configuration of the Wiz Sensor. It performs tasks such as fetching API keys, creating service accounts, configuring environment files, and installing the Wiz Sensor on Debian-based and Red Hat-based systems. It also installs a SELinux Policy if SELinux is enforced, and does some cleanup of temporary files after it's done with the install.

---
## Features of the Playbook

* **Checks if auth and tenant endpoints are reachable.**
* **Checks if the system is Debian or RedHat.**
* **Checks if the Wiz Sensor systemd service is running.**  
  *\<Playbook stops if one of the endpoints does not respond or the systemd service is running\>*
* **Compiles and installs SELinux policy if SELinux is enforced.**
* **Uses the Wiz GraphQL API to create a Service Account for the sensor** with the name `sensor_sa_$HOSTNAME`.
* **Installs the Wiz sensor using the official install script** (supports both RedHat and Ubuntu systems).
* **Deletes temporary files and clears environment variables** (including sensitive data like API_SECRET).

## Requirements

- Python must be installed on the target host.
- The target system must be either Debian-based (Ubuntu, Debian) or Red Hat-based (RHEL, CentOS, Rocky, AlmaLinux).
- The service account running the tasks need sudo access.
- Internet access is required for external repositories (unless using internal repositories, see variables).
- The following packages are required:
  - `curl` (for GraphQL API requests).
- Optional SELinux dependencies:
  - `policycoreutils`
  - `policycoreutils-python-utils`
  - `selinux-policy-devel`
---

## Role Variables

### Service Account and API Communication
These variables are used to authenticate and communicate with Wiz's GraphQL API:

- `wiz_api_client_id`: **(Required)** Client ID for the service account with permissions to create other service accounts.
- `wiz_api_client_secret`: **(Required)** Client Secret associated with the `wiz_api_client_id`.
- `wiz_api_bearer_token`: Token retrieved from the Wiz authentication API. This is dynamically generated by the role and stored in memory.
- `wiz_api_auth_url`: Default: `"https://auth.app.wiz.io/oauth/token"`. URL for obtaining the bearer token.
- `wiz_api_endpoint_url`: Default: `"https://api.eu14.app.wiz.io/graphql"`. GraphQL endpoint for Wiz API operations. Check under "Tenant info" in the Wiz Portal.

### Service Account Variables
- `sensor_sa_name`: Default: `"sensor-sa-{{ ansible_hostname }}"`. Name of the sensor service account that will be created dynamically.

### Local repositories and other settings
- `http_proxy`: Default: `false`. Set to `true` if you need a HTTP Proxy to reach www
- `sensor_proxy_url`: Default: `blank`. Combine with `http_proxy` to set the Proxy URL. E.g: `http://user:pass@x.x.x.x:yyyy`
- `skip_repo`: Default: `false`. Checks if wiz-sensor rpm package is available and passes `SKIP_ADD_REPOSITORY` to the install script.
---

## Dependencies

This role does not have any external dependencies on other Ansible Galaxy roles.

---

## Example Playbook

Here is an example of how to use the role in your playbook:

```yaml
- name: Deploy Wiz Sensor
  hosts: all
  vars:
    wiz_api_client_id: "your_client_id"
    wiz_api_client_secret: "your_client_secret"
  roles:
    - r_o3c_wizsensor
