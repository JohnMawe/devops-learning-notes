## 1. What is SSH?

SSH (Secure Shell) is a protocol for securely accessing and managing a remote system over a network. It encrypts the connection between the client and server.

Common uses include:

- Opening a remote shell.
- Running commands on another computer.
- Transferring files with `scp`.
- Managing servers without being physically present.

---

## 2. How SSH works

An SSH client connects to an SSH server. The server must be running, reachable over the network, and listening on the port the client uses. The server then authenticates the user.

Important terms:

- **Client:** the device initiating the connection, for example Ubuntu.
- **Server:** the device accepting the connection, for example Ubuntu with `openssh-server` or Android with Termux OpenSSH.
- **IP address:** identifies a device/interface on a network.
- **Port:** identifies a service on the target device.
- **Authentication:** checks whether the user is allowed to log in.
- **LAN IP:** a private network address such as `192.168.0.4`; it normally works only on the local network or through a suitable private network/VPN.

---

## 3. Common SSH and SCP commands

```bash
# Connect to a server on the default SSH port (usually 22)
ssh username@host

# Connect when the server uses a non-default port, such as Termux's usual port 8022
ssh -p 8022 username@host

# Copy a local file to the remote user's home directory
scp file.txt username@host:~/

# Copy a file from a remote server to the current local directory
scp username@host:~/file.txt .

# Use a non-default port with SCP (uppercase -P)
scp -P 8022 file.txt username@host:~/

# Show detailed diagnostic output for a connection attempt
ssh -vvv username@host
```

Replace `username` with the account name on the target server and `host` with its IP address or hostname.

Notes:

- `ssh -p` uses a lowercase `p`.
- `scp -P` uses an uppercase `P`.
- Termux's OpenSSH server commonly listens on port 8022.
- Ubuntu's OpenSSH server commonly listens on port 22.
- A file-transfer destination needs a path, such as `username@host:~/`.
- The username on the server may differ from the username on the client.

---

## 4. SSH between Ubuntu and Android Termux

### Android / Termux: prepare the SSH server

Install OpenSSH in Termux if it is not installed:

```bash
pkg update
pkg install openssh
```

Set a Termux account password if you plan to use password authentication:

```bash
passwd
```

Start the SSH server:

```bash
sshd
```

Find the Termux username:

```bash
whoami
```

Check whether SSH is listening:

```bash
ss -ltn
```

Termux commonly uses port 8022.

To find Android's Wi-Fi IP address, you can try:

```bash
ip addr show wlan0
```

Look for an `inet` address. The exact interface name may vary. You can also inspect the device's network details in Android settings.

### Ubuntu: connect to Termux

```bash
ssh -p 8022 TERMUX_USERNAME@ANDROID_IP
```

For example, replace `TERMUX_USERNAME` and `ANDROID_IP` with the actual values from your phone.

### Copy files between devices

Ubuntu to Termux:

```bash
scp -P 8022 file.txt TERMUX_USERNAME@ANDROID_IP:~/
```

Termux to Ubuntu (Ubuntu must be running an SSH server):

```bash
scp file.txt UBUNTU_USERNAME@UBUNTU_IP:~/
```

If connecting from Termux to Ubuntu, Ubuntu usually uses port 22, so `-P 8022` should not be used unless Ubuntu's SSH server has specifically been configured to use that port.

---

## 5. Connecting over different networks with Tailscale

A private LAN address such as `192.168.0.4` is not generally reachable directly from another unrelated network. A private networking tool such as Tailscale can connect devices across networks without the usual need to forward SSH ports on the router.

General workflow:

1. Install Tailscale on both devices using the official installation instructions.
2. Sign both devices into the same Tailscale network/account (tailnet).
3. Start the relevant SSH server on the target device.
4. Find the target's Tailscale IP using the Tailscale app or `tailscale status`.
5. Connect to that Tailscale IP and the correct SSH port.

Examples:

