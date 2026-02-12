# Installation Guide

```sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```
   

Log out and back in
Verify KVM Installation
First, let's make sure everything installed correctly:

# Check if KVM modules are loaded
```lsmod | grep kvm```

# Check if your user is in the right groups
```groups | grep -E 'libvirt|kvm'```

# Start and enable libvirt service
```sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```

# Check status
```sudo systemctl status libvirtd```

# Step 2: Download Ubuntu Server ISO
# Create a directory for ISOs
```mkdir -p ~/VMs/ISOs
cd ~/VMs/ISOs

wget https://releases.ubuntu.com/22.04/ubuntu-22.04.5-live-server-amd64.iso
```

# Verify download completed
```ls -lh ubuntu-22.04.5-live-server-amd64.iso```


# Step 3: Launch virt-manager and Create VM
# Launch virt-manager
```virt-manager```


In virt-manager GUI:
# 3.1 Create New Virtual Machine:

Click "Create a new virtual machine" (top-left icon)
Select "Local install media (ISO image or CDROM)"
Click Forward

# 3.2 Choose ISO:

- Click "Browse..."
- Click "Browse Local"
- Navigate to ~/VMs/ISOs/
- Select the Ubuntu Server ISO
- It should auto-detect as "Ubuntu 22.04"
- Click Forward

# 3.3 Memory and CPU:
Based on your total RAM:
If you have 8GB total:

Memory: 3072 MB (3GB) (minimum)
CPUs: 2

Click Forward

# 3.4 Storage:

Create disk image: 40 GB
Click Forward

# 3.5 Final Setup:

Name: Splunk-Server
Network: Virtual network 'default': NAT (default is fine)
Check "Customize configuration before install"
Click Finish

# 3.6 Customize Configuration:
Before starting the VM, optimize settings:

Overview:

Firmware: BIOS (unless you need UEFI)


CPUs:

Configuration: Check "Copy host CPU configuration"
Topology: 2 cores is fine


Network:

Device model: virtio (better performance)


Click "Begin Installation" (top-left)

Step 4: Install Ubuntu Server
The VM will boot into Ubuntu installer:
Installation Steps:

Language: English
Keyboard: Your layout
Installation type: Ubuntu Server
Network: Use defaults (DHCP)
Proxy: Leave blank
Mirror: Use defaults
Storage: Use entire disk (defaults)
Profile Setup:

Your name: soc-admin (or whatever you prefer)
Server name: splunk-server
Username: socadmin
Password: Create strong password & write it down!


SSH Setup:

Install OpenSSH server (makes life easier)


