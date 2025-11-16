# Ansible Role: OpenSSH

[![CI](https://github.com/pluggero/ansible-role-openssh/actions/workflows/ci.yml/badge.svg)](https://github.com/pluggero/ansible-role-openssh/actions/workflows/ci.yml) [![Ansible Galaxy downloads](https://img.shields.io/ansible/role/d/pluggero/openssh?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/pluggero/openssh)

An Ansible Role that installs and configures OpenSSH server and client with secure defaults.

## Requirements

None.

## Role Variables

### Installation Variables

```yaml
openssh_install_method: "package"
```

The method used to install OpenSSH can be defined in the variable `openssh_install_method`.
The following methods are available:

- `package`: Installs OpenSSH from the package manager of the distribution
  - **NOTE**: This method installs the latest version available in the package manager and not the version defined in `openssh_version`.

### OpenSSH Server Configuration

```yaml
openssh_server:
  port: 22 # OpenSSH server port
  listen_address: ["0.0.0.0", "::"] # IPv4 and IPv6 listen addresses
  permit_root_login: false # Permit root login (boolean)
  password_authentication: false # Enable password authentication (boolean)
  pubkey_authentication: true # Enable public key authentication (boolean)
  authentication_methods: "publickey" # Authentication methods
  kex: # Key exchange algorithms
    - "curve25519-sha256"
    - "sntrup761x25519-sha512@openssh.com"
  ciphers: # Encryption ciphers
    - "chacha20-poly1305@openssh.com"
    - "aes256-gcm@openssh.com"
  macs: # MAC algorithms
    - "hmac-sha2-512-etm@openssh.com"
    - "hmac-sha2-256-etm@openssh.com"
```

### OpenSSH Client Configuration (System-wide)

```yaml
openssh_client_system:
  send_env: ["LANG", "LC_*"] # Environment variables to send
  forward_agent: false # Enable agent forwarding (boolean)
```

### OpenSSH Client Configuration (Per-User)

```yaml
openssh_client_users:
  - user: "alice" # Username
    config:
      - host: "git.example.com" # Host pattern
        hostname: "git.example.com" # Actual hostname
        identity_file: "~/.ssh/id_ed25519_git" # OpenSSH key path
        user: "git" # Remote username
        forward_agent: false # Agent forwarding (boolean)
      - host: "prod-*" # Host pattern with wildcard
        proxy_jump: "bastion.example.com" # Jump host
        identity_file: "~/.ssh/id_ed25519_prod" # OpenSSH key path
        strict_host_key_checking: true # Strict host key checking (boolean)
        user_known_hosts_file: "~/.ssh/known_hosts_prod" # Custom known_hosts file
```

## Example Playbook

### Basic Usage

```yaml
- hosts: all
  roles:
    - pluggero.openssh
```

### Custom OpenSSH Server Configuration

```yaml
- hosts: servers
  roles:
    - pluggero.openssh
  vars:
    openssh_server:
      port: 2222
      permit_root_login: false
      password_authentication: false
```

### Per-User OpenSSH Client Configuration

```yaml
- hosts: workstations
  roles:
    - pluggero.openssh
  vars:
    openssh_client_users:
      - user: "alice"
        config:
          - host: "github.com"
            hostname: "github.com"
            identity_file: "~/.ssh/id_ed25519_github"
            user: "git"
      - user: "bob"
        config:
          - host: "*.production.example.com"
            proxy_jump: "bastion.example.com"
            strict_host_key_checking: true
```

## Security Best Practices

This role implements security-focused defaults:

- **No root login**: `permit_root_login: false`
- **Public key authentication only**: `password_authentication: false`
- **Modern cryptography**: Uses secure key exchange algorithms, ciphers, and MACs
- **Configuration validation**: Server config is validated before applying
- **Automatic backups**: Original configs are backed up before modification

## Dependencies

None.

## License

MIT / BSD

## Author Information

This role was created in 2025 by Robin Plugge.