```bash
# On Ubuntu, inspect devices in the tailnet
tailscale status

# Ubuntu to Termux
ssh -p 8022 TERMUX_USERNAME@ANDROID_TAILSCALE_IP

# Termux to Ubuntu (Ubuntu's SSH server normally uses port 22)
ssh UBUNTU_USERNAME@UBUNTU_TAILSCALE_IP
```

Tailscale provides network connectivity; it does not automatically start the target's SSH server or create a shell by itself. It also cannot make a powered-off or sleeping device available if that device is not connected.

Official guides:

- Tailscale Android: <https://tailscale.com/docs/install/android>
- Tailscale Linux: <https://tailscale.com/docs/install/linux>

---

## 6. SSH key management

SSH can authenticate with an account password or with a cryptographic key pair.

### Public key vs private key

Generate an Ed25519 key pair on the client (Ubuntu):

```bash
ssh-keygen -t ed25519 -C "ubuntu-to-termux"
```

The default files are usually:

```text
~/.ssh/id_ed25519       # Private key: keep secret
~/.ssh/id_ed25519.pub   # Public key: can be copied to servers
```

- **Private key:** stays on the client and proves possession of the key. Do not send it to the server, share it, or commit it to Git.
- **Public key:** install this on each server/account where the client should be allowed to log in.
- **`authorized_keys`:** the file on the server containing public keys allowed to authenticate to that account.

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

### Install the public key on Termux

First, make sure the Termux SSH server is running and that you know the Termux username and Android IP.

The following command reads the public key on Ubuntu and appends it to the authorized keys file in the Termux account. Replace the placeholders with your real values:

```bash
cat ~/.ssh/id_ed25519.pub | \
ssh -p 8022 TERMUX_USERNAME@ANDROID_IP \
'mkdir -p ~/.ssh &&
chmod 700 ~/.ssh &&
cat >> ~/.ssh/authorized_keys &&
chmod 600 ~/.ssh/authorized_keys'
```

How it works:

- `cat ~/.ssh/id_ed25519.pub` reads the public key file on Ubuntu.
- `|` (pipe) sends that output as input to the next command.
- `\` at the end of a line continues the command on the next line in Bash.
- `ssh -p 8022 TERMUX_USERNAME@ANDROID_IP` connects to Termux's SSH server.
- The single-quoted part is a command to execute on the remote Termux system.
- `mkdir -p ~/.ssh` creates the SSH directory if needed.
- `chmod 700 ~/.ssh` gives the owner full directory access and denies group/other access.
- `cat >> ~/.ssh/authorized_keys` appends incoming public-key text to the authorized keys file. `>>` appends; `>` would overwrite.
- `&&` runs the next command only if the previous command succeeds.
- `chmod 600 ~/.ssh/authorized_keys` gives the owner read/write access and denies group/other access.

The command may prompt for the Termux account password during setup. The password prompt is for the remote account; the private key is not transferred.

Then test from Ubuntu:

```bash
ssh -p 8022 TERMUX_USERNAME@ANDROID_IP
```

If key authentication works, SSH should not need the Termux account password. If you protected the private key with a passphrase, SSH may still ask for that passphrase.

### Reuse the same key or create different keys?

You can use the same client key pair with multiple servers. Install the same public key in each server account's `authorized_keys` file; keep the private key on the client.

You can also create separate key pairs for different servers or purposes:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_termux
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_newserver
```

Separate keys can make revocation and access management more precise. If a key is compromised, remove its public-key entry from every server where it is trusted and replace it.

### Useful key commands

```bash
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-keygen -lf ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/id_ed25519
chmod 700 ~/.ssh
```

Never share or commit private keys, passwords, API tokens, or other secrets.

---

## 7. Ubuntu SSH server and firewall basics

