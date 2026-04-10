# 🔐 SSH Key Setup for `hngdevops` User

## 🎯 Objective

Enable secure SSH access for the `hngdevops` user using public key authentication on a hardened Linux server where password login is disabled.

---

## 📌 Prerequisites

* Access to the server via an existing user (e.g., `ubuntu`)
* Local machine has an SSH key (`~/.ssh/id_rsa.pub`)
* `hngdevops` user already exists

---

## 🧩 Task Breakdown

### 1. Connect to Server

```bash
ssh ubuntu@<PUBLIC_IP>
```

---

### 2. Switch to Target User

```bash
sudo su - hngdevops
```

---

### 3. Prepare SSH Directory

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

---

### 4. Add Public Key

On your **local machine**:

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the output.

On the **server**:

```bash
nano ~/.ssh/authorized_keys
```

Paste the key, save, and exit.

---

### 5. Set Correct Permissions

```bash
chmod 600 ~/.ssh/authorized_keys
```

---

### 6. Fix Ownership

Run from another user (e.g., ubuntu):

```bash
sudo chown -R hngdevops:hngdevops /home/hngdevops/.ssh
```

---

### 7. Verify SSH Configuration

```bash
sudo nano /etc/ssh/sshd_config
```

Ensure the following are set:

```
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication no
```

---

### 8. Restart SSH Service

```bash
sudo systemctl restart ssh
```

---

### 9. Test Access (Local Machine)

```bash
ssh hngdevops@<PUBLIC_IP>
```

---

## ✅ Expected Outcome

* SSH login works using key only
* No password prompt
* Server remains hardened

---

## ⚠️ Notes

* Keep your current session open while testing
* Do NOT log out until SSH access is confirmed working
* If access fails, recheck permissions and key placement

---
