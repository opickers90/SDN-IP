# SDN IP DOCUMENTATION
# INTRODUCTION :
## What is ONOS?

ONOS stands for Open Network Operating System. ONOS provides the control plane for a software-defined network (SDN), managing network components, such as switches and links, and running software programs or modules to provide communication services to end hosts and neighboring networks.

The most important benefit of an operating system is that it provides a useful and usable platform for software programs designed for a particular application or use case. ONOS applications and use cases often consist of customized communication routing, management, or monitoring services for software-defined networks. Some examples of things which you can do with ONOS, and software written to run on ONOS, may be found in Apps and Use Cases.

ONOS can run as a distributed system across multiple servers, allowing it to use the CPU and memory resources of multiple servers while providing fault tolerance in the face of server failure and potentially supporting live/rolling upgrades of hardware and software without interrupting network traffic.

The ONOS kernel and core services, as well as ONOS applications, are written in Java as bundles that are loaded into the Karaf OSGi container. OSGi is a component system for Java that allows modules to be installed and run dynamically in a single JVM. Since ONOS runs in the JVM, it can run on several underlying OS platforms. More information on the internal design of ONOS may be found in Architecture and Internals Guide, and information on developing ONOS applications may be found in the Developer Guide.

ONOS is an open source project backed by an expanding community of developers and users, and you are invited and encouraged to join in discussion, development, documentation, and improvement of the ONOS system. This document itself is part of the ONOS wiki, so please feel free to fix any errors, add clarifying comments, and improve it as you see fit.

Welcome to ONOS!

