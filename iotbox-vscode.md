# Odoo IoT Box Development Workflow (SSHFS + VS Code)

This guide documents the complete setup for developing on the official Odoo IoT Box using **SSHFS** and **VS Code**.

## Why this approach?

The official **Odoo IoT Box** runs:

- **CPU:** `aarch64` (64-bit)
- **Userland:** `armhf` (32-bit)

Modern **VS Code Remote SSH** no longer supports **Linux ARM32 (armhf)**, so the VS Code server cannot be installed.

**SSHFS** is the recommended workaround because it mounts the IoT Box filesystem over SSH, allowing you to edit files locally with VS Code.

---

# One-Time Setup

## 1. Install SSHFS on your laptop

```bash
sudo apt update
sudo apt install sshfs
```

---

## 2. Create the mount directory

```bash
mkdir -p ~/iotbox
sudo chown $USER:$USER ~/iotbox
```

---

## 3. Set a root password on the IoT Box (Optional but Recommended)

Become root:

```bash
sudo -i
```

Set the password:

```bash
passwd root
```

Enter and confirm the new password.

---

## 4. Enable Root SSH Login (Optional)

Edit the SSH configuration:

```bash
nano /etc/ssh/sshd_config
```

Ensure these settings exist:

```text
PermitRootLogin yes
PasswordAuthentication yes
```

Restart the SSH service:

```bash
systemctl restart ssh
```

Test the connection:

```bash
ssh root@192.168.106.16
```

---

# Daily Workflow

## 1. Mount the IoT Box

Using the **root** user:

```bash
sshfs -o idmap=user root@192.168.106.16:/ ~/iotbox
```

Or using the **pi** user:

```bash
sshfs -o idmap=user pi@192.168.106.16:/ ~/iotbox
```

---

## 2. Open the Mounted Filesystem in VS Code

```bash
code ~/iotbox
```

Example project location:

```text
~/iotbox/home/pi/odoo/addons/iot_drivers/
```

Any changes saved in VS Code are written directly to the IoT Box.

---

## 3. Connect to the IoT Box

As **root**:

```bash
ssh root@192.168.106.16
```

Or connect as **pi** and switch to root:

```bash
ssh pi@192.168.106.16
sudo -i
```

Restart services whenever required.

---

## 4. Unmount When Finished

```bash
fusermount -uz ~/iotbox
```

---

# Useful Commands

## Check if the filesystem is mounted

```bash
mount | grep iotbox
```

or

```bash
df -h | grep iotbox
```

---

## Become root

```bash
sudo -i
```

---

## Return to the pi user

```bash
exit
```

or

```bash
su - pi
```

---

## Find processes using the mount

```bash
lsof +D ~/iotbox
```

or

```bash
fuser -vm ~/iotbox
```

---

## Force a lazy unmount

```bash
fusermount -uz ~/iotbox
```

---

# Troubleshooting

## VS Code Remote SSH asks for the password repeatedly

### Cause

VS Code Remote SSH does **not** support **ARM32 (armhf)** on the official Odoo IoT Box.

### Solution

Use **SSHFS** instead of **VS Code Remote SSH**.

---

## Saving files in VS Code fails

Verify that:

- The SSHFS mount is active.
- The file is opened under `~/iotbox/...`.
- The remote user has write permissions.

---

## "Device or resource busy" while unmounting

Close any applications using the mounted directory, including:

- VS Code
- Terminal sessions
- File managers

Then run:

```bash
fusermount -uz ~/iotbox
```