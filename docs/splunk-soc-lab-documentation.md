# SIEM Lab: Host Security Monitoring with Splunk

**Objective:** Establish a real-time security pipeline between a Pop!_OS host machine and a Splunk Enterprise VM to monitor brute-force login activity.

---

## 1. Environment Setup
- **Host:** Pop!_OS (Ubuntu-based) running Splunk Universal Forwarder.
- **Indexer/Search Head:** Linux VM running Splunk Enterprise.
- **Connection Port:** `9997` (Splunk indexer input).

---

## 2. Configuration Steps

### Phase A: Preparing the Indexer (VM)
1. Enable receiving on Splunk to accept data on port `9997`.
2. Fix directory permissions if Splunk displays "Could not find writer" errors.

```bash
sudo chown -R socadmin:socadmin /opt/splunk
```

### Phase B: Configuring the Universal Forwarder (Host)
If the Splunk administrator password is lost, use `user-seed.conf` to reset the admin password and restart Splunk.

```bash
# Example ownership correction for the forwarder installation
sudo chown -R brandogas:brandogas /opt/splunkforwarder

# Add an auth log monitor
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log
```

- Reset credentials by creating `user-seed.conf` and restarting the Splunk service.
- Ensure the forwarder can write log files and configuration files before starting.

## 3. The Failed Login Dashboard
A custom XML dashboard was created to visualize failed authentication events. See `docs/setup.md` for full dashboard creation instructions.

![Dashboard](../IMG/spike 2026-02-12_06-49.png)

## 4. Attack Simulation (The Brute Force)
To test the data pipeline, this loop was executed on the host machine:

```bash
for i in {1..100}; do
  su - root_hacker_demo -c "exit" 2>/dev/null
 done
```

![Brute Force Simulation](../IMG/brutefore execute 2026-02-12_07-25.png)

## 5. Results & Visualizations
- Successful pipeline: data flowed from `/var/log/auth.log` → forwarder → indexer.
- Detection: the dashboard displayed a spike of 152 events and highlighted the incident.
- Field extraction surfaced targeted users such as `whoopsie` and `root_hacker_demo`.

![Visualization](../IMG/new spike 2026-02-12_06-50.png)

## 6. Alerting Logic
- **Trigger:** Real-time monitoring of `index=main`.
- **Threshold:** More than 10 failed attempts within 1 minute.
- **Throttling:** 60-second suppression to reduce alert fatigue.
- **Action:** Logged to the incident tracking dashboard for follow-up.
