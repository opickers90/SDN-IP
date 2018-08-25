# INTRODUCTION :
## ONOS :
What is ONOS?

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

### Architectural goals :

SDN-IP has been designed with the following goals in mind:

1. Compatibility - SDN-IP can be integrated with networks that already use BGP - externally, internally, or both.
Operational flexibility - SDN-IP can run on one or multiple ONOS instances. Moreover, SDN-IP can be used in a variety of BGP deployment scenarios: full-mesh BGP, BGP Route Reflectors, BGP confederations, and with BGP Route Servers.
2. High availability - SDN-IP provides HA within the application itself. The SDN-IP service keeps working seamlessly, as long as there is at least one instance of the SDN-IP application running. In addition, SDN-IP leverages the HA mechanisms provided by ONOS for maintaining a consistent forwarding state in the data plane.
3. Scalability - Large-scale Software Defined Networks can be controlled by using BGP-based Confederations and multiple ONOS clusters, each running SDN-IP.
4. Protocol compatibility and vendor independence - SDN-IP relies on the standard BGP protocol for exchanging network information, and does not require any proprietary or vendor-specific extensions.

source (https://wiki.onosproject.org/display/ONOS/SDN-IP+Architecture)

## INTRODUCTION :