If Ubuntu is the target server, install and start the SSH server:

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable --now ssh
```

Check service status:

```bash
sudo systemctl status ssh
```

Check listening TCP ports:

```bash
sudo ss -ltnp | grep ':22'
```

Check UFW firewall status:

```bash
sudo ufw status verbose
```

If you intend to accept incoming SSH connections on Ubuntu and UFW is active, allow SSH:

```bash
sudo ufw allow OpenSSH
```

Do not enable a firewall blindly on a remote machine: ensure the necessary access rules are in place first to avoid locking yourself out. If you only use Ubuntu as an SSH client to connect to Termux, you do not need to open an inbound SSH port on Ubuntu.

A firewall rule does not start an SSH service. Likewise, an SSH service listening on a port does not guarantee it is reachable from every network.

---

## 8. Troubleshooting network connections

Troubleshoot from the lower layers upward. Before changing SSH settings, verify that the devices can communicate over the network.

### Useful checks

On the target system, check its IP:

```bash
hostname -I
```

On Ubuntu, inspect the routing table:

```bash
ip route
```

Test reachability (a failed ping does not necessarily mean SSH is unreachable because ICMP may be blocked):

```bash
ping -c 4 TARGET_IP
```

Test the TCP port if netcat is installed:

```bash
nc -vz TARGET_IP 22
```

For Termux, test port 8022 instead if that is the server port:

```bash
nc -vz ANDROID_IP 8022
```

For detailed SSH diagnostics:

```bash
ssh -vvv username@host
```

If Ubuntu is the server, inspect recent SSH service logs:

```bash
sudo journalctl -u ssh --since "30 minutes ago"
```

Client-side verbose output and server-side logs show different parts of the connection; check both when necessary.

### Common errors and what they suggest

- **Connection timed out:** the target may be unreachable, traffic may be dropped, or routing may be wrong.
- **Connection refused:** the target was reachable, but nothing accepted the connection on that port, or the connection was actively rejected.
- **Permission denied (publickey,password):** the server was reached, but authentication failed.
- **Could not resolve hostname:** the hostname may be misspelled or DNS/name resolution failed.
- **No route to host:** there may be a network reachability/routing problem, or a network device/firewall may be actively rejecting traffic.
- **Connection closed:** the connection was closed; use verbose output and server logs to determine where and why.

These messages are clues, not definitive diagnoses. Firewalls and network configurations can produce overlapping symptoms.

---

## 9. Personal troubleshooting case: "No route to host"

### Error encountered

```text
ssh: connect to host 192.168.0.4 port 22: No route to host
```

### Cause

The Android phone's Wi-Fi was turned off, so the phone could not communicate with the Ubuntu computer over the local network.

### Solution

Turned on Wi-Fi and restored network connectivity.

### Lesson learned

A `No route to host` error does not automatically mean SSH is misconfigured. Check the network connection and destination IP address before changing SSH configuration.

### Troubleshooting order

1. Check Wi-Fi or other network connectivity on both devices.
2. Verify the target IP address; it may change when a device reconnects.
3. Confirm both devices are on the same LAN, or use a private network such as Tailscale when they are on different networks.
4. Check network reachability and the correct port.
5. Check that the SSH server is running and listening.
6. Inspect firewall rules.
7. Investigate usernames, keys, passwords, and logs if the network connection reaches the server.

**Key takeaway:** Diagnose the network layer before changing SSH configuration.

---

## 10. How internet scanners find exposed SSH servers

Automated scanners can probe public IP addresses and common ports, including port 22. When a service responds, scanners may identify the protocol and sometimes learn implementation details from the response. Automated login attempts may then try common usernames or passwords.

A private address such as `192.168.0.4` is normally not directly routable from the public internet. However, router port forwarding, cloud firewall settings, or other network configurations can expose a service.

Important distinctions:

- Listening on `0.0.0.0` means listening on all local IPv4 interfaces; it does not by itself prove the service is accessible from the public internet.
- SSH encrypts traffic, but encryption does not protect a server whose account or private key has been compromised.
- Changing the SSH port may reduce background noise, but it is not a replacement for strong authentication and access controls.
- Avoid exposing SSH through router port forwarding for casual practice. Prefer a local network or a private networking solution such as Tailscale.

---

## 11. Practical exercises

- [ ] Connect from Ubuntu to Android Termux on the same Wi-Fi.
- [ ] Transfer a file from Ubuntu to Termux with `scp`.
- [ ] Transfer a file from Termux to Ubuntu with `scp`.
- [ ] Connect across different networks using Tailscale.
- [ ] Generate an Ed25519 key pair and install the public key on Termux.
- [ ] Confirm key authentication works in a new SSH session.
- [ ] Inspect Ubuntu's SSH service status, listening ports, and UFW rules.
- [ ] Practise diagnosing an incorrect IP, wrong port, stopped SSH server, and authentication failure.
- [ ] Review SSH logs on the server side when a connection fails.

---

## 12. What I still need to learn

- SSH key management and revocation.
- Firewall rules and safe firewall configuration.
- Troubleshooting network connections, routing, ports, and service logs.
- SSH hardening and access controls.

---

## 13. SSH Key Selection and Troubleshooting

### The problem I encountered

I tried connecting from Android Termux to my Ubuntu machine using SSH public-key authentication, but SSH kept asking for my Ubuntu account password.

The SSH debug output showed:

```text
Offering public key: ~/.ssh/id_ed25519
Authentications that can continue: publickey,password
Next authentication method: password
```

This revealed that SSH was offering a different key from the one I intended to use.

### Why it happened

SSH looks for standard identity filenames, such as `id_ed25519`, by default. My intended key had a custom filename:

- `id_ed25519.pub`: the default public key SSH was offering.
- `id_ed25519_sA16T.pub`: the public key I wanted to use.

The corresponding private key is used by the SSH client to authenticate. The public key must be present in the Ubuntu user's `~/.ssh/authorized_keys` file.

**Lesson:** Having multiple SSH keys means I may need to specify which private key SSH should use.

### Solution 1: Specify the private key

From Termux, connect using the `-i` option:

```bash
ssh -i ~/.ssh/id_ed25519_sA16T lubiya@192.168.0.4
```

The `-i` option tells SSH which private key to use. Do not specify the `.pub` file here.

### Solution 2: Verify the selected key

Use verbose mode to inspect the authentication process:

```bash
ssh -v -i ~/.ssh/id_ed25519_sA16T lubiya@192.168.0.4
```

Look for:

```text
Offering public key: .../.ssh/id_ed25519_sA16T
```

If the server accepts the key, authentication should succeed without requiring the account password.

### Solution 3: Configure a shortcut

Edit the SSH client configuration in Termux:

```bash
nano ~/.ssh/config
```

Add:

```text
Host ubuntu
    HostName 192.168.0.4
    User lubiya
    IdentityFile ~/.ssh/id_ed25519_sA16T
    IdentitiesOnly yes
