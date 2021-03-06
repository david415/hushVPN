
onionvpn
========

onionvpn utilizes a tun device to create a virtual-public-network... which
transports ipv6; using tor onion services as the transport. onionvpn does not
provide a cryptographic transport like most VPNs (virtual-private-networks).

onionvpn is now compatible with onioncat! i was able to verify this on my own system
but i did not find any onioncat ipv6 addresses in the wild that would respond to pings.


network configuration
---------------------

Here's how i setup my tun device in linux:

Firstly... make sure the ipv6 INPUT policy is not set to DROP:

    ip6tables -L

Setup the correct policy:

    ip6tables -P INPUT ACCEPT


I use my tor process to generate onion service key material.
You can use onionvpn's ``-displayconfig`` option to generate
Linux shell code for configuring your tun device:

    onionvpn -displayconfig 22u7o5ej47o5z7jf

    # your onion converted to ipv6 address is:
    # fd87:d87e:eb43:d6a9:f774:89e7:dddc:fd25

    ip tuntap add dev tun0 mode tun user user group user
    ip address add scope global fd87:d87e:eb43:d6a9:f774:89e7:dddc:fd25 peer fd87:d87e:eb43::/48 dev tun0
    ifconfig tun0 up


installing
----------

onionvpn depends on txtorcon and scapy. you can install via pip like this:

    pip install git+https://github.com/david415/onionvpn.git


running
-------

onionvpn usage:

    (onion-virtenv)human@debian-python2:~$ onionvpn -h
    WARNING: Failed to execute tcpdump. Check it is installed and in the PATH
    usage: onionvpn [-h] [-onion ONION] [-onion_endpoint ONION_ENDPOINT]
                    [-tun TUN] [-displayconfig ONION-ADDRESS]
    
    onionvpn - onion service tun device adapter
    
    optional arguments:
      -h, --help            show this help message and exit
      -onion ONION          Local onion address
      -onion_endpoint ONION_ENDPOINT
                            Twisted endpoint descriptor string for the onion
                            service which listens on onion virtport 8060
      -tun TUN              tun device name
      -displayconfig ONION-ADDRESS
                            Given an onion address,display ipv6 network
                            configuration for Linux


For example, run it like this:

    onionvpn -onion 22u7o5ej47o5z7jf -onion_endpoint tcp:interface=127.0.0.1:8060 -tun tun0


onion_endpoint must be specified such that it produces a Twisted
endpoint object capable of listening to the onion service on virtport 8060...
The above TCP server endpoint requires you to run tor with the appropriate onion service
configuration, like this:

    HiddenServiceDir /var/lib/tor/onionvpn
    HiddenServicePort 8060 127.0.0.1:8060


This above configuration will work... but you can also use txtorcon's onion service endpoint:

    onionvpn -onion 22u7o5ej47o5z7jf -onion_endpoint onion:8060:hiddenServiceDir=/home/human/onionvpn -tun tun0


That above endpoint results in the txtorcon endpoint launching a new tor instance;
you can instead specify a tor control port:

    onionvpn -onion 22u7o5ej47o5z7jf -onion_endpoint onion:8060:hiddenServiceDir=/home/human/onionvpn:controlPort=/var/run/tor/control -tun tun0


The above onion: endpoint string will work under debian based linux distros if the user belongs to the debian-tor group. Add your user to the debian-tor group like this:

    usermod -a -G debian-tor <USER>

Then as the <USER> join the debian-tor group before running onionvpn:

    newgrp debian-tor

With debian's default tor setup... this should allow you to read the tor
control unix domain socket file in /var/run/tor/control
