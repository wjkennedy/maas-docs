Title: Installation and Configuration Checklist

# William's notes
Building upon the checklist document, I felt I needed to provide some additional details and capture my own notes.

I may submit a pull request against a future revision of this doc. Nothing I did here is in the bug list.

# Installation and Configuration Checklist

This is a guide that you can use to achieve a working MAAS environment. Once
you perform a step come back here (using the menu in the left pane) to continue
on to the next one.


## Software installation

As explained [here][about-maas], the installation of MAAS consists of the
installation of a rack controller and a region controller which, in turn,
provide a multitude of services. Go ahead and [install MAAS][install-maas].

    apt install maas
    
### Create MAAS administrator user
    sudo maas createadmin
This user will log in to the Web UI using the name and password assigned with `maas createadmin`.

## Access the web UI

Access the [web UI][web-ui] now. This will involve the creation of
an administrator user. Notice how the web UI (API server) is accessed via port
5240 and not port 80.

[**http://yourserver:5240/MAAS/**](http://yourserver:5240/MAAS/)

!!! Note: Although the web UI may be accessed via port 80, this is not
guaranteed to work in future versions of MAAS.

If you have not yet created an admin user, you will see a message prompting you to do so:

![Create admin user](create_admin_user.png)

## Zones

For [zones][zones], many people won't need to change anything, as a default zone
is provided out of the box.

The default zone will contain any auto-discovered nodes available for commissioning, and will show any manually assigned nodes, to be configured in a later step.

## Networking

The Networking tab will show details related to the Networks known to the Region/Rack controller:

|Fabric|VLAN|Subnet|Available IPs|Space|
|------|----|------|-------------|-----|
|The named fabric|VLAN assignment|Subnet assignment|IP address range|Space ID|


In terms of IP addresses, understand what a *reserved range* is by
reading [Concepts and terms](intro-concepts.md#ip-ranges). Create one (not
*reserved dynamic range*) if you need one.

If required, configure a default gateway and a nameserver that your nodes will
use. See [Networking](installconfig-networking.md) for details on how to do this.

To reset or reconfigure the network interface used by MAAS for DHCP & PXE, you may run:

    dpkg-reconfigure maas-region-controller

<!--
[networks][networks] (points to skeleton page installconfig-network.md)
Lots of stuff can be added to it - see https://git.io/vPLHk
-->


## Import boot images

While you were in the web UI you may have seen a hint to *import boot images*.
[Read up on images][images] as they're quite important. Continue reading until
you have discovered how to import them. You will see that you have the choice
to use the CLI to do this.


!!! Note: The import process can take a while. Consider moving on and coming
back. Just ensure that the import has completed prior to adding a node.


## Access the MAAS CLI

Even if you've imported images with the web UI, it would be wise to
give the [CLI](manage-cli.md) a spin in case you ever need to use it later.
Although we strive to make the web UI feature-equivalent to the CLI, some
things can still only be done with the CLI.

    maas --help

## Enable DHCP

You won't get far without DHCP since it is required in order to make PXE work,
which, in turn, is necessary to introduce your systems to MAAS. 

Choose a VLAN by clicking on the VLAN "tag" in the Network tab, and choose "Provide DHCP" from the "Take Action" dropdown.

![Provide DHCP on tagged VLAN](enable_dhcp.jpg)


## Users and SSH keys

You already have an administrative user but MAAS can also have regular users
(who log in to the interface or use the CLI). What users you create depends on
how you intend to use MAAS.

Additionally, in order for a user to log into a MAAS-deployed machine that user
*must* have their public SSH key installed on it.

Study [user accounts][user-accounts] to learn about how to create more users
and how to add their SSH keys to MAAS. Once that's done, every deployed machine
will automatically have that key installed.


## Add a node

MAAS manages nodes, but at this time it
doesn't have any. [Add a node][add-nodes] now. You need
a spare physical system, but KVM can also be used to provision a node for commissioning.

###KVM Node
Creating a virtual machine that can be managed as a node with MAAS requires:

* Virtual network interface on a bridged interface
* Virtual storage

In the web UI, confirm that the
import of images has finished.

###Create the KVM node

A KVM virtual machine can be created via the cli with the virsh tools, or with an X application, `virt-manager`.

####Prerequisites

* libvirt-bin

* virt-manager

####CLI
The CLI method uses the 'virt-install' tool provided by the libvirt-bin package. This example will provision a KVM virtual machine node suitable for Commissioning with MAAS. Note: The network used by the node should be the same network interface known to KVM, e.g., ***virbr0*** 

    virt-install
     
####Virt Manager
The virt-manager GUI can be used to quickly create and graphically manage a virtual machine node.



###Verify

Go to the 'Nodes' page in the web UI. A successfully added node will soon
appear there with a status of *New*.

You will need to find the MAC address associated with your KVM node for assignment in the 

## Edit power type

A node needs to power cycle while being managed by MAAS. The next step is
therefore to tell MAAS how to do this. That is, you need to
[edit the power type][power-type] of the node's BMC.

For a KVM node, use the "Virsh (Virtual systems) power type.

Physical systems without IPMI or BMC interfaces should use "Manual".

## Commission a node

Commissioning a node involves MAAS testing it to ensure that it is able to
communicate properly with the region API server. [Commission][commission-nodes]
your node now.


## Deploy a node

Lots of folks would have [Juju][juju-site] take over at this point. Juju acts
as a sort of command & control centre for adding services/applications on top
of MAAS nodes (among other "clouds"). If you're just not there and/or you want
to quickly test things out you can use the web UI to
[deploy a node][deploy-nodes] directly.


## SSH to the node

If you [imported your SSH key][ssh-keys] then you should now be able to ssh to
the deployed node by connecting to the 'ubuntu' account. The node's page in the
web UI will inform you of its IP address. Mission accomplished!


<!-- LINKS -->
[about-maas]: index.md#key-components-and-colocation-of-all-services
[install-maas]: installconfig-install.md
[web-ui]: installconfig-gui.md
[zones]: installconfig-zones.md
[networks]: installconfig-networking.md
[images]: installconfig-images.md
[dhcp]: installconfig-subnets-dhcp.md
[add-nodes]: installconfig-add-nodes.md
[user-accounts]: manage-account.md
[power-type]: installconfig-power-types.md
[commission-nodes]: installconfig-commission-nodes.md
[juju-site]: https://jujucharms.com/docs/
[deploy-nodes]: installconfig-deploy-nodes.md
[ssh-keys]: manage-account.md#ssh-keys