source(https://wiki.onosproject.org/display/ONOS/Wiki+Home)

### ONOS DOWNLOAD :
We can download ONOS Contoller from this (https://wiki.onosproject.org/display/ONOS/Downloads)

## SDN-IP :
SDN-IP is an ONOS application that allows a Software Defined Network to connect to external networks on the Internet using the standard Border Gateway Protocol (BGP). Externally, from a BGP perspective, the SDN network appears as a single Autonomous System (AS) that behaves as any traditional AS. Within the AS, the SDN-IP application provides the integration mechanism between BGP and ONOS. At the protocol level, SDN-IP behaves as a regular BGP speaker. From ONOS perspective, it’s just an application that uses its services to install and update the appropriate forwarding state in the SDN data plane.

![SDN-IP Overview](https://wiki.onosproject.org/download/attachments/2130846/overview.png?version=1&modificationDate=1417566171959&api=v2)

In a typical deployment scenario, an SDN-IP controlled network acts as a transit Autonomous System (AS) interconnecting different IP networks. Each external network is a different AS domain, which interfaces with our SDN network through its BGP-speaking border routers.

The SDN-IP network is composed of OpenFlow switches controlled by ONOS. While SDN-IP can be used even with a single ONOS instance, for high availability and scalability, there should be a cluster of ONOS instances, where some of those instances run the SDN-IP application.

Within the SDN-IP network, there are one or more internal BGP speakers. The BGP speakers can be existing BGP routers or any software that implements BGP.  The speakers use eBGP to exchange BGP routing information with the border routers of the adjacent external networks, and iBGP to propagate that information amongst themselves and to the SDN-IP application instances.

The routes advertised by the external border routers belonging to external networks are received by the BGP speakers within the SDN-IP network, processed, and eventually re-advertised to the other external networks. The routes are processed according to the normal BGP processing and routing policies. Similarly, those routes are also advertised to the SDN-IP application instances which act as iBGP peers. The best route for each destination is selected by the SDN-IP application according to the iBGP rules, and translated into an ONOS Application Intent Request. ONOS translates the Application Intent Request into forwarding rules in the data plane. Those rules are used to forward the transit traffic between the interconnected IP networks.

In addition, SDN-IP creates the appropriate ONOS Intents that would allow the external BGP routers to peer with the BGP speakers by using the data plane. Thus, BGP itself can also detect failures in the data plane that affect the transiting traffic and act accordingly.

Among the SDN-IP application instances, each instance receives and contains exactly the same set of BGP routes (and the corresponding Application Intent Requests), but only one instance, the SDN-IP Leader, is responsible for making the appropriate ONOS API calls to install the necessary intents at any given time.

### Architectural goals :
SDN-IP has been designed with the following goals in mind:

1. Compatibility - SDN-IP can be integrated with networks that already use BGP - externally, internally, or both.
Operational flexibility - SDN-IP can run on one or multiple ONOS instances. Moreover, SDN-IP can be used in a variety of BGP deployment scenarios: full-mesh BGP, BGP Route Reflectors, BGP confederations, and with BGP Route Servers.
2. High availability - SDN-IP provides HA within the application itself. The SDN-IP service keeps working seamlessly, as long as there is at least one instance of the SDN-IP application running. In addition, SDN-IP leverages the HA mechanisms provided by ONOS for maintaining a consistent forwarding state in the data plane.
3. Scalability - Large-scale Software Defined Networks can be controlled by using BGP-based Confederations and multiple ONOS clusters, each running SDN-IP.
4. Protocol compatibility and vendor independence - SDN-IP relies on the standard BGP protocol for exchanging network information, and does not require any proprietary or vendor-specific extensions.

source (https://wiki.onosproject.org/display/ONOS/SDN-IP+Architecture)

## TUTORIAL :
### TestBed Environment Topology :
![Topology](https://github.com/opickers90/SDN-IP/blob/master/SDNIP%20Topology.png?raw=true)

|No   |Device   |Port Number   |IP Address   |AS NUMBER   |
|---|---|---|---|---|
| 1  |Ubuntu Server (ONOS + Speaker)   |Management |10.10.11.11/20|     |
|    |    |Eth0-1    |172.16.1.1/24   |65000   |
|   |   |Eth0-2   |172.16.2.1/24   |65000   |
|2   |Linux Quagga Router   |Management |10.10.11.12/20|       |
|    |    |Eth0   |172.16.1.2/24   |65001   |
|3   |Cisco Router 2811   |Management | 10.10.11.13/20 |     |
|    |     |Fast Ethernet 0/1   |172.16.2.2/24   |65002   |
|4   |Pica 8 Switch   |Management | 10.10.11.14/20|   |
|   |   |1/1/1   |   |   |
|   |   |1/1/2   |   |   |
|   |   |1/1/3   |   |   |

### Device Installation & Configuration:
#### Ubuntu Server :
##### Speaker Router and ONOS Controller  : 
1. Install Ubuntu Server 16.04 LTS in PC Server
2. Configure subinterface :
  ```sa
  ip link set up dev eth0
  ip link add link eth0 dev eth0-1 address 00:00:00:00:00:a1 type macvlan
  ip link add link eth0 dev eth0-2 address 00:00:00:00:00:a2 type macvlan
  ```
 3. Set IP address for each interace :
 ```
  ifconfig eth0-1 172.16.1.1 netmask 255.255.255.0 up
  ifconfig eth0-2 172.16.2.1 netmask 255.255.255.0 up
  ```
  4. Install and Configure Quagga Router :
  ```
  apt-get install quagga
  ```
  Enable IPv4 BGP Daemon :
  ```
  vim /etc/quagga/daemons
  ```
  ```
  zebra=yes
  bgpd=yes
  ospfd=no
  ospf6d=no
  ripd=no
  ripngd=no
  ```
  Restart Quagga Service :
  ```
  #/etc/init.d/quagga restart
  ```
  Check if Quagga service already up :
  ```
  #ps -ef | grep quagga
  ```
  ```
  UID 	PID 	PPID 	C 	STIME 	TTY 	TIME 	CMD
  quagga 	4632 	1 	0 	22:25 	? 	00:00:00 	/usr/lib/quagga/bgpd --daemon
  quagga 	4636 	1 	0 	22:25 	? 	00:00:00 	/usr/lib/quagga/zebra --daemon
  ```
  5. Configure Quagga BGP Files and change filename to Speaker:
  ```
  #cp /usr/share/doc/quagga/examples/zebra.conf.sample /etc/quagga/zebra.conf
  #cp /usr/share/doc/quagga/examples/bgpd.conf.sample /etc/quagga/speakerd.conf 
  #chown quagga.quaggavty /etc/quagga/*.conf
  #chmod 640 /etc/quagga/*.conf 
  ```
  Configure BGP Routing : 
  ```
  vim /etc/quagga/speakerd.conf
  ```
  ```
  !
  hostname speaker1
  password sdnip
  log stdout
  !
  router bgp 65000
  bgp router-id 10.10.11.11
  timers bgp 3 9
  ! ONOS SDN IP
  neighbor 127.0.0.1 remote-as 65000
  neighbor 127.0.0.1 port 2000
  neighbor 127.0.0.1 timers connect 5
  ! BGP External 1
  neighbor 172.16.1.2 remote-as 65001
  neighbor 172.16.1.2 ebgp-multihop 255
  neighbor 172.16.1.2 advertisement-interval 5
  neighbor 172.16.1.2 timers connect 5
  !  BGP External 2
  neighbor 172.16.2.2 remote-as 65002
  neighbor 172.16.2.2 ebgp-multihop 255
  neighbor 172.16.2.2 advertisement-interval 5
  ! 
  line vty
  !
  end
  ```
  Restart Quagga Service :
  ```
  #/etc/init.d/quagga restart
  ```
  Configure IP Forwarding :
  ```
  #echo "1" > /proc/sys/net/ipv4/ip_forward
  #echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
  ```
  
 5.  Download ONOS Contoller from http://repo1.maven.org/maven2/org/onosproject/onos-releases/1.13.2/onos-1.13.2.zip (in my tutorial, i am using ONOS-1.13.2)
 ```
 $ wget http://repo1.maven.org/maven2/org/onosproject/onos-releases/1.13.2/onos-1.13.2.zip 
```
extract ONOS zip file :
```
$ unzip onos-1.13.2.zip
```
7. Install JAVA: 
```
$ sudo apt-get install software-properties-common -y && \
  sudo add-apt-repository ppa:webupd8team/java -y && \
  sudo apt-get update && \
  echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections && \
  sudo apt-get install oracle-java8-installer oracle-java8-set-default -y
``` 
8. Start ONOS:
```
$ cd onos-1.13.2/bin
`~/onos-1.13.2/bin$./onos-service start
```
```
Welcome to Open Network Operating System (ONOS)!
     ____  _  ______  ____    
    / __ \/ |/ / __ \/ __/  
   / /_/ /    / /_/ /\ \    
   \____/_/|_/\____/___/    
Documentation: wiki.onosproject.org      
Tutorials:     tutorials.onosproject.org
Mailing lists: lists.onosproject.org    
Come help out! Find out how at: contribute.onosproject.org
Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown ONOS.
```
9. Install the application:
```
onos> app activate org.onosproject.openflow-base
onos> app activate org.onosproject.openflow
onos> app activate org.onosproject.fwd
onos> app activate org.onosproject.config
onos> app activate org.onosproject.proxyarp
onos> app activate org.onosproject.sdnip
```

8.  Create ONOS Configuration files and configure BGP interface (netcfg.json) in /tmp/ and push to ONOS:
```
vim /tmp/netcfg.json
```
```
{
    "ports" : {
        "of:0000000000000001/1" : {
            "interfaces" : [
                {
                    "ips"  : [ "172.16.1.1/24" ],
                    "mac"  : "52:54:00:ec:4c:59"
                }
            ]
        },
        "of:0000000000000001/2" : {
            "interfaces" : [
                {
                    "ips"  : [ "172.16.2.1/24" ],
                    "mac"  : "00:14:6a:a3:08:11"
                }
            ]
        }
    },    
    "apps" : {
       "org.onosproject.router" : {
            "bgp" : {
                "bgpSpeakers" : [
                    {
                        "name" : "speaker1",
                        "connectPoint" : "of:0000000000000001/3",
                        "peers" : [
                            "172.16.1.2",
			    "172.16.2.2"
                        ]
                    }
                ]
            }
        }
    }
}
```
```
curl --user onos:rocks -X POST -H "Content-Type: application/json" http://10.10.11.11:8181/onos/v1/network/configuration/ -d @/tmp/cfg.json
```
