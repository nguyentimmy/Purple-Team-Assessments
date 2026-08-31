# 🟣 Purple Team Assessments

Hands-on purple-team lab projects from my time as a cybersecurity teaching assistant at edX. Each project walks the **full attack-then-detect loop** — running Red Team offense against a controlled target, then switching to the Blue Team seat to hunt the same activity in a SIEM. The goal: prove detection actually catches what the attack did.

---

## 🧪 Projects

### 🎯 [Red Vs Blue Team Project](./Red%20Vs%20Blue%20Team%20Project)

A three-part purple-team exercise against a vulnerable Linux web server.

- **🔴 Part 1 — Red Team:** nmap recon → hydra brute-force → password cracking → WebDAV → PHP reverse shell → flag capture
- **🔵 Part 2 — Blue Team:** hunt the attack in Kibana using Packetbeat — offensive traffic, brute-force patterns, WebDAV connections, and meterpreter shell activity
- **📝 Part 3 — Reporting:** compile findings into a report with hardening recommendations

**Stack:** Kali · Nmap · Hydra · Metasploit / msfvenom · WebDAV · ELK Stack · Kibana · Packetbeat

### 🚩 [Security Operations CTF Project](./Security%20Operations%20CTF%20Project)

A two-part CTF that pairs SIEM alert configuration with a WordPress attack chain and Wireshark forensics.

- **🔵 Part 1 — Alerts + Target 1 CTF:** stand up Kibana Watcher alerts, then attack a WordPress target (nmap → WPScan → SSH → MySQL dump → John the Ripper → privilege escalation) and capture four flags
- **🌊 Part 2 — Wireshark Strikes Back:** packet-level forensics on two infection scenarios — Time Thieves (frank-n-ted.com C2, `june11.dll` trojan) and Vulnerable Windows Machines (Rotterdam-PC infection)

**Stack:** Kali · Nmap · WPScan · MySQL · John the Ripper · Kibana / ELK · Wireshark · VirusTotal

---

## 🎯 The Purple-Team Loop

Every project follows the same principle:

1. **Attack** — run the offensive chain in a controlled lab
2. **Detect** — verify the SIEM / logs actually captured the activity
3. **Analyze** — identify what stands out and what alarms would catch it earlier
4. **Report** — document findings and recommend hardening

---

## ⚠️ Disclaimer

All activity was performed in isolated lab environments I owned, for educational and detection-validation purposes only. Nothing here targets systems I did not control.
