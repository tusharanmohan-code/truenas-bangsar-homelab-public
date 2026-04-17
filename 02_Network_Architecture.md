# 02_Network_Architecture.md

## Network Architecture Overview

The TrueNAS homelab operates within a structured LAN environment behind a consumer fiber connection. The network design prioritizes stability, simplicity, and service accessibility.

---

## 1. ISP & Internet Layer

### Internet Service Provider
Fiber ISP

### Bandwidth
300 Mbps Download / 300 Mbps Upload

### Termination Device
GPON Fiber Modem (ISP-provided)

The GPON modem handles fiber termination and passes traffic to the main router for internal LAN management.

---

## 2. Core Routing Layer

### Main Router
Xiaomi AX3000 Mesh Router

Responsibilities:
- DHCP Server
- NAT (Network Address Translation)
- Firewall management
- Internal LAN routing
- Wi-Fi distribution (2.4GHz & 5GHz)

The router acts as the central gateway between WAN and LAN devices.

---

## 3. Switching Layer

### Device
TP-Link 8-Port Unmanaged Switch

Purpose:
- Expands wired LAN connectivity
- Provides stable Ethernet connections for high-bandwidth devices

Connected Devices:
- TrueNAS Server
- Smart TV
- PlayStation 5

The switch operates at Layer 2 and does not perform routing functions.

---

## 4. Network Flow Diagram

Internet
> GPON Modem
> Xiaomi AX3000 Router
> TP-Link 8-Port Switch
> TrueNAS Server / Smart TV / PlayStation 5

---

## 5. NAS Network Integration

The TrueNAS server is connected via Ethernet (wired connection) to ensure:

- Stable throughput for media streaming
- Lower latency compared to Wi-Fi
- Consistent transfer speeds for backups and file transfers

The NAS receives IP configuration via DHCP from the router.

---

## 6. Network Design Decisions

| Decision                       | Reason                                  |
|--------------------------------|-----------------------------------------|
| Wired NAS connection           | Stability and throughput                |
| Dedicated router for DHCP/NAT  | Clean separation of responsibilities    |
| Unmanaged switch               | Simplicity and low overhead             |
| Single LAN subnet              | Reduced complexity for homelab scale    |

---

## 7. Security & Access

### Internal Access
Services are accessible via local IP within LAN.

### Remote Access
Tailscale is deployed to provide:
- Secure encrypted remote access
- No port forwarding required
- Reduced external attack surface

This avoids exposing NAS services directly to the public internet.

---

## 8. Limitations

- No VLAN segmentation
- No managed switch for traffic control
- No IDS/IPS layer
- Consumer-grade router hardware

These limitations are acceptable for a learning-focused homelab environment.

---

## 9. Engineering Assessment

This network demonstrates:

- Layered network design (WAN > Router > Switch > Clients)
- Understanding of routing vs switching roles
- Secure remote access implementation
- Stability-focused NAS deployment
- Risk-aware exposure strategy

---

End of Network Architecture Documentation