```

Secure the configuration file:

```bash
chmod 600 ~/.ssh/config
```

Now connect using the shortcut:

```bash
ssh ubuntu
```

`IdentityFile` specifies the private key to use, while `IdentitiesOnly yes` tells SSH to use the explicitly configured identity rather than offering other available identities unnecessarily.

### Important security notes

- Never share or upload private keys.
- Public keys can be copied to a server's `authorized_keys` file.
- Multiple public keys can be stored in `authorized_keys`, one key per line.
- The public key on the server must match the private key used by the client.
- Use `ssh -v` to troubleshoot authentication and confirm which key is being offered.

### What I learned

1. SSH keys come in public/private pairs.
2. SSH can select a default key instead of the custom key I intended to use.
3. The `-i` option lets me choose a specific private key.
4. The SSH client configuration makes key selection reusable and convenient.
5. Verbose output helps identify authentication problems without guessing or regenerating keys unnecessarily.

---

## 14. Transferring Files with a Specific SSH Key

When I have multiple SSH keys, I can specify which private key to use when transferring files between devices.

### 14.1 Using SCP

The `scp` command securely copies files between systems over SSH.

General syntax:

```bash
scp -i PRIVATE_KEY SOURCE DESTINATION
```

The `-i` option specifies the private key used for authentication.

**Send a file from Termux to Ubuntu**

```bash
scp -i ~/.ssh/id_ed25519_sA16T \
    ~/Documents/notes.md \
    lubiya@192.168.0.4:~/Documents/
