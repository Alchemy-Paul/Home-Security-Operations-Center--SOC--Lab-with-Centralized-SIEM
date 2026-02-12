# SIEM Lab: Host Security Monitoring with Splunk
**Objective:** Establish a real-time security pipeline between a Pop!_OS host machine and a Splunk Enterprise VM to monitor for brute-force attacks.

---

## 1. Environment Setup
* **Host:** Pop!_OS (Ubuntu-based) running Splunk Universal Forwarder.
* **Indexer/Search Head:** Linux VM running Splunk Enterprise (IP: `192.168.xxx.xxx`).
* **Connection Port:** `9997` (Splunk Indexing).

---

## 2. Configuration Steps

### Phase A: Preparing the Indexer (VM)
1. **Enable Receiving:** Configured Splunk to listen for data on port `9997`.
2. **Permission Fix:** Resolved "Could not find writer" errors in the Web UI by reclaiming ownership of the Splunk directory:
   ```sudo chown -R socadmin:socadmin /opt/splunk

## Phase B: Configuring the Universal Forwarder (Host)
yup i forgot my password and had to reset

- Reset Credentials: Recovered access by creating a user-seed.conf and restarting the service.

- Ownership Correction: Ensured the forwarder could write its own logs and configs:
```sudo chown -R brandogas:brandogas /opt/splunkforwarder```

- Log Monitoring: Added a monitor for the authentication logs:
``` /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log```


## 3. The Failed Login Dashboard
A custom XML dashboard was created to visualize security events. [setup.md]

![dashboard](IMG/dashboard.png)

## 4. Attack Simulation (The Brute Force)
To test the pipeline, a bash loop was executed on the host machine to simulate a high-volume credential stuffing attack:

```for i in {1..100}; do su - root_hacker_demo -c "exit" 2>/dev/null; done```

![Bruteforce](IMG/brutefore execute 2026-02-12_07-25.png)

## 5. Results & Visualizations
- Successful Pipeline: Data moved from /var/log/auth.log -> UF -> VM Indexer.

- Detection: The dashboard successfully showed a spike of 152 events, crossing the critical threshold and turning the display red.

- Field Extraction: Identified targeted users such as whoopsie and root_hacker_demo.
![Visualisation](IMG/new spike 2026-02-12_06-50.png)

## 6. Alerting Logic
* **Trigger:** Real-time monitoring of `index=main`.
* **Threshold:** > 10 failed attempts within a 1-minute window.
* **Throttling:** 60-second suppression to prevent alert fatigue.
* **Action:** Logged to "Triggered Alerts" dashboard for incident response.
