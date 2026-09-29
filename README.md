# Cybersecurity-Threat-Detector

Isolated 3-VM lab for practicing attack generation and detection. Kali Linux generates traffic, Suricata IDS monitors it, and Metasploitable 2 serves as the target, all on a host-only network with no internet exposure.

## Architecture

![Network diagram](docs/network-diagram.png)

| VM | Role | IP |
|----|------|----|
| Kali Linux | Attacker | 192.168.56.<x> |
| Ubuntu Server + Suricata | Network sensor | 192.168.56.<x> |
| Metasploitable 2 | Target | 192.168.56.<x> |

- **Network:** VirtualBox host-only, 192.168.56.0/24
- **Ruleset:** ET Open (49,000+ rules)

## Setup

1. Create the host-only network in VirtualBox
2. Deploy the three VMs and assign IPs
3. Install Suricata on Ubuntu Server and set the monitored interface in `suricata.yaml`
4. Update rules with `suricata-update`
5. Confirm Suricata is running in monitoring mode

## Validation: Nmap Scan

- Command run from Kali: `nmap <flags> 192.168.56.<x>`
- Alerts observed in `fast.log` and `eve.json`

![fast.log alerts](evidence/fast-log-nmap.png)

## Custom Rule: ICMP Flood Detection

```
<your threshold rule here>
```

- **Logic:** <threshold, tracking, count, seconds>
- **Test:** <command used to generate flood from Kali>
- **Result:** <alerts observed>

## Lessons Learned

- <e.g., tuning thresholds to avoid false positives>
- <e.g., interface/promiscuous mode issues>

## Next Steps

- <e.g., forward eve.json to a SIEM>

## Disclaimer

Built in an isolated environment for educational purposes only.
