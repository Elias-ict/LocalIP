# LocalIP

**Version 1.0.19 — Developed by Elias-ICT.**

A small Windows desktop utility for network administrators and support teams. It shows the computer name and the current local IPv4 address in a small floating window, and repairs the most common client-network issues with a press-and-hold.

## Features

- Always-visible floating window with computer name and current local IPv4 address (one line per address).
- Press and hold the left mouse button (~1 second) to run a network adapter refresh operation. A plain click does nothing, so stray clicks can never disturb the network.
- Right-click copies the current IP to the clipboard; the window can be dragged anywhere and remembers its position.
- The address list refreshes itself at startup and follows the network on its own.
- A small maintenance log is kept on the local disk.
- On a managed client the window is brought back on its own if it was closed (for example from Task Manager); only one instance ever runs.
- Administrators can run a deployment self-check from the command line at any time.

## Installation

Run `LocalIPSetup.exe` **as an administrator**. The single-file installer asks for the local installation folder and the parent folder of the central `LocalIPShare` distribution share (created automatically), publishes the release files into the share, and finishes with an automatic deployment-verification report. No Python or extra runtime is required.

## Server mode

Normal installation for the administrator's own computer: installs the application, publishes the release files to the Share Folder with read-only client access, and creates a desktop shortcut. No forced startup execution is enabled on that computer. In a Domain, Setup can also configure the centralized deployment itself (Domain Administrator rights required), and the clients are then served from the share through a Group Policy Computer Startup Script (available from the developer); in a Workgroup, run the same Setup on each client with local Administrator credentials.

## Client mode

Managed-client installation: the background maintenance runs automatically at every Windows startup as the local system account, and the window appears after a user logs on. The maintenance also runs once immediately after the installation. A second background check keeps the window present if it was closed. On a managed computer the forced window cannot be dismissed with Alt+F4 (administrators can still remove it via uninstall).

## Central configuration

The Group Policy does one thing: it hands each client the Share Folder path. Everything else travels in one small configuration file, `LocalIP.command.json`, that Setup publishes into the Share Folder and that every managed client reads at startup.

Edit that file with a text editor; the next startup of each client applies it. It carries the central release version and the deployment settings, including the client DNS choice. The installer asks for that choice in two states: **Automatic (DHCP)** is the default and the recommended state - the client takes the DNS the network gives it - and **Manual** pins a comma-separated list of servers (primary, then secondary) as the static IPv4 DNS of the connected client adapters. Use Manual only when you must pin specific servers. The value covers client machines only — the administrator's own machine keeps its own DNS — and it reaches every managed client through the supplied deployment startup script.

## Security

- Installation and uninstallation require administrator privileges; standard users can neither install nor alter the deployment.
- The background maintenance runs as the local system account and performs local network repairs only; it never displays a UI before logon, and the interactive window appears only after a user logs on.
- No data ever leaves the machine: IP discovery uses the local routing table (no packet is sent) and all logs stay on the local disk.
- The installer makes exactly ONE firewall change: it enables the inbound SMB rule **"File and Printer Sharing (SMB-In)" (TCP 445)** when that rule is closed, on the network profile the computer is currently connected to. The reason is that the Share Folder is an SMB share: while that port is blocked, no client can open it at all. The rule is only ever opened, never closed; no other firewall rule and no firewall profile setting is touched; and the uninstaller does not revert it, because the same port is what the Windows administrative shares use as well. If a policy of yours forbids that rule, the Share Folder cannot be reached by clients — have the rule enabled by policy instead.
- The installer grants clients read-only access to the distribution share. The share carries the release files and the configuration file only; it holds no credentials and no personal data.
- Network communication is limited to the delivery share on the local network or domain: the software opens no connection to the internet at all — no telemetry, no update checks, no external services, no account of any kind.
- The two background tasks on a managed client act only on that computer's own network configuration and its connection to the delivery share; they never reach other machines and never read or modify your documents or user files.
- The delivery share is the one trust boundary of a centralized deployment: keep it on a machine you control and leave write access to administrators only. Clients apply the publication and check its version marker before replacing local files, so a stale or copied-around share cannot inject an older release.
- Uninstall removes the application files, the background tasks, the launcher value, the shortcut, and the stored settings (including the settings key itself when empty); the central share is left intact for the remaining clients.
