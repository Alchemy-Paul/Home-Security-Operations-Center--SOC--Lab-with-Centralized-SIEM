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
# Replace <INDEXER_IP> with your Splunk server's IP address
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

# Repeat for additional forwarder VMs as needed