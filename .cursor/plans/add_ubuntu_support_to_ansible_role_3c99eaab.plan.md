---
name: Add Ubuntu Support to Ansible Role
overview: Make the ansible-role-prepare-k8s role compatible with both RHEL and Ubuntu by adding OS detection and conditional logic for package management, firewall configuration, service management, and config file paths.
todos: []
---

# Add Ubuntu Support t

o Ansible RoleThis role is currently RHEL-specific and needs modifications to support Ubuntu. The main changes involve package management, firewall configuration, service management, and config file paths.

## Analysis

The role currently uses RHEL-specific components:

- Package manager: `yum` module
- Package names: `iptables-services`, `ipset-service`, `glibc-langpack-ru`
- Config paths: `/etc/sysconfig/iptables`, `/etc/sysconfig/ipset`
- Services: `iptables`, `ipset` systemd services
- Firewall: `firewalld` removal

## Implementation Plan

### 1. Add OS-specific variables to `defaults/main.yml`

- Define package lists for RedHat and Debian families
- Define config file paths for each OS family
- Define service names for each OS family

### 2. Update package management in `tasks/04_configuring_servers_for_use.yml`

- Replace `yum` module with `ansible.builtin.package` (works cross-platform) OR use conditional `yum`/`apt` tasks
- Add conditional package lists:
    - RHEL: `iptables-services`, `ipset-service`, `glibc-langpack-ru`
    - Ubuntu: `iptables-persistent`, `ipset`, `language-pack-ru-base`
- Handle firewall removal: `firewalld` on RHEL, `ufw` on Ubuntu (if present)

### 3. Update config file paths

- RHEL: `/etc/sysconfig/iptables`, `/etc/sysconfig/ipset`
- Ubuntu: `/etc/iptables/rules.v4`, `/etc/ipset.conf` (or keep same path if using iptables-persistent restore format)
- Use variables to set paths based on OS family
- Update template destinations accordingly

### 4. Update service management

- RHEL: Enable `iptables` and `ipset` services
- Ubuntu: Enable `netfilter-persistent` service (handles iptables persistence)
- ipset on Ubuntu may not need a separate service

### 5. Update iptables/ipset restore commands

- Adjust paths in restore commands based on OS family
- On Ubuntu, may need to use `iptables-restore` with proper path format

### 6. Update meta/main.yml

- Add supported platforms: RedHat, Debian, Ubuntu

## Key Files to Modify

- `prepare-k8s/defaults/main.yml` - Add OS-specific variables
- `prepare-k8s/tasks/04_configuring_servers_for_use.yml` - Main changes for OS compatibility
- `prepare-k8s/meta/main.yml` - Update platform support

## Notes

- The template files (`iptables.j2`, `ipset.j2`) should work for both OSes as they contain rule definitions, not OS-specific syntax