# Kioptrix 3: Remote Code Execution and MySQL Data Extraction

**Objective:** To perform RCE on the Kioptrix 3 machine and grab the user account details stored on the MYSQL server.

---

## Stage 1: Identifying the Target Machine and Its Open Ports

### Step 1: Establish the IP Address for the System

```bash
sudo netdiscover -P -r <IP>
```

We have identified the IP, now we proceed to look up its ports.

### Step 2: Using the nmap Tool to Scan the Ports

```bash
nmap -A -T4 -p- <Target_IP>
```

**Findings:**
- Ports 22 and 80 are currently open
- Port 80 is running an Apache server
- We attempt to access the system on Port 80 through the browser
- We are prompted for account details to login

### Step 3: Running a nikto Scan on Port 80

```bash
nikto -port 80 -host <Target_IP>
```

**Findings:**
- Nikto pulls up a directory `/phpmyadmin`
- This is used to manage a MySQL database, confirming access to user account details is possible
- We attempt to access the directory: `http://<Target_IP>/phpmyadmin`

---

## Stage 2: Identifying Vulnerabilities to Gain RCE into the System

### Step 1: Surveying the website to gain additional information.

On the Login page, we attempt SQL injection, however, this is not successful, hence we must find more information to break in through other means.

**Findings:**
- The security provided for the site is through a vendor called **LotusCMS**
- We look up LotusCMS on searchsploit to see if there are any vulnerabilities available
- The search shows us a few exploits we could use
- We load up Metasploit and run the `LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)` exploit
- The exploit executed successfully, but failed to open up a session.

---

## Stage 3: Using External Resources on GitHub to Find Another Version of the Exploit

**Findings:**
- A LotusCMS-Exploit was obtained on GitHub
- **Link:** [https://github.com/Hood3dRob1n/LotusCMS-Exploit/tree/master](https://github.com/Hood3dRob1n/LotusCMS-Exploit/tree/master)
- We set up the exploit on our machine
- We run the exploit and specify the Target_IP it is to work with

```bash
./filename.sh <Target_IP> /
```

- We must open up a listening port on our device as the script will attempt to gain access to the root of the website source code

```bash
nc -lnvp 4444
```

**Findings:**
- We have achieved RCE to the website source code
- A list of files come up with the `ls` command
- Upon going through the files, we find the file `config.php` located in the Gallery directory, which contains the admin username and password to access the phpmyadmin MySQL server

---

## Stage 4: Grabbing the User Account Details for Privilege Users

**Findings:**
- From the admin account details obtained from the `config.php` file, we login into the MYSQL server.
- By navigating the database, we find the admin password details located in the `gallarific_users` column.
- Additionally, we have also obtained the Dev account details for two users in the table

---

## Conclusion

- We have successfully performed RCE to the system initially through the server running on Port 80
- Once complete, we used the account details to gain access to the MySQL server
- The MySQL server contained the passwords for both admin and dev accounts in the system

---

## References

- [https://infosecwriteups.com/vulnhub-kioptrix-level-3-1-2-3-a7ff58cbfb8f](https://infosecwriteups.com/vulnhub-kioptrix-level-3-1-2-3-a7ff58cbfb8f)
- [https://systemweakness.com/kioptrix-level-3-write-up-1655c647503d](https://systemweakness.com/kioptrix-level-3-write-up-1655c647503d)
- **Exploit:** [https://github.com/Hood3dRob1n/LotusCMS-Exploit/tree/master](https://github.com/Hood3dRob1n/LotusCMS-Exploit/tree/master)
