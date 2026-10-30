[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)
[![Dev Spaces](https://img.shields.io/badge/Customize%20Here-0078d7.svg?style=for-the-badge&logo=visual-studio-code&logoColor=white)](https://workspaces.openshift.com/f?url=https://github.com/ansible/product-demos)

# APD - Ansible Product Demos

The Ansible Product Demos (APD) project is a set of Ansible demos that run on the [Red Hat Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible) (AAP).  These demos are deployed using configuraton-as-code and playbooks that create AAP resources (such as projects, templates, and credentials) meant for demonstrating automation use cases in several technology domains.

> **This branch targets AAP 2.4** (automation controller 4.4 or 4.5). Upstream `main` targets AAP 2.5 and later and talks to the platform gateway. Install with `install-apd.yml` against the controller API, using an execution environment built from `execution_environments/apd-ee-24.yml`.

| Demo Name | Description |
|-----------|-------------|
| [Linux](linux/README.md) | Repository of demos for RHEL and Linux automation |
| [Windows](windows/README.md) | Repository of demos for Windows Server automation |
| [Cloud](cloud/README.md) | Demo for infrastructure and cloud provisioning automation |
| [Network](network/README.md) | Network automation demos |
| [OpenShift](openshift/README.md) | OpenShift automation demos |
| [Infrastructure](infrastructure/README.md) | Infrastructure provisioning demos (ROSA cluster lifecycle, etc.) |
| [Satellite](satellite/README.md) | Demos of automation with Red Hat Satellite Server |

## Demo catalog

Browse all demos, presenter guides, and walkthroughs on the **[Ansible Product Demos — GitHub Pages site](https://ansible.github.io/product-demos/)**.

## Installation

APD can be added to an existing AAP 2.4 deployment by running the installation playbook. The playbook talks to the automation controller at `/api/v2/` and applies configuration with `infra.controller_configuration` 2.11. It does not use the platform gateway.

Build the AAP 2.4 execution environment before installing. `quay.io/ansible-product-demos/apd-ee-24` is not published upstream. The controller also pulls that image when it syncs the Ansible Product Demos project, so the image must be in a registry the controller can reach. Override the image with `APD_EE_IMAGE` if you push it somewhere other than the default name.

### Build the execution environment

From `execution_environments/`, after `podman login registry.redhat.io` and exporting Automation Hub tokens (`ANSIBLE_GALAXY_SERVER_CERTIFIED_TOKEN` and `ANSIBLE_GALAXY_SERVER_VALIDATED_TOKEN`):

```bash
./build.sh quay.io/<your-registry>/apd-ee-24
```

The definition expects the current `ansible-automation-platform-24/ee-minimal-rhel9` image (ansible-core 2.16, Python 3.12). Tag and push the result, then export that reference:

```bash
export APD_EE_IMAGE=quay.io/<your-registry>/apd-ee-24:latest
```

### Publish this branch

The controller project clones `APD_GIT_REPO` at `APD_GIT_REPO_BRANCH` (default `aap-2.4`). That branch is not on `github.com/ansible/product-demos`. Push this branch to a git remote the controller can read, then:

```bash
export APD_GIT_REPO=https://github.com/<you>/aap-product-demos
export APD_GIT_REPO_BRANCH=aap-2.4
```

`install-apd.yml` stops if those still point at the upstream repository and the `aap-2.4` branch.

### Installing with the installation playbook

1. Set authentication for the automation controller. `AAP_HOSTNAME` is the controller URL, not a gateway URL.

```
export AAP_HOSTNAME=https://your-aap-controller.example.com

# either set AAP_USERNAME and AAP_PASSWORD for password auth
export AAP_USERNAME=admin  # or another controller account with superuser privileges
export AAP_PASSWORD=<admin_user_password>

# or alternately set AAP_TOKEN if you have an admin token
#export AAP_TOKEN=<admin_user_token>
```

2. Use `ansible-navigator` to run the installation playbook. ansible-navigator must be installed, either from the AAP package repository or from the [upstream ansible-dev-tools PyPI package](https://pypi.org/project/ansible-dev-tools/). `ansible-navigator.yml` selects `apd-ee-24`. Point `--eei` at the image you built if it is not the default quay name.

```
ansible-navigator run -m stdout install-apd.yml --eei "$APD_EE_IMAGE"
```

If that image exists only on the machine running ansible-navigator, add `--pull-policy never` so navigator does not try to pull it. The controller still needs a copy it can pull for project sync and jobs.

#### Quick-start one-liner (no environment variables needed)

```bash
ansible-navigator run install-apd.yml \
  --eei quay.io/<your-registry>/apd-ee-24:latest \
  -e aap_hostname=https://your-aap-controller.example.com \
  -e aap_username=admin \
  -e aap_password='your-password' \
  -e '{"aap_validate_certs": false}' \
  -e apd_git_repo=https://github.com/<you>/aap-product-demos \
  -e apd_git_repo_branch=aap-2.4 \
  -e apd_ee_image=quay.io/<your-registry>/apd-ee-24:latest \
  -m stdout
```

> [!IMPORTANT]
> If your controller uses a self-signed or untrusted TLS certificate, `aap_validate_certs` must be the boolean `false` (not the string `"false"`). Use the JSON syntax shown above: `-e '{"aap_validate_certs": false}'`

### What still expects AAP 2.5 or later

These jobs are created with the rest of the catalog, and they still call gateway-era interfaces:

- **Config drift EDA activation** (`infrastructure/config-drift/playbooks/setup_eda_activation.yml`) uses `infra.aap_configuration` EDA roles, an AAP 2.7 decision-environment image, and an `/api/controller` credential host.
- **Platform gateway objects** (authenticators, gateway role definitions) are not created. Organization admin for `apd-admin` is a controller RBAC grant.

Controller API calls in the Mattermost and Automation Orchestrator tasks use `/api/v2/` on this branch.

### Use a pre-installed APD environment on the Red Hat Demo Platform (account required)

For Red Hat associates and partners, there is an Ansible Product Demos catalog item [available on demo.redhat.com](https://red.ht/apd-sandbox) that provides a pre-installed environment for demo purposes.  An existing account is required for access to the Red Hat Demo Platform system.

## Bring Your Own Demo

Can't find what you're looking for? Customize this repo to make it your own.

1. Create a fork of this repo.
2. Update the URL of the `Ansible Project Demos` project your Ansible Automation Platform controller.
3. Make changes to your fork as needed and run the **Product Demos | Single demo setup** job

See the [contributing guide](CONTRIBUTING.md) for more details on how to customize the project.

---

[Privacy statement](https://www.redhat.com/en/about/privacy-policy) | [Terms of use](https://www.redhat.com/en/about/terms-use) | [Security disclosure](https://www.ansible.com/security?hsLang=en-us) | [All policies and guidelines](https://www.redhat.com/en/about/all-policies-guidelines)
