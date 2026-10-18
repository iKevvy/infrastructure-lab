# Project 001 - Automated Linux Deployment

## Overview
Automated the configuration of a Debian 13 server using Ansible.

## Environment
- Proxmox VE with unprivileged LXC containers
- CT104: Ansible controller
- CT105: Managed Debian server
- Ansible Core 2.19
- SSH with ED25519 authentication
- Nginx web server

## Implementation
1. Provisioned two Debian containers.
2. Configured SSH key authentication.
3. Created an Ansible inventory and playbook.
4. Automated Nginx installation and webpage deployment.
5. Verified service availability over HTTP.

## Validation
| Test | Result |
| --- | --- |
| Ansible connectivity | Passed |
| HTTP response | Passed |
| Idempotency | changed=0 |
| Configuration recovery | changed=2 |
| Final verification | changed=0 |

## Recovery Test
Stopped Nginx and deleted the managed webpage on CT105.

Re-running Ansible restored both resources without manual repair.

## Usage
Copy inventory.example.ini to inventory.ini and configure the target.

Run the following commands from this directory:

    ansible -i inventory.ini linux_servers -m ping
    ansible-playbook -i inventory.ini deploy.yml --ask-become-pass

## Lessons Learned
- Configuration management and desired state
- SSH key authentication and SSH agents
- Ansible idempotency
- Detection and recovery of configuration drift

## Limitations
- Single managed node
- DHCP-dependent addressing
- Interactive privilege escalation
- Manual recovery execution
