# Home Security Operations Center (SOC) Lab with Centralized SIEM

## Overview
This project documents a home SOC lab built around Splunk Enterprise and Splunk Universal Forwarder. It covers KVM/QEMU virtualization on Pop!_OS, centralized log collection, dashboard creation, and threat detection use cases.

## Table of Contents
- [Overview](#overview)
- [What’s included](#whats-included)
- [Repository structure](#repository-structure)
- [Getting started](#getting-started)
- [Key components](#key-components)
- [Notes](#notes)

## What’s included
- Central SIEM architecture using Splunk Enterprise
- Distributed forwarder setup and log ingestion
- Dashboard and alerting examples for security monitoring
- Documentation for environment setup and troubleshooting
- Visualizations for authentication and brute-force detection

## Repository structure
- `README.md` — high-level project summary
- `docs/Architecture.md` — architecture, features, and technical stack
- `docs/setup.md` — installation and Splunk deployment instructions
- `docs/splunk-soc-lab-documentation.md` — lab notes, validation, and verification
- `IMG/` — lab screenshots and diagrams

## Getting started
1. Review `docs/setup.md` to prepare the host and create the Splunk VM.
2. Use `docs/Architecture.md` to understand the lab design and supported use cases.
3. Follow `docs/splunk-soc-lab-documentation.md` for validation steps and attack simulation results.

## Key components
- Pop!_OS host with KVM/QEMU virtualization
- Ubuntu Server VMs for Splunk indexer and forwarders
- Splunk Enterprise 9.x as the central SIEM
- Splunk Universal Forwarder for log collection
- Security dashboards for authentication and event correlation

## Notes
- Update IP addresses and credentials before applying configuration commands.
- Change default Splunk passwords after the first login.
- Use the `IMG/` folder for reference screenshots and dashboard examples.
