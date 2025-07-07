# Ansible-BGP-Automation

## Usage

1. **Configure variables**:
    + `inventory/hosts.ini`: list your AS containers under `[transit_AS]`

    + `group_vars/all.yml`/`roles/frr-config/defaults/main.yml`: adjust paths, SSH options, user

    + Per-AS `routers` list: define within `host_vars` 

2. **Validate connectivity to the router**:
    ```
    ssh root@<router_ip> vtysh -c 'show run'
    ```

3. **Run the playbook**: 
    ```
    ansible-playbook playbook.yml
    ``` 
## Overview

This project automates the configuration of FRRouting (FRR) on a Virtual Mini-Internet topology, consisting of multiple Autonomous Systems (AS) and routers; it is used for the inter-domain routing course labs in the University of Strasbourg. Using ansible it:

### Key Features

+ Automated FRR Provisioning: Generate and deploy router-specific FRR configs

+ Dynamic Templating: Jinja2 templates parameterize interfaces, OSPF, BGP, and policy constructs

+ SSH Chaining: Nested SSH from control machine → AS container → router namespace

## Prerequisites

SSH key-based access to AS containers (`ansible_user: root`)

## How It Works

1. **Loop Over Routers**: `roles/frr-config/tasks/main.yml` iterates `routers` for each AS host.

2. **Render Config**: `router.yml` uses the template lookup to create `/tmp/<router>.conf` inside the AS container.

3. **Copy & Apply**: `scp` or piped SSH sends the file to the router and executes `vtysh -f /tmp/<router>.conf`.


## Template Details

+ **Interfaces**: loop through `router.interfaces`; adds IP and description

+ **OSPF**: `router.ospf_interfaces` in area 0

+ **Prefix & Community Lists**: from `router.prefix_lists` and `router.community_lists`

+ **Route-Maps**: Gao–Rexford policy entries in `router.route_maps`

+ **BGP**:

    + eBGP neighbors (router.ebgp_neighbors)

    + iBGP neighbors (router.ibgp_neighbors), optional RR-client

    + IPv4 unicast AF: router.networks, neighbor activations, route-map attachments, next-hop-self