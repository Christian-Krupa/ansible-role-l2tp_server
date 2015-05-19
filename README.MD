ipsec / l2tp server
===================

this role manages vpn-accounts on an openswan vpn-server.
vpn clients have to support l2tp.

#### tested server

* raspberry pi

        Debian GNU/Linux 7.8 (wheezy)
        Linux Openswan U2.6.37/K3.18.7+ (netkey)

* amazon ec2

        Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-accff2b1
        Linux Openswan U2.6.38/K3.13.0-52-generic (netkey)

#### tested clients

* iphone
* ipad


## role variables

`l2tp_server_ipaddress:`

the external ip adress to bind the server on.
example @192.168.0.123@ or @{{ ansible_eth0.ipv4.address }}@

`l2tp_server_ipaddress_private:`

the internal ip adress to bind the server on. most configs use the same ip as `l2tp_server_ipaddress`.
only special network configs like ec2 servers needs a different private ip.
example @172.31.23.239@ or @{{ ansible_eth0.ipv4.address }}@


`l2tp_server_preshared_key:`

a preshared key for every user example @Reishoh3gauweng4eigh@

`l2tp_server_users:`

    chris:
        password: Shohgohwi6ph
    foouser:
        password: weim2Ail5eem


## example host-vars

    l2tp_server_ipaddress: 192.168.0.70
    l2tp_server_ipaddress_private: 192.168.0.70
    l2tp_server_preshared_key: Reishoh3gauweng4eigh
    l2tp_server_users:
        chris:
            password: Shohgohwi6ph
        foouser:
            password: weim2Ail5eem


## config sources

ubuntu / debian:
- http://blog.riobard.com/2010/04/30/l2tp-over-ipsec-ubuntu
- http://sysadmin.compxtreme.ro/setting-up-a-l2tp-over-ipsec-vpn-on-debian-on-10-steps/

raspberry:
- http://linux.tips/tutorials/how-to-setup-l2tp-vpn-server-on-raspberry-pi#comment-2
- http://willitscript.com/post/40357408648/using-your-pi-as-a-l2tp-vpn-server

amazon ec2:
- https://gist.github.com/hwdsl2/9030462

## troubleshooting

#### when you encounter any errors test your system settings
`sudo ipsec verify`

    root@host:~# sudo ipsec verify
    Checking your system to see if IPsec got installed and started correctly:
    Version check and ipsec on-path                             	[OK]
    Linux Openswan U2.6.38/K3.10.23-xxxx-std-ipv6-64 (netkey)
    Checking for IPsec support in kernel                        	[OK]
     SAref kernel support                                       	[N/A]
     NETKEY:  Testing XFRM related proc values                  	[OK]
    	[OK]
    	[OK]
    Checking that pluto is running                              	[OK]
     Pluto listening for IKE on udp 500                         	[OK]
     Pluto listening for NAT-T on udp 4500                      	[OK]
    Two or more interfaces found, checking IP forwarding        	[FAILED]
    Checking NAT and MASQUERADEing                              	[OK]
    Checking for 'ip' command                                   	[OK]
    Checking /bin/sh is not /bin/dash                           	[OK]
    Checking for 'iptables' command                             	[OK]
    Opportunistic Encryption Support                            	[DISABLED]

and read http://blog.riobard.com/2010/04/30/l2tp-over-ipsec-ubuntu

#### your raspberry may not work with the current openswan version

install openswan whith this version (see https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=73962):

    - get_url: url="http://snapshot.raspbian.org/201403301125/raspbian/pool/main/o/openswan/openswan_2.6.37-3_armhf.deb" dest="/tmp/openswan_2.6.37-3_armhf.deb"

or (depending on ansible version)

    #https://github.com/ansible/ansible-modules-core/issues/327
    - apt: deb="/tmp/openswan_2.6.37-3_armhf.deb" force=yes
    - command: "dpkg --force-confdef --force-confold -i /tmp/openswan_2.6.37-3_armhf.deb"

#### odroid u3

syslog show something like

    pluto[31772]: FATAL ERROR: socket() in init_netlink(). Errno 93: Protocol not supported
    ...
    ipsec__plutorun: whack: Pluto is not running (no "/var/run/pluto/pluto.ctl")

the current "Ubuntu 14.04 LTS" does not support ipsec by default: https://kill-9.me/355/odroid-c1-ipsec-support sorry there is no out of the the box solution

#### ubuntu on ovh-server may trigger errors like

    Couldn't open the /dev/ppp device: No such file or directory
    FATAL: Could not load /lib/modules/3.2.13-grsec-xxxx-grs-ipv6-64/modules.dep: No such file or directory
    /usr/sbin/pppd: You need to create the /dev/ppp device node by

executing the following command as root

    mknod /dev/ppp c 108 0

#### amazon ec2

you need to configure the public ip and private ip, collect them with this curl requests (from the ec2-server)

    curl http://169.254.169.254/latest/meta-data/public-ipv4
    curl http://169.254.169.254/latest/meta-data/local-ipv4

### todo's

- [ ] migrate puppet exec

`exec { "/dev/ppp":
      command => 'mknod /dev/ppp c 108 0',
      unless => ["/usr/bin/test -e /dev/ppp"],
      notify => Service['ipsec'],
      logoutput => true
    }`
    
- [ ] auto start iptables command
- [ ] don't add iptables command more than once
- [ ] add dns config (currently google dns servers are used)
- [ ] test cloud based systems ( rackspace, digitalocean)
- [x] test on amazon ec2

## license

MIT

## author

role created by christian krupa 2015
