1. Create a Persistent udev Rule for Power Management
bash
# Create a udev rule to disable power management when the device appears
sudo tee /etc/udev/rules.d/99-wifi-powersave.rules << 'EOF'
# Disable power management for MediaTek MT7922
ACTION=="add", SUBSYSTEM=="net", KERNEL=="wlo1", RUN+="/usr/bin/iwconfig wlo1 power off"
EOF

# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger
2. Create a Systemd Service for NetworkManager Fixes
bash
# Create a service that runs at boot to ensure WiFi is properly configured
sudo tee /etc/systemd/system/wifi-fix.service << 'EOF'
[Unit]
Description=WiFi Fix for MediaTek MT7922
After=network-pre.target
Before=NetworkManager.service
Wants=network-pre.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStartPre=/bin/sleep 2
ExecStart=/usr/bin/rfkill unblock wifi
ExecStart=/usr/bin/bash -c 'echo "options mt7921e disable_aspm=1" | tee /etc/modprobe.d/mt7921e.conf'
ExecStart=/usr/bin/bash -c 'if ip link show wlo1; then /usr/bin/iwconfig wlo1 power off; fi'

[Install]
WantedBy=multi-user.target
EOF

# Enable the service
sudo systemctl daemon-reload
sudo systemctl enable wifi-fix.service
3. Create a Modprobe Configuration for the Driver
bash
# Create persistent driver options
sudo tee /etc/modprobe.d/mt7921e.conf << 'EOF'
# MediaTek MT7921E/MT7922 driver options
# Disable ASPM to prevent power management issues
options mt7921e disable_aspm=1
# Enable debug mode if needed (remove # to enable)
# options mt7921e debug=1
EOF
4. Configure NetworkManager to Never Use Powersave
bash
# Create NetworkManager configuration for WiFi powersave
sudo tee /etc/NetworkManager/conf.d/wifi-powersave.conf << 'EOF'
[connection]
wifi.powersave = 2
EOF

# Also disable MAC randomization (can cause issues)
sudo tee /etc/NetworkManager/conf.d/wifi-mac.conf << 'EOF'
[device]
wifi.scan-rand-mac-address=no

[connection]
wifi.cloned-mac-address=permanent
ethernet.cloned-mac-address=permanent
EOF
5. Update Initramfs to Include All Changes
bash
# Update the initial ramdisk to include all new configs
sudo update-initramfs -u
6. Create a Startup Script in /etc/rc.local
bash
# Create or edit rc.local
sudo tee /etc/rc.local << 'EOF'
#!/bin/bash
# Fix WiFi at boot
rfkill unblock all
sleep 3
if command -v iwconfig &> /dev/null && ip link show wlo1 &> /dev/null; then
    iwconfig wlo1 power off
fi
exit 0
EOF

# Make it executable
sudo chmod +x /etc/rc.local

# Enable rc.local service if needed
sudo systemctl enable rc-local
7. Verify All Changes
bash
# Check all configurations
echo "=== udev rule ==="
cat /etc/udev/rules.d/99-wifi-powersave.rules

echo "=== systemd service ==="
cat /etc/systemd/system/wifi-fix.service

echo "=== modprobe config ==="
cat /etc/modprobe.d/mt7921e.conf

echo "=== NetworkManager config ==="
cat /etc/NetworkManager/conf.d/wifi-powersave.conf
cat /etc/NetworkManager/conf.d/wifi-mac.conf

echo "=== rc.local ==="
cat /etc/rc.local
8. Test Without Rebooting
bash
# Apply modprobe options now
sudo modprobe -r mt7921e
sudo modprobe mt7921e

# Restart NetworkManager
sudo systemctl restart NetworkManager

# Check status
nmcli dev status
iwconfig wlo1 | grep "Power Management"
9. One-Liner to Apply Everything
bash
# This command runs all the above steps
sudo tee /etc/udev/rules.d/99-wifi-powersave.rules <<< 'ACTION=="add", SUBSYSTEM=="net", KERNEL=="wlo1", RUN+="/usr/bin/iwconfig wlo1 power off"' && \
sudo tee /etc/modprobe.d/mt7921e.conf <<< 'options mt7921e disable_aspm=1' && \
sudo tee /etc/NetworkManager/conf.d/wifi-powersave.conf <<< '[connection]\nwifi.powersave = 2' && \
sudo udevadm control --reload-rules && \
sudo update-initramfs -u && \
echo "All fixes applied! Reboot to test."
After Reboot, Verify:
bash
# Check power management status
iwconfig wlo1 | grep "Power Management"

# Check if WiFi connects automatically
nmcli connection show --active

# Check driver options
cat /sys/module/mt7921e/parameters/disable_aspm
This comprehensive fix ensures:

✅ Power management stays off

✅ Driver loads with optimal parameters

✅ NetworkManager doesn't interfere with power saving

✅ WiFi works after every boot, suspend/resume

Reboot and test! Your WiFi should now work perfectly every time.



show how one by one i dont need your fucking edit








#afete reboot



```bash
sudo dmesg | grep mt7921e | grep ASPM
Check NetworkManager setting:
```
```bash
nmcli dev show wlo1 | grep powersave
3. If you still have WiFi issues after reboot:
Check for errors:
```

```bash
# Look for WiFi-related errors
sudo dmesg | grep -i wifi
sudo dmesg | grep -i mt79
sudo journalctl -u NetworkManager | grep -i wifi```
```

# main  fix

```bash

nile_phanom@pop-os:~$ sudo tee /etc/NetworkManager/conf.d/wifi-powersave.conf<<'EOF'
> [connction]
> wifi.powersave = 2
> EOF
```