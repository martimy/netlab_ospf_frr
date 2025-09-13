# NetLab Tutorial: Simple OSPF Lab with FRR

This tutorial guides you through installing [netlab](https://netlab.tools/), a network simulation tool for automating virtual network labs. We'll use it to deploy a simple OSPF lab with three FRR (Free Range Routing) routers. The lab demonstrates basic OSPF routing between three routers with stub networks.

NetworkLab is a platform for creating and managing virtual network environments, using [Containerlab](https://containerlab.dev/) with Docker to quickly deploy and test network topologies. With Ansible integration, users automate device provisioning and configuration, ensuring efficient, repeatable setups for simulation, education, and network design.

It is highly recommended that you become familiar with clab, Docker, and Ansible to gain the full benefits of netlab and also to be able to troubleshoot any problems that may arise while creating netlab environments.

![Network Topology](topology.png)

## Prerequisites

- **OS**: Ubuntu 22.04 LTS (tested in a Vagrant box; works on similar Debian-based systems).
- **Hardware**: At least 4GB RAM and 2 CPU cores (for the lab containers).
- **Git**: For cloning repos (optional, but recommended):

## Step 1: Install NetLab

NetLab is a Python-based tool. Install Python 3 and pip first, then netlab.

1. Update packages and install Python 3 pip (this pulls in build essentials and dependencies like GCC):
   ```
   sudo apt update
   sudo apt install python3-pip -y
   pip install networklab
   netlab version
   netlab version 25.07
   ```

2. Install the required tools such as containerlab and ansible. You can install them separately, but Netlab makes it easier to do it directly through the netlab command.

    ```
    $ netlab install
    ┏━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
    ┃ Script       ┃ Installs                                          ┃
    ┡━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
    │ ubuntu       │ Mandatory and nice-to have Debian/Ubuntu packages │
    │ libvirt      │ QEMU, KVM, libvirt, and Vagrant                   │
    │ containerlab │ Docker and containerlab                           │
    │ ansible      │ Ansible and prerequisite Python libraries         │
    │ grpc         │ GRPC libraries and Nokia GRPC Ansible collection  │
    └──────────────┴───────────────────────────────────────────────────┘
    ```

    Install each tool and verify the installation before proceeding.

    ```
    netlab install ubuntu
    ```

    ```
    netlab install ansible
    ansible --version
    ```

    ```
    netlab install containerlab
    clab version
    ...
    ```

## Step 2: Prepare the Lab Topology

Create a directory for your lab and define the topology in YAML.

1. Create and navigate to the lab directory:
   ```
   mkdir ~/nlab_ospf_frr
   cd ~/nlab_ospf_frr
   ```

2. Create `topology.yml` that defines the network topology with the following content (this defines two FRR routers with OSPF):
   ```yaml
    ---
    provider: clab
    defaults.device: frr
    module: [ ospf ]

    nodes: [ r1, r2 ]
    links: [ r1, r2, r1-r2 ]
   ```

## Step 3: Deploy the Lab

NetLab generates configs, deploys containers, and applies initial OSPF configs.

1. Start the lab:

   To start the lab, execute the command:

   ```
   netlab up
   ```

    The command **netlab up** creates *clab.yml* (containerlab configuration file), and Ansible inventory and variables files, starts the devices with **containerlab deploy** command, and configures the devices with **netlab initial** and **netlab create** commands.

2. Verify the network deployment:

    ```
    sudo clab inspect -t clab.yml  # List running containers and IPs
    ```

## Step 4: Interact with the devices

Once devices are running, connect to devices and verify OSPF.

1. Connect to `r1`:
   ```
   netlab connect r1
   ```
   
   This drops you into bash on the container. Use `vtysh` for FRR CLI.

2. In the container (vtysh mode):
   ```
   r1(bash)# vtysh

   Hello, this is FRRouting (version 10.4.1_git).
   Copyright 1996-2005 Kunihiro Ishiguro, et al.

   r1# show interface brief
   Interface       Status  VRF             Addresses
   ---------       ------  ---             ---------
   eth1            up      default         172.16.0.1/24
   eth2            up      default         10.1.0.1/30
   eth3            up      default         10.1.0.9/30
   lo              up      default         10.0.0.1/32

   Interface       Status  VRF             Addresses
   ---------       ------  ---             ---------
   eth0            up      mgmt            192.168.121.101/24
   mgmt            up      mgmt

   r1# exit
   r1(bash)# exit
   logout
   ```

## Step 5: Cleanup

Stop and remove the lab.

1. Stop the lab (keeps files):
   ```
   netlab down
   ```

2. Full cleanup (removes containers, networks, and generated files):
   ```
   netlab down --cleanup
   ```
   
   This removes `clab.yml`, `clab_files/`, `hosts.yml`, etc. Only `topology.yml` remains.

## Verification and Reporting

The **netlab status** command displays the status and workload (VMs or containers) of the current or selected lab instance.

```
$ netlab status

lab default in /home/vagrant/nlab_ospf_frr
  status: started
  provider(s): clab

┏━━━━━━┳━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓
┃ node ┃ device ┃ image                    ┃ mgmt IPv4       ┃ connection ┃ provider ┃ VM/container        ┃ status        ┃
┡━━━━━━╇━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━╇━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩
│ r1   │ frr    │ quay.io/frrouting/frr:1… │ 192.168.121.101 │ docker     │ clab     │ clab-nlabospffrr-r1 │ Up 16 minutes │
├──────┼────────┼──────────────────────────┼─────────────────┼────────────┼──────────┼─────────────────────┼───────────────┤
│ r2   │ frr    │ quay.io/frrouting/frr:1… │ 192.168.121.102 │ docker     │ clab     │ clab-nlabospffrr-r2 │ Up 16 minutes │
├──────┼────────┼──────────────────────────┼─────────────────┼────────────┼──────────┼─────────────────────┼───────────────┤
│ r3   │ frr    │ quay.io/frrouting/frr:1… │ 192.168.121.103 │ docker     │ clab     │ clab-nlabospffrr-r3 │ Up 16 minutes │
└──────┴────────┴──────────────────────────┴─────────────────┴────────────┴──────────┴─────────────────────┴───────────────┘
```

The **netlab report** command creates one of available system reports.

```
netlab report ospf-areas

OSPF Area 0.0.0.0

|-----------------|-----------------|-----------------|----------------------|
| Router          | Interface       |    IPv4 Address | Neighbor(s)          |
|-----------------|-----------------|-----------------|----------------------|
| r1              | Loopback        |     10.0.0.1/32 |                      |
|                 | eth2            |     10.1.0.1/30 | r2                   |
|                 | eth3            |     10.1.0.9/30 | r3                   |
|-----------------|-----------------|-----------------|----------------------|
| r2              | Loopback        |     10.0.0.2/32 |                      |
|                 | eth2            |     10.1.0.2/30 | r1                   |
|                 | eth3            |     10.1.0.5/30 | r3                   |
|-----------------|-----------------|-----------------|----------------------|
| r3              | Loopback        |     10.0.0.3/32 |                      |
|                 | eth2            |     10.1.0.6/30 | r2                   |
|                 | eth3            |    10.1.0.10/30 | r1                   |
|-----------------|-----------------|-----------------|----------------------|

OSPF Area 0.0.0.1

|-----------------|-----------------|-----------------|----------------------|
| Router          | Interface       |    IPv4 Address | Neighbor(s)          |
|-----------------|-----------------|-----------------|----------------------|
| r1              | eth1            |   172.16.0.1/24 | passive              |
|-----------------|-----------------|-----------------|----------------------|

OSPF Area 0.0.0.2

|-----------------|-----------------|-----------------|----------------------|
| Router          | Interface       |    IPv4 Address | Neighbor(s)          |
|-----------------|-----------------|-----------------|----------------------|
| r2              | eth1            |   172.16.1.2/24 | passive              |
|-----------------|-----------------|-----------------|----------------------|

OSPF Area 0.0.0.3

|-----------------|-----------------|-----------------|----------------------|
| Router          | Interface       |    IPv4 Address | Neighbor(s)          |
|-----------------|-----------------|-----------------|----------------------|
| r3              | eth1            |   172.16.2.3/24 | passive              |
|-----------------|-----------------|-----------------|----------------------|
```

# References

- https://www.packetswitch.co.uk/netlab-the-fastest-way-to-build-network-labs/
