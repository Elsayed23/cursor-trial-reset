# Cursor Reset & Reinstallation Script (Linux)

This script helps you completely remove, reset, and reinstall the **Cursor editor** on Linux.  
It also resolves the common issue:

> "Too many free trial accounts used on this machine. Please upgrade to pro."

It works by resetting key system identifiers such as the **machine ID** and **MAC address**, giving your system a fresh state so that Cursor can be reinstalled without conflict.

---

## Overview

The script performs the following actions in order:

1. Removes all Cursor-related files, configs, and binaries from both user and system paths.  
2. Resets the machine’s unique identifiers (`/etc/machine-id` and D-Bus UUID).  
3. Installs **macchanger** (if not already installed) and randomizes your primary network MAC address.  
4. Reinstalls the Cursor **AppImage** and links it globally so you can launch it easily.  
5. Verifies everything is cleaned and ready to run.

---

## Requirements

Before running this script, make sure:

- You are using a **Debian/Ubuntu-based** Linux distribution.  
- You have **sudo privileges**.  
- The **Cursor AppImage** file (e.g. `Cursor-1.5.9-x86_64.AppImage`) is already downloaded and located at:

  ```
  ~/Downloads/Cursor-1.5.9-x86_64.AppImage
  ```

  > ⚠️ If you placed the file in a different directory, update this line in the script:
  >
  > ```bash
  > sudo cp ~/Downloads/Cursor-1.5.9-x86_64.AppImage /opt/Cursor/Cursor.AppImage
  > ```
  > to match your actual path.

---

## Usage

1. Open your terminal and create the script:
   ```bash
   nano cursor-reset.sh
   ```

2. Paste the full script (below) and save it with `Ctrl + O`, then exit with `Ctrl + X`.

3. Make it executable:
   ```bash
   chmod +x cursor-reset.sh
   ```

4. Run it with admin privileges:
   ```bash
   ./cursor-reset.sh
   ```

5. After completion, start Cursor:
   ```bash
   cursor --no-sandbox
   ```

---

## Step-by-Step Explanation

| Step | Purpose |
|------|----------|
| **Cleanup** | Removes all Cursor configuration folders and installation files |
| **Reset Machine ID** | Deletes and regenerates the Linux system’s unique identifier |
| **Reset D-Bus UUID** | Ensures your D-Bus session uses a new unique ID |
| **Change MAC Address** | Randomizes your primary network interface’s MAC address |
| **Reinstall Cursor** | Installs the AppImage into `/opt/Cursor` and links it to `/usr/local/bin` |
| **Verification** | Prints success messages and confirms readiness to launch Cursor |

---

## Full Script

```bash
#!/bin/bash

echo "[+] Removing Cursor config and installation..."
sudo rm -rf ~/.config/Cursor
sudo rm -rf /root/.config/Cursor
sudo rm -rf /opt/Cursor
sudo rm -f /usr/local/bin/cursor

echo "[+] Resetting machine ID..."
sudo rm -f /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
sudo dbus-uuidgen --ensure=/etc/machine-id
sudo dbus-uuidgen --ensure

echo "[+] Installing macchanger if not present..."
sudo apt-get install -y macchanger

echo "[+] Detecting network interfaces..."
iface=$(ip link | grep -Eo '^[0-9]+: [^:]*' | grep -v lo | awk '{print $2}' | head -n 1)

if [ -z "$iface" ]; then
  echo "[!] No network interface found."
  exit 1
fi

echo "[+] Changing MAC address for $iface..."
sudo ip link set $iface down
sudo macchanger -r $iface
sudo ip link set $iface up

echo "[✓] System identifiers reset completed."

sudo mkdir -p /opt/Cursor
echo "[+] Copying Cursor AppImage from Downloads..."

# Detect latest AppImage automatically if filename differs
appimage_path=$(ls ~/Downloads/Cursor-*.AppImage 2>/dev/null | head -n 1)

if [ -z "$appimage_path" ]; then
  echo "[!] Cursor AppImage not found in ~/Downloads."
  echo "    Please place it there or modify the path inside the script."
  exit 1
else
  sudo cp "$appimage_path" /opt/Cursor/Cursor.AppImage
fi

sudo chmod +x /opt/Cursor/Cursor.AppImage
sudo ln -sf /opt/Cursor/Cursor.AppImage /usr/local/bin/cursor

echo "[✓] Cursor installation completed."
echo "You can now run Cursor using: cursor --no-sandbox"
```

---

## Troubleshooting

If something doesn’t work as expected:

| Issue | Solution |
|--------|-----------|
| **AppImage not found** | Make sure the file exists in `~/Downloads/` or change the script path. |
| **Cursor still shows the trial error** | Reboot the system after running the script to ensure all IDs and network changes apply. |
| **Network not working after MAC change** | Run `sudo systemctl restart NetworkManager` or reboot your device. |
| **Permission denied** | Make sure you gave the script executable permission using `chmod +x cursor-reset.sh`. |

---

## Example Output

```bash
[+] Removing Cursor config and installation...
[+] Resetting machine ID...
[+] Installing macchanger if not present...
[+] Detecting network interfaces...
[+] Changing MAC address for eth0...
[✓] System identifiers reset completed.
[+] Copying Cursor AppImage from Downloads...
[✓] Cursor installation completed.
You can now run Cursor using: cursor --no-sandbox
```

---

**Author:** Elsayed Kewan
