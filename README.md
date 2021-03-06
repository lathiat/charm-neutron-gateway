Overview
--------

Neutron provides flexible software defined networking (SDN) for OpenStack.

This charm is designed to be used in conjunction with the rest of the OpenStack
related charms in the charm store to virtualize the network that Nova Compute
instances plug into.

It's designed as a replacement for nova-network; however it does not yet
support all of the features of nova-network (such as multihost) so may not
be suitable for all.

Neutron supports a rich plugin/extension framework for propriety networking
solutions and supports (in core) Nicira NVP, NEC, Cisco and others...

See the upstream [Neutron documentation](http://docs.openstack.org/trunk/openstack-network/admin/content/use_cases_single_router.html)
for more details.

Usage
-----

In order to use Neutron with OpenStack, you will need to deploy the
nova-compute and nova-cloud-controller charms with the network-manager
configuration set to 'Neutron':

    nova-cloud-controller:
        network-manager: Neutron

This decision must be made prior to deploying OpenStack with Juju as
Neutron is deployed baked into these charms from install onwards:

    juju deploy nova-compute
    juju deploy --config config.yaml nova-cloud-controller
    juju add-relation nova-compute nova-cloud-controller

The Neutron Gateway can then be added to the deploying:

    juju deploy neutron-gateway
    juju add-relation neutron-gateway mysql
    juju add-relation neutron-gateway rabbitmq-server
    juju add-relation neutron-gateway nova-cloud-controller

The gateway provides two key services; L3 network routing and DHCP services.

These are both required in a fully functional Neutron OpenStack deployment.

See upstream [Neutron multi extnet](http://docs.openstack.org/trunk/config-reference/content/adv_cfg_l3_agent_multi_extnet.html)

Configuration Options
---------------------

Port Configuration
==================

All network types (internal, external) are configured with bridge-mappings and
data-port.  Once deployed, you can configure the network specifics using
neutron net-create.

If the device name is not consistent between hosts, you can specify the same
bridge multiple times with MAC addresses instead of interface names.  The charm
will loop through the list and configure the first matching interface.

Basic configuration of a single external network, typically used as floating IP
addresses combined with a GRE private network:

    neutron-gateway:
        bridge-mappings:         physnet1:br-ex
        data-port:               br-ex:eth1

Alternative configuration with two networks, where the internal private
network is directly connected to the gateway with public IP addresses but a
floating IP address range is also offered.

    neutron-gateway:
        bridge-mappings:         physnet1:br-data external:br-ex
        data-port:               br-data:eth1 br-ex:eth2

Alternative configuration with two external networks, one for public instance
addresses and one for floating IP addresses.  Both networks are on the same
physical network connection (but they might be on different VLANs, that is
configured later using neutron net-create).

    neutron-gateway:
        bridge-mappings:         physnet1:br-data
        data-port:               br-data:eth1

    # neutron net-create --provider:network_type vlan --provider:segmentation_id 400 --provider:physical_network physnet1 --shared external
    # neutron net-create --provider:network_type vlan --provider:segmentation_id 401 --provider:physical_network physnet1 --router:external=true router_uplink

This replaces the previous system of using ext-port, which always created a bridge
called br-ex for external networks which was used implicitly by external router
interfaces.

Instance MTU
============

When using Open vSwitch plugin with GRE tunnels default MTU of 1500 can cause
packet fragmentation due to GRE overhead. One solution is to increase the MTU on
physical hosts and network equipment. When this is not possible or practical the
charm's instance-mtu option can be used to reduce instance MTU via DHCP.

    juju set neutron-gateway instance-mtu=1400

OpenStack upstream documentation recommends a MTU value of 1400:
[OpenStack documentation](http://docs.openstack.org/admin-guide-cloud/content/openvswitch_plugin.html)

Note that this option was added in Havana and will be ignored in older releases.

Deploying from source
=====================

The minimum openstack-origin-git config required to deploy from source is:

    openstack-origin-git: include-file://neutron-juno.yaml

    neutron-juno.yaml
    -----------------
    repositories:
    - {name: requirements,
       repository: 'git://github.com/openstack/requirements',
       branch: stable/juno}
    - {name: neutron,
       repository: 'git://github.com/openstack/neutron',
       branch: stable/juno}

Note that there are only two 'name' values the charm knows about: 'requirements'
and 'neutron'. These repositories must correspond to these 'name' values.
Additionally, the requirements repository must be specified first and the
neutron repository must be specified last. All other repositories are installed
in the order in which they are specified.

The following is a full list of current tip repos (may not be up-to-date):

    openstack-origin-git: include-file://neutron-master.yaml

    neutron-master.yaml
    -------------------
    repositories:
    - {name: requirements,
       repository: 'git://github.com/openstack/requirements',
       branch: master}
    - {name: oslo-concurrency,
       repository: 'git://github.com/openstack/oslo.concurrency',
       branch: master}
    - {name: oslo-config,
       repository: 'git://github.com/openstack/oslo.config',
       branch: master}
    - {name: oslo-context,
       repository: 'git://github.com/openstack/oslo.context',
       branch: master}
    - {name: oslo-db,
       repository: 'git://github.com/openstack/oslo.db',
       branch: master}
    - {name: oslo-i18n,
       repository: 'git://github.com/openstack/oslo.i18n',
       branch: master}
    - {name: oslo-messaging,
       repository: 'git://github.com/openstack/oslo.messaging',
       branch: master}
    - {name: oslo-middleware,
       repository': 'git://github.com/openstack/oslo.middleware',
       branch: master}
    - {name: oslo-rootwrap',
       repository: 'git://github.com/openstack/oslo.rootwrap',
       branch: master}
    - {name: oslo-serialization,
       repository: 'git://github.com/openstack/oslo.serialization',
       branch: master}
    - {name: oslo-utils,
       repository: 'git://github.com/openstack/oslo.utils',
       branch: master}
    - {name: pbr,
       repository: 'git://github.com/openstack-dev/pbr',
       branch: master}
    - {name: stevedore,
       repository: 'git://github.com/openstack/stevedore',
       branch: 'master'}
    - {name: python-keystoneclient,
       repository: 'git://github.com/openstack/python-keystoneclient',
       branch: master}
    - {name: python-neutronclient,
       repository: 'git://github.com/openstack/python-neutronclient',
       branch: master}
    - {name: python-novaclient,
       repository': 'git://github.com/openstack/python-novaclient',
       branch: master}
    - {name: keystonemiddleware,
       repository: 'git://github.com/openstack/keystonemiddleware',
       branch: master}
    - {name: neutron-fwaas,
       repository': 'git://github.com/openstack/neutron-fwaas',
       branch: master}
    - {name: neutron-lbaas,
       repository: 'git://github.com/openstack/neutron-lbaas',
       branch: master}
    - {name: neutron-vpnaas,
       repository: 'git://github.com/openstack/neutron-vpnaas',
       branch: master}
    - {name: neutron,
       repository: 'git://github.com/openstack/neutron',
       branch: master}
