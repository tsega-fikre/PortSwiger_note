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







Lab: Blind SQL injection with conditional responses

PRACTITIONER
LAB Not solved

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a Welcome back message in the page if the query returns any rows.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.
Hint

ACCESS THE LAB
Solution

    Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the TrackingId cookie. For simplicity, let's say the original value of the cookie is TrackingId=xyz.

    Modify the TrackingId cookie, changing it to:
    TrackingId=xyz' AND '1'='1

    Verify that the Welcome back message appears in the response.

    Now change it to:
    TrackingId=xyz' AND '1'='2

    Verify that the Welcome back message does not appear in the response. This demonstrates how you can test a single boolean condition and infer the result.

    Now change it to:
    TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

    Verify that the condition is true, confirming that there is a table called users.

    Now change it to:
    TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

    Verify that the condition is true, confirming that there is a user called administrator.

    The next step is to determine how many characters are in the password of the administrator user. To do this, change the value to:
    TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a

    This condition should be true, confirming that the password is greater than 1 character in length.

    Send a series of follow-up values to test different password lengths. Send:
    TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a

    Then send:
    TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a

    And so on. You can do this manually using Burp Repeater, since the length is likely to be short. When the condition stops being true (i.e. when the Welcome back message disappears), you have determined the length of the password, which is in fact 20 characters long.
    After determining the length of the password, the next step is to test the character at each position to determine its value. This involves a much larger number of requests, so you need to use Burp Intruder. Send the request you are working on to Burp Intruder, using the context menu.

    In Burp Intruder, change the value of the cookie to:
    TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a

    This uses the SUBSTRING() function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.

    Place payload position markers around the final a character in the cookie value. To do this, select just the a, and click the Add § button. You should then see the following as the cookie value (note the payload position markers):
    TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
    To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. In the Payloads side panel, check that Simple list is selected, and under Payload configuration add the payloads in the range a - z and 0 - 9. You can select these easily using the Add from list drop-down.
    To be able to tell when the correct character was submitted, you'll need to grep each response for the expression Welcome back. To do this, click on the Settings tab to open the Settings side panel. In the Grep - Match section, clear existing entries in the list, then add the value Welcome back.
    Launch the attack by clicking the Start attack button.
    Review the attack results to find the value of the character at the first position. You should see a column in the results called Welcome back. One of the rows should have a tick in this column. The payload showing for that row is the value of the character at the first position.

    Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the Intruder tab, and change the specified offset from 1 to 2. You should then see the following as the cookie value:
    TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
    Launch the modified attack, review the results, and note the character at the second offset.
    Continue this process testing offset 3, 4, and so on, until you have the whole password.
    In the browser, click My account to open the login page. Use the password to log in as the administrator user.

Note

For more advanced users, the solution described here could be made more elegant in various ways. For example, instead of iterating over every character, you could perform a binary search of the character space. Or you could create a single Intruder attack with two payload positions and the cluster bomb attack type, and work through all permutations of offsets and character values.
