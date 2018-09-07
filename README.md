# ansible-hitch

[![Build Status](https://api.travis-ci.org/ThomasGsp/ansible-hitch.svg?branch=master)](https://travis-ci.org/ThomasGsp/ansible-hitch) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-ThomasGsp.hitch-blue.svg?style=flat)](https://galaxy.ansible.com/detail#/role/xxx)

 - Ansible 2.2+ (1.x not supported)
 - Compatible with most versions of Ubuntu/Debian
 - Compatibility with RHEL/CentOS 6.x will come soon

## Contents

 1. [Installation](#installation)
 2. [Getting Started](#getting-started)
  1. [Hitch TLS - With single Varnish backend](#single-hitch-varnish)
 3. [Advanced Options](#advanced-options)
  1. [Verifying checksums](#verifying-checksums)
  2. [Install from local tarball](#install-from-local-tarball)
  3. [Building 32-bit binaries](#building-32-bit-binaries)
 4. [Role Variables](#role-variables)

## Installation

``` bash
$ ansible-galaxy install thomasgsp.hitch
```

## Getting started

Below are a few example playbooks and configurations for deploying a variety of hitch architectures.

This role expects to be run as root or as a user with sudo privileges.

### Hitch TLS - With single Varnish backend

Deploying a single Hitch server node is pretty trivial; just add the role to your playbook and go.

``` yml
---
- hosts: hitch01.example.com
  vars:
    frontend:
      - bind: "*:443"
        pem_file:
          - "/etc/hitch/certs/default.pem"

      # Secondary bind
      #- bind: "*:5443"
      #  pem_file:
      #    - "/etc/hitch/certs/default.pem"

    backend: "[127.0.0.1]:6081"

  roles:
    - Thomasgsp.hitch
```

``` bash
$ ansible-playbook -i hitch01.example.com hitch.yml
```

**Note:** You may have noticed above that I just passed a hostname in as the Ansible inventory file. This is an easy way to run Ansible without first having to create an inventory file, you just need to suffix the hostname with a comma so Ansible knows what to do with it.

That's it! You'll now have a Hitch server listening on 127.0.0.1 https on hitch01.example.com. By default, the Hitch binaries are installed under /opt/hitch, though this can be overridden by setting the `install_dir` variable.


## Advanced Options

### Verifying checksums

Set the `verify_checksum` variable to true to use the checksum verification option for `get_url`.
Note that this will only verify checksums when Hitch is downloaded from a URL, not when one is provided in a tarball with `tarball`.
Due to differences in the `get_url` module in Ansible 1.x and Ansible 2.x, this feature behaves differently depending on the version of Ansible which you are using.

#### Ansible 1.x

In Ansible 1.x, the `get_url` module only supports verifying sha256 checksums, which are not provided by default.
If you wish to set `verify_checksum`, you must also define a sha256 checksum with the `checksum` variable.

``` yaml
- name: install hitch on ansible 1.x and verify checksums
  hosts: all
  roles:
    - role: Thomasgsp.hitch
      version: 1.4.8
      verify_checksum: true
      checksum: d52ba690d90c25bbfca73f5e0ed427738366dac12faf46fb5834e497cc2d1ac3
```

#### Ansible 2.x

When using Ansible 2.x, this role will verify the sha1 checksum of the download against checksums defined in the `checksums` variable in `vars/main.yml`.
If your version is not defined in here or you wish to override the checksum with one of your own, simply set the `checksum` variable. As in the example below, you will need to prefix the checksum with the type of hash which you are using.

``` yaml
- name: install hitch on ansible 1.x and verify checksums
  hosts: all
  roles:
    - role: Thomasgsp.hitch
      version: 1.4.8
      verify_checksum: true
      checksum: "sha256:d52ba690d90c25bbfca73f5e0ed427738366dac12faf46fb5834e497cc2d1ac3"
```

### Install Hitch from local tarball

If the environment your server resides in does not allow downloads (i.e. if the machine is sitting in a dmz) set the variable `tarball` to the path of a locally downloaded Hitch tarball to use instead of downloading over HTTP from hitch-tls.org.

Do not forget to set the version variable to the same version of the tarball to avoid confusion! For example:

```yml
vars:
  version: 1.4.8
  tarball: /path/to/hitch-1.4.8.tar.gz
```

In this case the source archive is copied to the server over SSH rather than downloaded.

### Building 32 bit binaries

To build 32-bit binaries of Hitch (which can be used for, set `make_32bit: true`.
This installs the necessary dependencies (x86 glibc) on RHEL/Debian and sets the option '32bit' when running make.

## Role Variables

Here is a list of all the default variables for this role, which are also available in defaults/main.yml. One of these days I'll format these into a table or something.

``` yml
---
## Installation options
version         : 1.4.8
install_dir     : "/opt/hitch"
download_url    : "https://hitch-tls.org/source/hitch-1.4.8.tar.gz"
dir             : "/etc/hitch/"
# Set this to true to build 32-bit binaries
make_32bit      : false
# Set this value to a local path of a tarball to use for installation instead of downloading
tarball         : false
# Set this to true to validate hitch tarball checksum against vars/main.ym
verify_checksum : false

## Role options
# Configure Hitch as a service
as_service: true
pidfile: /var/run/hitch/hitch.pid
logfile: /var/log/hitch.log
# Add local facts to /etc/ansible/facts.d for hitch
local_facts: true
# Service name
service_name: "hitch"

## front end option
# Hitch accept several frontend definition
frontend:
  - bind: "*:443"
  #- bind: "*:4443"

## back end option
# Default Varnish backend (IP:PORT)
# Can also take a UNIX domain socket path E.g. backend="/path/to/sock"
# brackets are mandatory .
backend         : "[127.0.0.1]:6081"
#TCP keepalive on client socket
keepalive       : 3600

# Periodic backend IP lookup, 0 to disable
backend_refresh : 0

## ocsp options
# Set OCSP staple cache directory This enables automated retrieval and stapling of OCSP responses
ocsp_dir          : "/var/lib/hitch/"
ocsp_connect_tmo  :
ocsp_resp_tmo     :
ocsp_refresh_interval :
ocsp_verify_staple    :

# Pem cert list
pem_file        :
  -  "/etc/hitch/certs/default.pem"

private_key     :
prefer_server_ciphers :
proxy_proxy     : off

# Sets the protocols for ALPN/NPN negotiation, given by a comma separated list.
# If this is not set explicitly, ALPN/NPN will not be used. Requires OpenSSL 1.0.1 for NPN and OpenSSL 1.0.2 for ALPN.
alpn_protos     : "h2,http/1.1"
# All TLS versions, no SSLv3 (deprecated)
tls_protos      : "TLSv1.0 TLSv1.1 TLSv1.2 SSLv3"
# Sets allowed ciphers
ciphers         : "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH"
# Abort handshake when client submits an unrecognized SNI server name
sni_nomatch_abort : off
# Sets OpenSSL engine
ssl_engine      :

# Number of worker processes.
workers         : 1

## Daemon options
# Be quiet; emit only error messages
quiet           : on
# Send log message to syslog in addition to stderr/stdout
syslog          : on
# Syslog facility to use
syslog_facility : daemon
# Set listen backlog size
backlog         : 100
# Set uid/gid after binding the socket
user            : _hitch
# Set gid after binding the socket
group           : _hitch
# Fork into background and become a daemon; this also sets the --quiet option
daemon          : on
# Sets chroot directory
chroot          : /etc/hitch
# Write 1 octet with the IP family followed by the IP address in 4 (IPv4) or 16 (IPv6) octets
# little-endian to backend before the actual data
write_ip        : off
# Write HaProxy's PROXY v1 (IPv4 or IPv6) protocol line before actual data
write_proxy_v1  : off
# Write HaProxy's PROXY v2 binary (IPv4 or IPv6) protocol line before actual data
write_proxy_v2  : on
```

## Facts

The following facts are accessible in your inventory or tasks outside of this role.

- `{{ ansible_local.hitch.frontend }}`
- `{{ ansible_local.hitch.backend }}`
- `{{ ansible_local.hitch.pem-file }}`


To disable these facts, set `local_facts` to a false value.