```

This sends `notes.md` from Termux to the Ubuntu user's `Documents` directory.

**Download a file from Ubuntu to Termux**

```bash
scp -i ~/.ssh/id_ed25519_sA16T \
    lubiya@192.168.0.4:~/Documents/notes.md \
    ~/Downloads/
```

This copies the remote file to the Termux `Downloads` directory. The destination directory must exist.

**Copy a directory recursively**

```bash
scp -r -i ~/.ssh/id_ed25519_sA16T \
    ~/myproject \
    lubiya@192.168.0.4:~/Documents/
```

The `-r` option copies directories and their contents recursively.

### 14.2 Using SCP with a Non-Default SSH Port

If the Ubuntu SSH server listens on a port other than 22, specify it using uppercase `-P`:

```bash
scp -P 2222 -i ~/.ssh/id_ed25519_sA16T \
    ~/Documents/notes.md \
    lubiya@192.168.0.4:~/Documents/
```

Remember:

- `ssh -p` uses a lowercase `p` for the port.
- `scp -P` uses an uppercase `P` for the port.
- `scp -i` specifies the private key.

### 14.3 Using the SSH Configuration Shortcut

If I configured `~/.ssh/config` with the alias `ubuntu`, I can reuse that configuration when transferring files.

Example configuration:

```text
Host ubuntu
    HostName 192.168.0.4
    User lubiya
    IdentityFile ~/.ssh/id_ed25519_sA16T
    IdentitiesOnly yes
```

Send a file:

```bash
scp ~/Documents/notes.md ubuntu:~/Documents/
```

Download a file:

```bash
scp ubuntu:~/Documents/notes.md ~/Downloads/
```

SCP automatically uses the hostname, username, identity file, and other applicable settings from the SSH configuration.

### 14.4 Using RSYNC with a Specific SSH Key

`rsync` is useful for synchronizing directories and transferring only the changes needed on subsequent runs.

```bash
rsync -av \
    -e "ssh -i ~/.ssh/id_ed25519_sA16T" \
    ~/myproject/ \
    lubiya@192.168.0.4:~/myproject/
```

Options:

- `-a`: archive mode, preserving common file attributes and copying directories recursively.
- `-v`: verbose output.
- `-e`: specifies the remote shell command, in this case SSH with a specific private key.

The trailing slash in `~/myproject/` means to copy the directory's contents into the remote `~/myproject/` directory.

### 14.5 Troubleshooting File Transfers

If SCP or RSYNC asks for a password unexpectedly:

1. Verify that the specified private key exists.
2. Confirm that the matching public key is in the Ubuntu user's `~/.ssh/authorized_keys`.
3. Test authentication first:

   ```bash
   ssh -v -i ~/.ssh/id_ed25519_sA16T lubiya@192.168.0.4
   ```

4. Check that the destination directory exists and that the Ubuntu user has permission to write to it.
5. Remember that SSH authentication and file permissions are separate issues: successful login does not guarantee permission to write to every directory.

### What I learned

- `scp -i` lets me choose a specific private key for file transfers.
- SCP supports uploads, downloads, and recursive directory copies.
- SSH configuration aliases can simplify both remote login and file transfers.
- RSYNC is useful for repeated synchronization of project files.

---

## 15. Useful links

- OpenSSH manual page: <https://man.openbsd.org/ssh>
- Ubuntu OpenSSH Server guide: <https://documentation.ubuntu.com/server/how-to/security/openssh-server/>
- Tailscale Android: <https://tailscale.com/docs/install/android>
- Tailscale Linux: <https://tailscale.com/docs/install/linux>
