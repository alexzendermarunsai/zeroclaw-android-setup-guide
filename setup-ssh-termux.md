# ZeroClaw Guide: SSH Access for Termux on Android

Back to the main setup guide: `README.md`

This guide covers OpenSSH setup for Termux on Android without root, so you can connect to your phone remotely during ZeroClaw setup and maintenance.

[wiki.termux](https://wiki.termux.com/wiki/Remote_Access)

## Step 1: Prerequisites

- Install Termux from F‑Droid or GitHub (not Google Play).  
- Allow Termux to access storage when prompted, or run `termux-setup-storage` to enable it. [github](https://github.com/rishabhrpg/sshd-in-termux)

Open Termux and run:

```bash
pkg update && pkg upgrade
pkg install openssh
```

This installs the OpenSSH client and server (`sshd`). [dev](https://dev.to/terminaltools/how-to-set-up-ssh-server-on-termux-remote-shell-access-made-easy-5ci7)

## Step 2: Set Up Your User Password

In Termux:

```bash
whoami
passwd
```

- `whoami` prints your username (usually `u0_a...` or `app`).  
- `passwd` lets you set a login password for SSH. Don’t reuse a critical password; keep it local‑only. [samutz](https://samutz.com/docs/books/tech/page/setting-up-ssh-on-termux)

## Step 3: Start the SSH Server

Still in Termux:

```bash
sshd
```

OpenSSH listens on port `8022` by default on address `0.0.0.0` (no root required). [youtube](https://www.youtube.com/watch?v=52Tf0r_jqXE)

Optionally check if it’s running:

```bash
sshd -T | grep port
```

You should see `port 8022`. [wiki.termux](https://wiki.termux.com/wiki/Remote_Access)

## Step 4: Find Your Android IP

On the same network (WiFi or hotspot), get your phone’s local IP:

```bash
ip a show wlan0 | grep inet
```

or (older devices):

```bash
ifconfig wlan0 | grep 'inet '
```

Note the `inet` address (e.g., `192.168.1.123`). [youtube](https://www.youtube.com/watch?v=AG_7qgAfPmY)

## Step 5: Connect From Another Machine

From your PC or laptop, use your SSH client:

- **Linux/macOS terminal:**

  ```bash
  ssh -p 8022 u0_aN@192.168.1.123
  ```

- **Windows (PuTTY):**  
  - Host: your phone’s IP  
  - Port: `8022`  
  - Connection type: SSH  
  - Login: the username from `whoami`  
  - Password: the one you set with `passwd` [youtube](https://www.youtube.com/watch?v=d2oqfLiORlQ)

## Step 6: Optional Auto-Start On Termux Launch

If you want Termux to start SSHd automatically on launch:

1. Install `termux-services` (if not already present):

   ```bash
   pkg install termux-services
   ```

2. Start SSH as a service:

   ```bash
   sv-enable sshd
   ```

On the next Termux start, `sshd` will auto‑start on port `8022`. [github](https://github.com/rishabhrpg/sshd-in-termux)

## Step 7: Harden With SSH Keys

1. On your PC, generate a key pair:

   ```bash
   ssh-keygen -t ed25519 -C "termux@$(hostname)"
   ```

2. On Android (in Termux), prepare the directory:

   ```bash
   mkdir -p ~/.ssh && chmod 700 ~/.ssh
   touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
   ```

3. Copy the public key from your PC into `~/.ssh/authorized_keys` (e.g., via `cat ~/.ssh/id_ed25519.pub | ssh -p 8022 u0_aN@192.168.1.123 'cat >> ~/.ssh/authorized_keys'`).  [wiki.termux](https://wiki.termux.com/wiki/Remote_Access)  

4. Optionally disable password auth (more secure, but only if you are sure you can still access Termux):

   Edit `$PREFIX/etc/ssh/sshd_config`:

   ```bash
   nano $PREFIX/etc/ssh/sshd_config
   ```

   Set:

   ```text
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```

   Then restart the daemon:

   ```bash
   pkill sshd
   sshd
   ```

 [samutz](https://samutz.com/docs/books/tech/page/setting-up-ssh-on-termux)

## Notes

- **Port 8022 only:** Termux uses `8022` by design; you cannot bind `22` without root. [reddit](https://www.reddit.com/r/termux/comments/e4yt6r/change_ssh_port_for_openssh/)
- **Security:** Keep the SSH user local, avoid exposing it to the internet unless behind a VPN, and prefer key‑based auth. [dev](https://dev.to/terminaltools/how-to-set-up-ssh-server-on-termux-remote-shell-access-made-easy-5ci7)
