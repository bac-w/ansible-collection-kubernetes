# Ansible Collection for Kubernetes

[![Travis](https://img.shields.io/travis/com/alvistack/ansible-collection-kubernetes.svg)](https://travis-ci.com/alvistack/ansible-collection-kubernetes)
[![GitHub release](https://img.shields.io/github/release/alvistack/ansible-collection-kubernetes.svg)](https://github.com/alvistack/ansible-collection-kubernetes/releases)
[![GitHub license](https://img.shields.io/github/license/alvistack/ansible-collection-kubernetes.svg)](https://github.com/alvistack/ansible-collection-kubernetes/blob/master/LICENSE)
[![Ansible Collection](https://img.shields.io/badge/galaxy-alvistack.kubernetes-blue.svg)](https://galaxy.ansible.com/alvistack/kubernetes)

Ansible collection for deploying Kubernetes.

This Ansible collection provides Ansible playbooks and roles for the deployment and configuration of an [Kubernetes](https://github.com/kubernetes/kubernetes) environment.

## Requirements

This playbook require Ansible 2.9 or higher.

This playbook was designed for:

  - Ubuntu 18.04/19.10/20.04
  - RHEL/CentOS 7/8
  - openSUSE Leap 15.1
  - Debian 10
  - Fedora 32

### Fedora 32

Our default Ceph 15.2 is not supported by Fedora 32 (only Ceph 14.2), with solution:

  - Switch our variable as `ceph_release: "14.2"` for deployment (see [inventory/default/group\_vars/all/00-defaults.yml](inventory/default/group_vars/all/00-defaults.yml))

Fedora 31+ is now using cgroup v2 by default which not supported by kubelet (see <https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/20191118-cgroups-v2.md>), with solution:

  - Setup `systemd.unified_cgroup_hierarchy=1` to the kernel arguments and reboot (see <https://github.com/rancher/k3s/issues/900#issuecomment-551337575>)

## Quick Start

### Bootstrap Ansible and Roles

Start by cloning the Alvistack-Ansible repository, checkout the corresponding branch, and init with `git submodule`, then bootstrap Python3 + Ansible with provided helper script:

    # GIT clone the development branch
    git clone --branch develop https://github.com/alvistack/ansible-collection-kubernetes
    cd ansible-collection-kubernetes
    
    # Setup Roles with GIT submodule
    git submodule init
    git submodule sync
    git submodule update
    
    # Bootstrap Ansible
    ./scripts/bootstrap-ansible.sh
    
    # Confirm the version of Python3, PIP3 and Ansible
    python3 --version
    pip3 --version
    ansible --version

### AIO

All-in-one (AIO) build is a great way to perform an Kubernetes build for:

  - A development environment
  - An overview of how all the Kubernetes services fit together
  - A simple lab deployment

Simply execule our default Molecule test case and it will deploy all default components into your localhost:

    # Run Molecule test case
    molecule test -s default
    
    # Confirm the version and status of Ceph
    ceph --version
    ceph --status
    ceph health detail
    
    # Confirm the version and status of Kubernetes
    kubectl version
    kubectl get node --output wide
    kubectl get pod --all-namespaces

### Production

For production environment we should backed with [Ceph File System](https://docs.ceph.com/docs/master/cephfs/) for [Kubernetes Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) with `ReadWriteMany` support. Corresponding dynamic provisioning could be handled by using [CSI CephFS](https://github.com/ceph/ceph-csi).

Traditionally we could use [Docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker) or [containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd) as [Kubernetes container runtime (CRI)](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/). Now a day, this collection is default with the modern [CRI-O](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o) implementation.

Moreover, we are using [Weave Net](https://github.com/weaveworks/weave) as [Kubernetes network plugin (CNI)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) so we could support [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).

Finally, in order to avoid [Single Point of Failure](https://en.wikipedia.org/wiki/Single_point_of_failure), at least 3 instances for CephFS and 3 instances for Kubernetes is recommended (i.e. 3 + 3 = 6 nodes if CephFS and Kubernetes are running individually; well, or you could also stack up them together so at least 3 nodes).

This deployment will setup the follow components:

  - [Ceph](https://ceph.io/)
  - [Kubernetes](https://kubernetes.io/)
      - CRI: [CRI-O](https://cri-o.io/)
      - CNI: [Weave Net](https://github.com/weaveworks/weave)
      - CSI: [CSI Ceph](https://github.com/ceph/ceph-csi)
      - Addons:
          - [Kubernetes Dashboard](https://github.com/kubernetes/dashboard)
          - [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx)
          - [Cert Manager](https://github.com/jetstack/cert-manager)

Start by copying the default inventory for customization:

    # Copy default inventory
    cp -rfp inventory/default inventory/myinventory

You should update the following files as per your production environment:

    - `inventory/myinventory/hosts`
      - Update with your inventory hostnames and IPs
    - `inventory/myinventory/group_vars/all/00-defaults.yml`
      - Update `*_release` and `*_version` if you hope to pin the deployment into any legacy supported version

Once update now run the playbooks:

    # Run playbooks
    ansible-playbook -i inventory/myinventory/hosts playbooks/converge.yml
    
    # Confirm the version and status of Ceph
    ceph --version
    ceph --status
    ceph health detail
    
    # Confirm the version and status of Kubernetes
    kubectl version
    kubectl get node --output wide
    kubectl get pod --all-namespaces

### Molecule

You could also run our [Molecule](https://molecule.readthedocs.io/en/stable/) test cases if you have [Vagrant](https://www.vagrantup.com/) and [Libvirt](https://libvirt.org/) installed, e.g.

    # Run Molecule on Ubuntu 18.04 with Vagrant and Libvirt
    molecule converge -s ubuntu-18.04

Please refer to [.travis.yml](.travis.yml) for more information on running Molecule.

## License

  - Code released under [Apache License 2.0](LICENSE)
  - Docs released under [CC BY 4.0](http://creativecommons.org/licenses/by/4.0/)

## Author Information

  - Wong Hoi Sing Edison
      - <https://twitter.com/hswong3i>
      - <https://github.com/hswong3i>
