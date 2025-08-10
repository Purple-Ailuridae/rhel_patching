# RHEL Server Patching Workflow

This document provides a comprehensive overview of the RHEL server patching workflow, a set of Ansible playbooks designed to automate the patching process for Red Hat Enterprise Linux servers in a VMware environment.

## Overview

This automated workflow ensures that RHEL servers are securely patched while maintaining system stability and availability. It provides rollback capabilities for failed operations and comprehensive reporting for operational visibility. The workflow is designed to be robust, scalable, and suitable for enterprise environments.

## Architecture

The workflow consists of four main playbooks that operate in sequence:

1. **01-snapshot.yml** - Creates VMware snapshots for rollback capability
2. **02-patch.yml** - Applies security and bug fix patches
3. **03-reboot.yml** - Reboots servers to integrate patches
4. **04-healthchecks.yml** - Validates system health post-patching

## Prerequisites

Before implementing this workflow, ensure the following:

- Red Hat Ansible Automation Platform (AAP) is installed and configured
- Access to VMware vCenter for snapshot operations
- Appropriate permissions for vCenter operations
- Properly configured inventory with RHEL servers
- Required Ansible collections installed:
  - `vmware.vmware`
  - `community.vmware`
  - `ansible.builtin`

## Playbook Details

### 01-snapshot.yml - VMware Snapshot Creation

This playbook creates pre-patch snapshots for all targeted RHEL servers. Snapshots serve as rollback points for failed patching operations.

**Key Features:**
- Validates that servers are running RHEL 7, 8, or 9
- Confirms servers are running on VMware infrastructure
- Creates unique snapshot names with timestamps
- Handles snapshot creation failures gracefully
- Provides detailed reporting on success/failure rates

### 02-patch.yml - Package Updates and Patching

This playbook applies the latest patches to servers with successful snapshots.

**Key Features:**
- Detects RHEL version and selects appropriate package manager (yum/dnf)
- Refreshes repository metadata for fresh package information
- Applies latest security and bug fix patches
- Handles patch application failures gracefully
- Tracks package updates and failed packages

### 03-reboot.yml - System Reboot and Verification

This playbook reboots servers that successfully completed patching to ensure applied patches are properly integrated.

**Key Features:**
- Reboots servers with configurable timeout settings
- Verifies system responsiveness after reboot
- Checks critical service status (ssh, firewalld, etc.)
- Monitors reboot duration and performance metrics
- Handles reboot failures gracefully

### 04-healthchecks.yml - Health Check and Validation

This playbook performs comprehensive health checks on all servers to validate operational status.

**Key Features:**
- Validates system services (sshd, firewalld, sssd, fapolicyd for RHEL 8+)
- Checks resource utilization (CPU, memory, disk)
- Identifies critical and warning issues
- Classifies servers as healthy, needing fix, or unreachable
- Generates detailed health profiles for reporting

## Configuration Requirements

### Inventory Setup

The workflow requires a properly configured Ansible inventory with the following variables for each server:

```ini
[patchable_servers]
rhel7-server-01 ansible_host=192.168.1.10
rhel8-server-02 ansible_host=192.168.1.11
rhel9-server-03 ansible_host=192.168.1.12

[vcenter_credentials]
vcenter_hostname=vcenter.company.local
vcenter_username=admin@vsphere.local
vcenter_password=securepassword
vcenter_validate_certs=false
vm_datacenter=Datacenter1
vm_folder=/Datacenters/Datacenter1/VirtualMachines
```

### Required Variables

The following variables should be defined in your inventory or vault:

- `vcenter_hostname`: VMware vCenter server hostname
- `vcenter_username`: vCenter authentication username
- `vcenter_password`: vCenter authentication password
- `vcenter_validate_certs`: Certificate validation setting
- `vm_datacenter`: vCenter datacenter name
- `vm_folder`: Folder path for virtual machines

## Reporting

The workflow generates comprehensive reports in the `/tmp/ansible_reports` directory:
- Detailed HTML report with workflow statistics
- Executive summary in text format
- CSV file with servers requiring manual intervention (NEED_FIX)

## Implementation Guide

### Step 1: Prepare Environment

1. Install required Ansible collections:
```bash
ansible-galaxy collection install vmware.vmware
ansible-galaxy collection install community.vmware
```

2. Create the New_Playbook directory structure:
```bash
mkdir -p New_Playbook/template New_Playbook/vars New_Playbook/inventory
```

3. Copy all playbook files to the New_Playbook directory

### Step 2: Configure Inventory

Create an inventory file with your RHEL servers and vCenter credentials:
```ini
[patchable_servers]
server1.example.com ansible_host=192.168.1.10
server2.example.com ansible_host=192.168.1.11

[vcenter_credentials]
vcenter_hostname=vcenter.company.com
vcenter_username=admin@vsphere.local
vcenter_password=password
vcenter_validate_certs=false
vm_datacenter=Datacenter1
vm_folder=/Datacenters/Datacenter1/VirtualMachines
```

### Step 3: Configure Playbook Variables

Set up variables in your inventory or create a vars file:
```yaml
# vars/main.yml
vcenter_hostname: "vcenter.company.com"
vcenter_username: "admin@vsphere.local"
vcenter_password: "securepassword"
vcenter_validate_certs: false
vm_datacenter: "Datacenter1"
vm_folder: "/Datacenters/Datacenter1/VirtualMachines"
```

### Step 4: Run Playbooks

Execute the playbooks in sequence:
```bash
ansible-playbook 01-snapshot.yml
ansible-playbook 02-patch.yml
ansible-playbook 03-reboot.yml
ansible-playbook 04-healthchecks.yml
```

### Step 5: Review Reports

Check the generated reports in `/tmp/ansible_reports/` for detailed workflow statistics and server health information.

## Troubleshooting

### Common Issues and Solutions

1. **Connection Failures to vCenter:**
   - Verify vCenter hostname and credentials
   - Check network connectivity to vCenter server
   - Confirm SSL certificate validation settings

2. **Snapshot Creation Failures:**
   - Ensure VMware Tools are running on target servers
   - Check available disk space on datastore
   - Verify permissions for snapshot operations

3. **Package Update Failures:**
   - Confirm repository access and network connectivity
   - Check for dependency conflicts
   - Verify package manager configuration

4. **Reboot Issues:**
   - Ensure SSH connectivity is maintained during reboot
   - Check timeout settings for long-running reboots
   - Verify system service status after reboot

## Security Considerations

1. Store credentials in Ansible Vault for production environments
2. Implement proper RBAC for vCenter access
3. Regularly update and audit playbook permissions
4. Monitor and log all automation activities

## Maintenance and Updates

Regular maintenance of the workflow should include:
- Updating to latest Ansible collections
- Reviewing and updating security thresholds
- Monitoring for deprecated modules or parameters
- Regular testing of rollback procedures