Featured snaps: Skip (don't install anything)
Wait for installation to complete
Reboot when prompted

Step 5: First Login & Update
After reboot, login with your credentials:

# Update system
```sudo apt update && sudo apt upgrade -y```

# Install useful tools
```sudo apt install -y net-tools curl wget vim```

# Check IP address (you'll need this)
```ip addr show```

# Or if net-tools installed:
```ifconfig```


* Note your VM's IP address - it'll be  something like 192.168.122.X



# Step 6: Install Splunk Enterprise on Splunk-Server VM

# Download Splunk Enterprise
```wget -O splunk-9.1.2-2b6e6e66d8a6-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.1.2/linux/splunk-9.1.2-2b6e6e66d8a6-linux-2.6-amd64.deb"```

# Install Splunk
```sudo dpkg -i splunk-9.1.2-2b6e6e66d8a6-linux-2.6-amd64.deb```

# Start Splunk and accept license
```sudo /opt/splunk/bin/splunk start --accept-license --answer-yes --no-prompt```

# Enable Splunk to start on boot
```sudo /opt/splunk/bin/splunk enable boot-start```

# Access Splunk Web UI
# From your host machine, open browser to http://<VM_IP>:8000
# Default credentials: admin/changeme (change password immediately)

# Step 7: Create and Configure Forwarder VMs

# Create additional VMs for log collection (repeat Step 3 for each forwarder VM)

# For each forwarder VM:
# - Name: e.g., Forwarder-1, Forwarder-2
# - Memory: 2048 MB (2GB)
# - CPUs: 1
# - Storage: 20 GB
# - Install Ubuntu Server 22.04 LTS (same as Splunk server)

# After Ubuntu installation on forwarder VM:

# Update system
```sudo apt update && sudo apt upgrade -y```

# Install required packages
```sudo apt install -y curl wget vim```

# Download Splunk Universal Forwarder
```wget -O splunkforwarder-9.1.2-2b6e6e66d8a6-linux-2.6-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.1.2/linux/splunkforwarder-9.1.2-2b6e6e66d8a6-linux-2.6-amd64.deb"```

# Install Universal Forwarder
```sudo dpkg -i splunkforwarder-9.1.2-2b6e6e66d8a6-linux-2.6-amd64.deb```

# Configure forwarder to send data to indexer
# Replace <INDEXER_IP> with your Splunk server's IP address note that changeme (is your set password)
```sudo /opt/splunkforwarder/bin/splunk add forward-server <INDEXER_IP>:9997 -auth admin:changeme```

# Set up data inputs (example: system logs)
```sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -auth admin:changeme```
```sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -auth admin:changeme```

# Start the forwarder
```sudo /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes --no-prompt```

# Enable forwarder to start on boot
```sudo /opt/splunkforwarder/bin/splunk enable boot-start```

# Verify forwarder connection
# On Splunk server, check Forwarder Management in Settings > Forwarder Management

# Step 8: Create Splunk Dashboards

## 8.1 Access Splunk Web Interface
```
1. Open web browser on your host machine
2. Navigate to http://<SPLUNK_SERVER_IP>:8000
3. Login with admin credentials (admin/changeme - or your new password)
```

## 8.2 Create Security Overview Dashboard

**Via GUI:**
```
1. Click "Dashboards" in top navigation
2. Click "Create New Dashboard"
3. Enter Dashboard Name: "Security Operations Overview"
4. Choose layout: Classic (2-column)
5. Click "Create Dashboard"
```

**Add Panel 1: Authentication Events**
```
1. Click "Edit" (top right)
2. Click "Add Panel" > "New"
3. Paste this SPL query:
   index=_internal group=queue name=tcpout_queue | stats count by host
4. Click "Visualize"
5. Choose "Column Chart" visualization
6. Click "Save"
7. Title: "Forwarder Status"
```

**Add Panel 2: Failed Authentication Attempts**
```
1. Click "Add Panel" > "New"
2. Paste this SPL query:
   index=main source=/var/log/auth.log "Failed password" | stats count by host
3. Choose "Table" visualization
4. Title: "Failed Login Attempts"
```

**Add Panel 3: System Events Timeline**
```
1. Click "Add Panel" > "New"
2. Paste this SPL query:
   index=main | timechart count by host
3. Choose "Line Chart" visualization
4. Title: "Event Activity Timeline"
```

## 8.3 Create Custom Dashboard via XML (Advanced)

**On Splunk Server, create dashboard XML:**
```bash
sudo vim /opt/splunk/etc/apps/search/local/data/ui/views/soc_dashboard.xml
```

**Paste this content:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<dashboard version="1.1">
  <label>SOC Security Dashboard</label>
  <description>Real-time security monitoring and threat detection</description>
  <refresh>30</refresh>
  
  <row>
    <panel>
      <title>Authentication Failures by Host</title>
      <single>
        <search>
          <query>index=main source=/var/log/auth.log "Failed password" | stats count</query>
          <earliest>-24h@h</earliest>
          <latest>now</latest>
          <refresh>5m</refresh>
        </search>
        <option name="drilldown">all</option>
      </single>
    </panel>
    
    <panel>
      <title>Connected Forwarders</title>
      <single>
        <search>
          <query>index=_internal group=queue name=tcpout_queue | stats count</query>
          <earliest>-1h</earliest>
          <latest>now</latest>
        </search>
      </single>
    </panel>
  </row>
  
  <row>
    <panel>
      <title>Failed Login Attempts (Last 24h)</title>
      <table>
        <search>
          <query>index=main source=/var/log/auth.log "Failed password" | stats count by host, user | sort - count</query>
          <earliest>-24h@h</earliest>
          <latest>now</latest>
        </search>
        <option name="wrap">true</option>
        <option name="count">20</option>
      </table>
    </panel>
  </row>
  
  <row>
    <panel>
      <title>Event Volume by Host</title>
      <chart>
        <search>
          <query>index=main | timechart count by host</query>
          <earliest>-7d@h</earliest>
          <latest>now</latest>
        </search>
        <option name="charting.chart">line</option>
        <option name="charting.axisTitleX.text">Time</option>
        <option name="charting.axisTitleY.text">Event Count</option>
      </chart>
    </panel>
  </row>
</dashboard>
```

**Save and exit (vim commands):**
```
Press ESC
Type :wq
Press ENTER
```

## 8.4 Refresh Splunk and View Dashboard

```bash
# Restart Splunk to load new dashboard
sudo /opt/splunk/bin/splunk restart

# Or just refresh in Web UI: Settings > General Settings > Restart Splunk
```

**Access your custom dashboard:**
```
1. Login to Splunk web (http://<IP>:8000)
2. Click "Dashboards"
3. Click "SOC Security Dashboard"
4. Monitor real-time security events!
```

## 8.5 Verify Data is Being Collected

Before dashboards display data, verify logs are being ingested:

```
1. Go to Splunk web interface
2. Click "Search & Reporting"
3. Run this query: index=* | stats count by host
4. Should return results from forwarders and server
5. If no data, check forwarder connectivity in Settings > Forwarder Management
```

## Dashboard Best Practices

- **Refresh Rate:** Set to 5-10 minutes for dashboards with heavy queries
- **Time Range:** Use appropriate ranges (-24h for daily analysis, -7d for trends)
- **Alerts:** Add alerts to high-risk panels (Settings > Searches, Reports and Alerts)
- **Permissions:** Share dashboards with SOC team (Sharing & Permissions)