## Cybersecurity Threat Detector

An interactive, self-contained Network Security Monitoring (NSM) lab designed to simulate attacks and detect network anomalies using Suricata IDS. This project demonstrates the setup of an isolated virtual network, traffic sniffing via a dedicated network sensor, and writing custom signature rules to identify adversarial activity.

---

## 🛠️ Tech Stack & Key Concepts
- **Intrusion Detection System (IDS):** Suricata
- **Operating Systems:** Ubuntu Server (IDS Sensor), Kali Linux (Attacker VM), Metasploitable 2 (Vulnerable Target VM)
- **Virtualization:** VirtualBox (Isolated Host-Only Network)
- **Log Analysis:** fast.log, eve.json, jq
- **Tools Used:** Nmap, Git, PowerShell / Bash

---

## 🗺️ Lab Network Architecture
The environment consists of three virtual machines communicating over an isolated VirtualBox host-only adapter network (`192.168.56.0/24`). This design ensures that malicious traffic remains contained and cannot leak onto the production network or the internet.

1. **Attacker Node (Kali Linux):** Runs active reconnaissance and attack simulations.
2. **IDS Sensor Node (Ubuntu Server):** Configured in promiscuous mode to capture network packets on its interface using Suricata.
3. **Target Node (Metasploitable 2):** An intentionally vulnerable server hosting multiple protocols (FTP, SSH, HTTP, SMB).

---

## 🚀 Setup & Configuration

### Phase 1: Virtual Networking & Interface Setup
- Configured a VirtualBox **Host-Only Network** adapter (`192.168.56.X`).
- Assigned static/dynamic IPs inside the private range:
  - **Kali Linux IP:** `[192.168.56.102]`
  - **Ubuntu IDS Sensor IP:** `[192.168.56.103]`
  - **Metasploitable 2 IP:** `[192.168.56.101]`

### Phase 2: Installing and Tuning Suricata
1. Installed Suricata on the Ubuntu Sensor Node:
   ```bash
   sudo apt update
   sudo apt install suricata -y
   ```
2. Configured the global rule file `/etc/suricata/suricata.yaml`:
   - Set the `HOME_NET` variable to cover our local lab network:
     ```yaml
     HOME_NET: "[192.168.56.0/24]"
     ```
   - Configured the `af-packet` interface monitoring setting to target the active host-only interface (e.g., `enp0s3` / `enp0s8`).
3. Enabled and loaded the **ET Open (Emerging Threats)** ruleset.

---

## ⚡ Attack Simulation & Detection Validation

### 1. Nmap SYN Port Scan (Reconnaissance)
From the Kali Linux attacker terminal, a TCP SYN port scan with version detection was executed against the Metasploitable 2 server to generate reconnaissance traffic:

```bash
sudo nmap -sS -sV [192.168.56.101]
```

#### Results & Detection:
Suricata monitored the passive traffic crossing the host-only adapter and instantly triggered security alerts. 

**Suricata alert in `fast.log`:**
```text
[Insert a few sample lines from your /var/log/suricata/fast.log showing ET SCAN alerts here]
```

**Structured Event Logging in `eve.json` (filtered via `jq`):**
```json
[Insert a snippet of the JSON alert block found in your eve.json showing the alert signature metadata]
```

*📂 [Optional: Place a screenshot of your terminal showing the fast.log alerts here!]*
`![Nmap Alert Logs](images/nmap_alerts.png)`

---

## ✍️ Writing & Testing a Custom Detection Rule
To gain a deep understanding of Suricata's detection engine syntax, a custom signature rule was written from scratch.

### The Objective
Alert on any inbound ICMP echo request (ping) coming from any network and directed to any host on our `$HOME_NET`.

### The Custom Rule
Added to `/var/lib/suricata/rules/local.rules`:
```text
alert icmp any any -> $HOME_NET any (msg:"LAB - ICMP Ping Detected"; itype:8; sid:10000001; rev:1;)
```

### Validation Steps
1. Confirmed Suricata's configuration was valid:
   ```bash
   sudo suricata -T -c /etc/suricata/suricata.yaml
   ```
2. Ran Suricata in the foreground in live IDS mode:
   ```bash
   sudo suricata -c /etc/suricata/suricata.yaml -i [INTERFACE]
   ```
3. Executed 4 ping requests from the Kali VM to the IDS Sensor:
   ```bash
   ping -c 4 [SENSOR_IP]
   ```

### Custom Rule Proof
The custom rule successfully detected the ICMP packets. Looking at `/var/log/suricata/fast.log`:

```text
[Insert your "LAB - ICMP Ping Detected" alert output here]
```

*📂 [Optional: Place a screenshot of your custom local.rules file or custom ping alert here!]*
`![Custom Ping Alert](images/custom_alert_proof.png)`

---

## 🧠 Key Takeaways
- **Network Containment:** Learned how to safely configure host-only VM adapters to prevent security testing traffic from escaping into production networks.
- **IDS Configuration:** Configured passive scanning interfaces and correctly scoped monitored networks (`HOME_NET`).
- **Signature Anatomy:** Mastered Suricata rule components including headers, rules action (alert), direction arrows, metadata messages, unique signature IDs (`sid`), and payload options (`itype`).
- **Log Parsing:** Gained experience analyzing both human-readable telemetry (`fast.log`) and structured JSON telemetry (`eve.json`) used by modern Security Operations Centers (SOCs) and SIEM platforms.
