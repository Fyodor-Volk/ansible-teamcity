# Ansible TeamCity

Ansible project for deploying a TeamCity CI/CD environment.

## Structure

- `inventories/prod/hosts.yml` - production inventory example.
- `inventories/prod/group_vars/` - common and group-specific variables.
- `playbooks/bootstrap.yml` - base host preparation.
- `playbooks/teamcity_server.yml` - TeamCity server deployment.
- `playbooks/teamcity_agents.yml` - TeamCity agent deployment.
- `playbooks/site.yml` - full deployment entrypoint.
- `roles/common` - package updates, base packages and Docker installation.
- `roles/teamcity_server` - Docker Compose based TeamCity server stack.
- `roles/teamcity_agent` - OS-level TeamCity build agent installation.

## Quick Start

Install required collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

Run full deployment:

```bash
ansible-playbook playbooks/site.yml
```

Run only server:

```bash
ansible-playbook playbooks/teamcity_server.yml
```

Run only agents:

```bash
ansible-playbook playbooks/teamcity_agents.yml
```

## Important Variables

- `teamcity_server_compose_dir` - directory for `docker-compose.yml` and persistent data.
- `teamcity_server_url` - URL used by agents to download and connect to TeamCity.
- `teamcity_version` - TeamCity Docker image tag.
- `teamcity_agent_user` - OS user used to run the agent.
- `teamcity_agent_java_package_debian` - Java package for Ubuntu/Debian.
- `teamcity_agent_java_package_redhat` - Java package for Alma/RHEL.

Replace placeholder hosts, users and passwords before running in production.
