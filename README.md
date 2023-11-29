# Hydrogen Protocol

The Hydrogen Protocol (HP) is a Low Power Wide Area Network (LPWAN) roaming and interoperability framework for IoT networks that can be used on top of existing IoT protocol physical layers (PHYs). The purpose of HP is to enable development of unified IoT applications that can operate across a range of IoT technologies, network operators, and regulatory environments. HP builds on existing Internet standards including IPv6 and the Border Gateway Protocol (BGP).

[Intro figure showing IPv6 Anycast network](https://app.eraser.io/workspace/uv5MXZIUEEOrvsbqiRAy/preview?elements=JeGGfq_uICFBAou6Bl_Qsw&type=embed)


**HP goals include:**
- Protocol- and provider-independent deployment of a unified IPv6 IoT network with cross-provider/network roaming, adapting existing internet standards to IoT environments.
- Provide a framework to support device roaming and traffic peering between network operators with parallel networks using the same PHY layer (e.g. regional LoRaWAN providers).
- Providing IPv6 compatibility, while minimizing data overhead for power and bandwidth constrained devices (33-45 byte overhead per message frame).
- Providing device authentication and end to end encryption using best-available methods, with ability to support new  encryption methods.  
- Provide secure, lightweight over-the-air (OTA) device provisioning wherever  supported by the PHY layer.
- Layered implementation allowing deployment alongside existing LPWAN protocols at the application-level, or as a replacement for Layer 2-3 data link and network implementations.
- Provide a transitional implementation for network operators and protocols that do not yet support standards-based Layer 2-3 
- Allow for federated implementation via standard protocols. Anyone can deploy an HP network and devices by provisioning an IPv6 /48 address block and an ASN from their regional internet registry. By relying on existing internet routing infrastructure to identify networks there's no need for a central network registry or peering infrastructure as has been [﻿developed proposed for existing LPWAN protocols](https://www.thethingsindustries.com/peering/).


**HP non-goals include:**

- Providing or specifying LPWAN a complete protocol. HP is designed to work across a range of existing LPWAN PHY implementations, including open spectrum and carrier-based services. HP prvodies Layer 2-3 (e.g. hardware MAC and network IP address) services and integrates with existing IoT PHY layers. HP can augment also LPWAN technologies with Layer 2-3 services, providing application-level interoperability via on-device firmware or at a network gateway.
- Provide a facility for paid or conditional traffic peering. Traffic peering rules and billing could be implemented on top of HP networks, however, the HP protocol recommends free/open peering where upstream network costs allow, and transparent bilateral and multilateral agreements only where required to ensure financial sustainability of peering relationships.
- Provide an OTA provisioning method for all PHY layers. We are working with hardware vendors and network operators that are building OTA provision for IoT devices via carrier-based protocols (e.g. iSIM/SoftSIM), however, HP works alongside existing carrier provisioning requirements.  
- Force network operators to alter existing deployments to support IPv6-based IoT. However, we hope the project will facilitate on-going transition efforts as carriers migrate to IPv6.


**What does it do?**

HP provides a lightweight IPv6 implementation geared towards power and bandwidth constrained IoT devices. HP uses the IPv6 address space to identify and authenticate devices, including on networks that don't yet support IPv6 addressing, as is the case for many mobile network operators today. 

By combining  IPv6 and IPsec implementations augmented for IoT with BGP, HP provides a standards-aligned framework for device authentication that spans network and protocol boundaries, and reduces per-message overhead. 

HP is designed to complement existing LPWAN technologies, providing a set of services that can be layered on top of existing protocols that already provide device-level authentication (e.g. LTE/NB-IoT), and can be implemented on-device for end-to-end deployment, or as a proxy/gateway into existing networks.

Messages sent to or from devices use a HP network gateway for device authentication and as a data relay for subscribing applications. HP combines global, internet routable addresses, PKI-based device authentication, and message security and integrity. By using a single 128 bit IPv6 address to define the device, and a BGP hosted subnet as the gateway, HP provides a compact, self-describing IoT network that leverages existing internet address space and protocols.


 **How does it work?**

HP uses 128 bit IPv6 addresses for device identification and network roaming (see figure 1).  The first 48 bits the routing prefix for a BGP-provisioned publicly routed IPv6 network, followed by a 16 bit application ID, and a 64 bit device identifier. The initial 48 bit network ID serves as the gateway address for all devices on this network. 

![Figure 1: HP IPv6 Addressing](https://app.eraser.io/workspace/uv5MXZIUEEOrvsbqiRAy/preview?elements=t5zMdtEgnCZodQQteiVgTA&type=embed)
Figure 1: HP IPv6 Addressing 

The HP message frame combines the IPv6 address with an 8 bit header containing HP protocol settings and 128 bit signature generated using network provided public key infrastructure (PKI), and an optional 96 bit nonce. This results in a 33-45 byte overhead per message frame. 

![Figure 2: HP message frame](https://app.eraser.io/workspace/uv5MXZIUEEOrvsbqiRAy/preview?elements=U0waJFDwEAViroloVTkH0g&type=embed)
Figure 2: HP message frame

HP uses local relays, with IoT PHY-specific radios (e.g. LoRa, MIOTY, NB-IoT) to relay messages to and from IoT devices and transfer them to the HP network gateway. 

The relay runs a minimal HP protocol implementation that can parse the HP message headers, and (optionally) verify the message was sent from a valid, authenticated device using PKI. If the local relay supports IPv6 then it transfers the complete HP message directly to the network gateway defined by the first 48 bits of the device's address. In the case of relays with upstream networks (either wired or wireless) that don't offer native IPv6 support, the relay can forward the message (e.g. via IPv4 tunneling) to an upstream relay that can reach the gateway via IPv6.

This approach allows any local relay that supports the IoT PHY layer (e.g. LoRa, NB-IoT) to forward messages to the correct network gateway without any additional knowledge about the device or its network beyond the IPv6 address. This is different from existing IoT networks which require PHY and protocol specific look-ups to resolve devices and network gateways. A key tenet of HP is open and transparent traffic peering between networks wherever upstream bandwidth costs allow. 

![Figure 3: simplified HP network architecture with direct relay-gateway connection](https://app.eraser.io/workspace/uv5MXZIUEEOrvsbqiRAy/preview?elements=nW_saSvpS3eF8PLnOYrGJg&type=embed)
Figure 3: simplified HP network architecture with direct relay-gateway connection

HP devices are provisioned using device- and network-specific key pairs configured via Elliptic Curve Diffie Hellman (ECDH) key exchange. Once provisioned messages are signed using the device's key,  message-level encryption can be performed using device, network, or application-specific keys using best-available methods (e.g. AES-GCM or ChaCha20).

Messages received by the gateway are placed in queues and can be subscribed to via the corresponding gateway, application,or device ID. This pub/sub style message broker builds in existing IoT best practices (e.g. MQTT), adding a unified device/queue address space and message source authentication. The gateway message queues operate bidirectionally and can stage data for transfer to intermittently connected IoT devices.

![Figure 4: IoT connections via IPv6 gateway](https://app.eraser.io/workspace/uv5MXZIUEEOrvsbqiRAy/preview?elements=GFSq1jMCh4CO3fZLDYhBTQ&type=embed)
Figure 4: IoT connections via IPv6 gateway

**Hydrogen Protocol Device Provisioning  and Network Management**

Devices and relays can be provisioned and operated using a lightweight protocol that enables control plane communication between IoT devices, relays and network gateways. These messages are configured in the 8 bit HP message header (header format TK). 



**Device provisioning lifecycle**

- REQUEST ADDRESS (optional -- IDs can be predefined)
    - Allows dynamic OTA assignment of device IDs (full 128 bit address) for devices with knowledge of a network or application address and public key. Device IDs once assigned are permanent and are never reused (responsibility of the network provisioning infrastructure to enforce this rule).
    - Once an address is assigned the device can request access and provision device specific keys.
- REQUEST ACCESS (optional -- keys can be predefined)
    - Allows dynamic OTA provisioning of device / network / application key pairs using ECDF key exchange.
    - Once keys are signed the device can send and receive messages until the device key is deprecated by the network.
    - Deprecated or compromised keys can be replaced at any time via an additional REQUEST ACCESS call.
- CONFIGURE RECEIVE 
    - Configures device-specific settings for receiving messages. These settings are often defined by the PHY and can be configured at an application or network-level if desired.
        - Synchronous/Asynchronous mode
        - Receive after send period
        - Receive polling frequency 
        - Receive polling period


**Message transfer**

- SEND [+ WAIT]
    - Sends a message and optionally waits to receive for a device or application defined period.
- RECEIVE
    - Polls for messages waiting to be transferred to device
- ACK 
    - Device acknowledge or heartbeat without a receive period.














