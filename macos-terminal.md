# macOS Terminal fixes for helpdesk

## A saved Wi-Fi network that keeps coming back

Since **macOS Sonoma**, known networks live in two places. Removing only one means the Mac rejoins.

```bash
# 1. Find the Wi-Fi interface (usually en0)
networksetup -listallhardwareports

# 2. Remove it from the preferred list
sudo networksetup -removepreferredwirelessnetwork en0 "Old-Network-Name"

# 3. Remove it from the known-networks store
sudo /usr/libexec/PlistBuddy -c "Print" /Library/Preferences/com.apple.wifi.known-networks.plist | grep -i "Old-Network-Name"
sudo /usr/libexec/PlistBuddy -c "Delete :wifi.network.ssid.Old-Network-Name" /Library/Preferences/com.apple.wifi.known-networks.plist

# 4. Toggle Wi-Fi
networksetup -setairportpower en0 off && networksetup -setairportpower en0 on
```

## "Rename the user" means three different things

| What users call it | What it is | Where to change it |
|---|---|---|
| Name on the login screen | **Full Name** | System Settings → Users & Groups (right-click → Advanced) |
| Home folder / terminal name | **Account name** (short name) | Needs an admin account: rename `/Users/<old>` *and* the account's home path together |
| Name on the network | **Computer / host name** | See below |

```bash
# See all three computer names
scutil --get ComputerName; scutil --get LocalHostName; scutil --get HostName

# Set them consistently
sudo scutil --set ComputerName  "MAC-JDOE-01"
sudo scutil --set LocalHostName "MAC-JDOE-01"
sudo scutil --set HostName      "MAC-JDOE-01"
```

> ⚠️ Renaming the **account name** wrong can lock the user out of their home folder. Always keep a second admin account signed in while you do it.
