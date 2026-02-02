# Copilot Instructions for SOC Lab Project

## Project Overview
This is a documentation repository for setting up a Home Security Operations Center (SOC) Lab with centralized Splunk SIEM. The project focuses on hands-on SOC training through virtualization and log analysis.

## Architecture Understanding
- **Host Environment**: Pop!_OS with KVM/QEMU virtualization
- **SIEM Infrastructure**: Dedicated Ubuntu Server VM running Splunk Enterprise (4GB RAM, 2 vCPUs, 40GB storage)
- **Data Collection**: Universal Forwarders on additional VMs sending logs to central indexer on port 9997
- **Web Interface**: Splunk web UI on port 8000

## Key Components
- `readme`: Project overview and features
- `docs/Architecture.md`: Technical stack, components, and detection rules
- `docs/setup.md`: Step-by-step KVM and VM installation guide

## Documentation Patterns
- Use markdown with code blocks for shell commands (```bash)
- Structure guides as numbered steps with clear headings
- Include verification commands after installations
- Document specific resource allocations (RAM, CPU, storage)
- List technical stack with versions (e.g., Ubuntu Server 22.04 LTS, Splunk Free 9.1.2)

## Development Workflows
- Update VM specifications in setup.md when testing new configurations
- Add new detection rules to Architecture.md with SPL examples
- Include IP address checks and network verification steps
- Test documentation on fresh Pop!_OS installations

## Automation Opportunities
- Generate bash scripts from setup.md command blocks for automated VM provisioning
- Create Ansible playbooks for Splunk forwarder deployment
- Develop monitoring scripts for forwarder health checks

## Code Generation Guidelines
- When creating scripts, use absolute paths and error handling
- Follow Ubuntu Server best practices (systemd services, apt updates)
- Include comments explaining Splunk-specific configurations
- Test scripts in KVM environment before documenting

## File Organization
- Keep setup instructions in `docs/setup.md`
- Technical details in `docs/Architecture.md`
- Images in `IMG/` directory
- No code files currently - focus on documentation maintenance