# SSH Remote Access

## 1. What is SSH?
SSH is a protocol for securely accessing a remote system over a network.

## 2. How does it work?
An SSH client connects to an SSH server. The connection is encrypted, and the server authenticates the user.

## 3. Commands I learned

```bash
ssh username@host
ssh -p 8022 username@host
scp file.txt username@host:~/
```
- **`ssh username@host`** use this command to connect client to server. Replace username with server name and host with server ip address.

- **`ssh -p 8022 username@host`** use this command if server android termux or if the server sshd is not on default port 22.

- **`scp file.txt username@host`** use this to send files.

## 4. Practical exercise
Connect from Ubuntu to Android using Termux, first on the same Wi-Fi and then through Tailscale.

## 5. Problems I encountered
Connection closed during an SSH/SCP attempt. Investigate the server, destination port, username, and authentication logs.

## 6. What I understand now
SSH provides remote access, but IP addressing and network routing determine whether the server is reachable.

## 8. What I still need to learn
SSH key management, firewall rules, and troubleshooting network connections.

## 9. Usefull links
**OpenSSH manual page:** https://man.openbsd.org/ssh
**Ubuntu OpenSSH Server guide:** https://documentation.ubuntu.com/server/how-to/security/openssh-server/