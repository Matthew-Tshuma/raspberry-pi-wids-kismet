# Raspberry Pi Wireless Intrusion Detection System (WIDS) using Kismet

## Project Overview

Wireless networks are susceptible to attacks such as de‑authentication floods, rogue access points, MAC spoofing, probe floods and beacon spoofing.  To explore these threats and gain hands‑on security experience, I designed and deployed a **Wireless Intrusion Detection System (WIDS)** on a Raspberry Pi using the open source tool **Kismet**.  The system monitors 802.11 traffic in real‑time, detects suspicious behaviour and logs alerts for analysis.

This project demonstrates skills in wireless networking, intrusion detection, Linux configuration, Wi‑Fi security and packet capture.  It forms part of my cyber security portfolio and prepares me for SOC and analyst roles.

## Architecture

```
+-----------------------+
| Raspberry Pi 4        |
| Kali Linux            |
| Kismet IDS            |
+----------+------------+
           |
     wlan1 (monitor mode)
           |
   Captures 802.11 frames
           |
+----------v------------+
| Kismet detection engine |
|  – Rogue AP detection  |
|  – Deauth attack alerts |
|  – Probe flood alerts  |
|  – MAC spoofing checks |
|  – Signal anomaly checks |
+----------+-------------+
           |
   Logs + alerts
           |
+----------v-------------+
| Analysis workstation   |
| (browser on laptop)    |
+------------------------+
```

## Hardware & Software

| Category      | Details                                                                    |
|--------------|----------------------------------------------------------------------------|
| Hardware     | Raspberry Pi 4 (4 GB RAM); BrosTrend AC3L USB Wi‑Fi adapter (RTL88x2BU)    |
| Software     | Kali Linux ARM; Kismet; airodump‑ng/aireplay‑ng (for validation); Wireshark |
| Network mode | `wlan0` – managed mode for normal internet; `wlan1` – monitor mode for IDS |

## System Setup

1. **Enable monitor mode on `wlan1`:**

   ```bash
   sudo nmcli dev set wlan1 managed no
   sudo ip link set wlan1 down
   sudo iw dev wlan1 set type monitor
   sudo ip link set wlan1 up
   iwconfig   # verify Mode:Monitor
   ```

2. **Install and start Kismet:**

   ```bash
   sudo apt update
   sudo apt install -y kismet
   sudo pkill -f kismet
   sudo rm -rf /root/.kismet
   sudo kismet   # keep this terminal open
   ```

3. **Access the web UI:** Determine your Pi’s IP (e.g. `192.168.0.35`) then browse to `http://<PiIP>:2501` from another device.  When prompted, create an admin username and password.

4. **Add `wlan1` as a data source:**

   - In the Kismet dashboard, go to **Data Sources** → **Add Source**.
   - Select **Linux Wi‑Fi**, choose interface **wlan1**, leave channel hopping enabled.
   - Kismet begins scanning and displays nearby access points and clients.

5. **Generate a test alert (optional):**  From another terminal run a deauthentication attack against your own router (legally!):

   ```bash
   sudo airodump-ng -c 36 --bssid <ROUTER_MAC> -w capture wlan1   # lock channel
   sudo aireplay-ng --deauth 10 -a <ROUTER_MAC> wlan1             # send deauth frames
   ```

   Kismet should trigger a **DEAUTH_FLOOD** alert.  Alerts and packets are stored under `~/.kismet/logs/`.

## Example Attacks Detected

- **Deauthentication floods:** repeated deauth frames cause clients to drop; Kismet logs an alert with time, MAC and channel.
- **Rogue access points:** a second BSSID broadcasting the same SSID; flagged as potential evil twin.
- **MAC spoofing:** identical MAC addresses with different radio signatures or channels.
- **Probe floods:** excessive probe requests from a single device; may indicate scanning tools.
- **Beacon anomalies:** irregular beacon intervals or abnormal capabilities.

## Logs and Analysis

Kismet stores its artefacts in `~/.kismet/logs/`.  Important files include:

- `kismet_alerts.json` – JSON formatted alerts (deauth floods, rogue APs, etc.).
- `kismet-YYYYMMDD-*.pcap` – packet captures for offline analysis in Wireshark.
- `kismet-YYYYMMDD-*.netxml` – XML with observed devices and networks.

To watch alerts live in the CLI:

```bash
sudo tail -f ~/.kismet/logs/kismet_alerts.json
```

## What I Learned

- How 802.11 frames are captured in monitor mode and the difference between managed and monitor modes.
- Why raw radiotap headers require special handling (many tools cannot parse them).
- How to identify deauthentication attacks, rogue APs and probe floods by analysing layer 2 traffic.
- The limitations of Suricata on Wi‑Fi and why Kismet is more suitable for wireless IDS.
- Linux networking commands (iw, ip, nmcli) and configuration of multiple interfaces.
- Best practice for isolating an IDS interface from NetworkManager using unmanaged settings.
- Logging and analysing PCAP files and JSON alerts.

## Professional Project Summary

I designed and implemented a **wireless intrusion detection system** using Kismet on a Raspberry Pi.  The project shows my ability to configure monitor‑mode wireless adapters, deploy security tools on constrained hardware, detect real‑world Wi‑Fi attacks and document the process.  Alerts for deauthentication floods, rogue APs and probing behaviour were validated using controlled attacks on my own network.  Logs and packet captures were collected for further analysis.  This project demonstrates my readiness for SOC roles and my hands‑on experience with network security, Linux administration and wireless threat detection.

## Future Improvements

- Integrate Kismet logs with a central SIEM (such as ELK or Splunk) for correlation with other network events.
- Add custom detection signatures in Kismet for specific attacks (e.g. WPS brute force, beacon stuffing).
- Develop dashboards using Grafana or EveBox for visualising alerts and trends.
- Expand the platform to monitor multiple channels simultaneously with additional adapters.
- Automate periodic updates and rule tuning via scripts.

