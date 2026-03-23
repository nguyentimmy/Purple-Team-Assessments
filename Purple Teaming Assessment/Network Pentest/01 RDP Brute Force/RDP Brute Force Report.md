# RDP Brute Force Lab 

**Purpose:**
This lab demonstrates end-to-end detection of an RDP brute-force attack in an Azure environment using native Microsoft Sentinel analytics rules. It combines red-team simulation (nmap + Hydra) with blue-team detection based on Windows Event ID 4625, showing how to turn raw failed logons into high-severity incidents with automated entity mapping.

**Technologies:** Microsoft Sentinel, Windows Server, Nmap, Hydra, KQL




***

# Phase 1 

![alt text](https://github.com/nguyentimmy/azure-lab/blob/main/Purple%20Teaming%20Assessment/Network%20Pentest/01%20RDP%20Brute%20Force/Pictures/1.%20rdp%20bf.png)

- **Bullet 1:** As you can see on the result on the nmap scan, RDP is open on this Windows server. If hackers see RDP or SSH open, they will try to exploit these protocol to gain access. Depending on the scenario, it's best not to use these NMAP flags. This scan is very loud and can get detected by most modern security solutions *(EX: IDS, IPS, SIEM, EDR, XDR)*. But for the sake of demonstration purposes, I will run the scans anyway to see what ports are open on the server.
  ```
  nmap -sC -sV -vvv -T5 -oN report.txt 10.0.0.4
  ```

- **Bullet 2:** Once RDP is discovered, I performed a brute force attack on the open RDP using Hydra. This should also generate the traffic and alerts I set up within Sentinel.
   ```
   hydra -t 1 -V -l winserver -P rockyou.txt rdp://10.0.0.4
   ```
   

***


# Phase 2

Next, it's time to see if we can detect the traffic and alerts on Microsoft Sentinel. I created an alert that will generate an incident if it detects multiple failed sign-ins based on **Event Code 4625.** The event code documents every failed login attempts on a local machine.

![alt text](https://github.com/nguyentimmy/azure-lab/blob/main/Purple%20Teaming%20Assessment/Network%20Pentest/01%20RDP%20Brute%20Force/Pictures/2.%20rdp%20bf.png)

- **Bullet 1:** The alerts and configuration I set up picked up the "possible" brute forcing attack.
- **Bullet 2:** I've set the alert to auto-assign the incident to myself, update the severity to high, and the map entities. 


***


# Phase 3
Sentinel successfully detected the brute-force activity and created a high-severity incident in real time. The rule correctly correlated the flood of failed RDP logons and surfaced it as a single actionable incident. Successfully create a analytic rule that detects brute forcing.

![alt text](https://github.com/nguyentimmy/azure-lab/blob/main/Purple%20Teaming%20Assessment/Network%20Pentest/01%20RDP%20Brute%20Force/Pictures/3.%20rdp%20bf.png)

![alt text](https://github.com/nguyentimmy/azure-lab/blob/main/Purple%20Teaming%20Assessment/Network%20Pentest/01%20RDP%20Brute%20Force/Pictures/4.%20rdp%20bf.png)

***

