# host-security-threat-detection-lab
A hands-on network security lab simulating adversarial reconnaissance, host-based boundary hardening (UFW), and live packet payload extraction using Wireshark.

# End-to-End Threat Detection & Data Analysis Lab
**Bridging the Gap from Endpoint Support to Security Operations (SOC)**

## Project Overview
This hands-on project simulates a complete security lifecycle: from initial adversarial reconnaissance to network defensive hardening, concluding with data triage. By building a local virtual sandbox, I analyzed raw packet metrics to detect stealth attacks, configured perimeter rules to minimize the attack surface, and analyzed unencrypted data leakage streams directly on the wire.

## 🛠️ Lab Infrastructure & Topology
*   **Hypervisor:** VMware Workstation 17 Pro
*   **Network Architecture:** Private NAT Subnet (`10.0.110.0/24`)
*   **Attacker Node (Adversary):** Kali Linux (`10.0.110.55`)
*   **Target Node (Endpoint/Defender):** Ubuntu Server 26.04 (`10.0.110.52`)

---

## 🛑 Phase 1: Reconnaissance Detection via Packet Analysis
**Objective:** Identify and dissect stealth network mapping techniques before an escalation occurs.

### Execution
An aggressive Stealth SYN Scan (`nmap -sS`) was launched from the Kali adversary node to discover listening ports on the target endpoint.

### Technical Analysis
By capturing raw traffic on the target interface using **Wireshark**, I observed the mechanics of the attack:
1. The attacker sent thousands of uniform `[SYN]` packets to sequential destination ports.
2. Because the host firewall was inactive, the operating system instantly replied with a flood of red `[RST, ACK]` packets for closed ports.
3. For open services (such as SSH Port 22), the attacker received a `[SYN, ACK]` but instantly tore down the session with an abrupt `[RST]` packet before a full 3-way handshake completed, successfully avoiding application-layer logs.

![Wireshark Inbound Traffic Before Firewall](Wireshark_Inbound_Traffic_Before_Firewall_Implementation.png)
![Nmap Port Scan Before Firewall](Nmap_Port_Scan_Before_Firewall_Implementation.png)

---

## 🛡️ Phase 2: Perimeter Hardening & Verification
**Objective:** Eradicate unnecessary network visibility and slow down automated adversary mapping.

### Execution
I deployed host-based perimeter controls via the Uncomplicated Firewall (**UFW**) on the Ubuntu target to modify its default ingress behavior:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
```

### Verification & Analytical Impact
Re-scanning the target showed a drastic shift in network posture:
*   **The Attacker's Perspective:** Previously open and closed port designations disappeared. The port state altered directly to **`filtered`**, indicating that packets were dropping silently into a void.
*   **The Wire Perspective:** In Wireshark, the sea of red `[RST, ACK]` response packets dropped to zero. 

![Wireshark Inbound Traffic After Firewall](Wireshark_Inbound_Traffic_After_Firewall_Implementation.png)
![Nmap Port Scan After Firewall](Nmap_Port_Scan_After_Firewall_Implementation.png)

---

## 🔍 Phase 3: Payload Interception & Triage
**Objective:** Intercept and evaluate data payloads across the wire to discover indicators of compromise (IOCs).

### Technical Approach
To ensure a stable testing pipeline free of external internet CDN redirections, a local payload delivery system was staged. I simulated a plain-text data leakage scenario across the wire from Kali using a lightweight Python-hosted engine. 

By capturing the network stream on Ubuntu and using Wireshark's session analysis tools to **Follow the HTTP Stream**, the encryption blind spot was bypassed entirely. The conversation revealed the full HTTP request headers, the deployment tools, and the raw custom administrative credential text string standing completely uncovered on the wire.

![Payload Output](Payload_Output.png)

---

## 🧠 Core Competencies Demonstrated
*   **Network Auditing:** Packet-level triage, display filtering, and traffic-shape analysis.
*   **Defensive Engineering:** Implementing least-privilege boundary rules via host-based firewalls.
*   **Incident Triage:** Reconstructing raw network communication streams to identify compromised assets and exposed text payloads.
