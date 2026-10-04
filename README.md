# LocalIP

**Version 1.0.16 — Developed by Elias.**

A small Windows desktop utility for network administrators and support teams. It shows the computer name and the current local IPv4 address in a small floating window, and repairs the most common client-network issues with a single click.

## Features

- Always-visible floating window with computer name, current local IP, and a green `Connected` / red `Offline` status indicator.
- One left-click runs network maintenance and refreshes the IP: clears the LAN proxy setting, flushes DNS (`ipconfig /flushdns`), renews the DHCP lease (`ipconfig /release` + `ipconfig /renew`).
- Right-click copies the displayed IP to the clipboard; the `PIN` button toggles always-on-top; the window can be dragged anywhere.
- Automatic IP refresh at startup and a deployment self-check via `LocalIP.exe --deployment-check`.
- Self-healing client window: a headless watchdog task re-checks every 2 minutes and silently relaunches the window if it was closed (for example from Task Manager); only one instance ever runs.
- Maintenance log at `C:\ProgramData\LocalIP\maintenance.log`.

## Installation

Run `LocalIPSetup.exe` **as an administrator**. The single-file installer asks for the local installation folder and the parent folder of the central `LocalIPShare` distribution share (created automatically), publishes the release files into the share, and finishes with an automatic deployment-verification report. No Python or extra runtime is required.

## Server mode

Normal installation for the administrator's own computer: installs the application, publishes `LocalIP.exe` + `LocalIP.version` to the Share Folder with read-only client access, and creates a desktop shortcut. No forced startup execution is enabled on that computer. An optional installer checkbox can also create the centralized Group Policy deployment directly (Domain Administrator only). In a Domain, clients are served centrally from the share through a Group Policy Computer Startup Script (available from the developer); in a Workgroup, run the same Setup on each client with local Administrator credentials.

## Client mode

Managed-client installation: registers the `LocalIP` SYSTEM startup task (runs headless maintenance at every Windows startup, no delay) plus the all-user launcher that shows the window after logon, and runs the maintenance task once immediately. A `LocalIPWatchdog` SYSTEM task re-checks every 2 minutes and silently relaunches the window if it was closed. On a managed computer the forced window cannot be dismissed with Alt+F4 (administrators can still remove it via uninstall).

## Security

- Installation and uninstallation require administrator privileges; standard users can neither install nor alter the deployment.
- The SYSTEM task performs local repairs only (proxy/DNS/DHCP) and never displays a UI before logon; the interactive window appears only after a user logs on.
- No data ever leaves the machine: IP discovery uses the local routing table (no packet is sent) and all logs stay on the local disk.
- The installer grants clients read-only access to the distribution share and never changes firewall rules or profiles.
- Uninstall removes the application files, both scheduled tasks, the launcher value, the shortcut, and the stored settings; the central share is left intact for remaining clients.

## Code signing

Release binaries are planned to be code-signed through the SignPath Foundation.
