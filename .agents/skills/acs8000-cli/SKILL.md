---
name: acs8000-cli
description: Translate natural language requests into Avocent ACS800/8000 Advanced Console System CLI commands for SSH sessions. managing console servers, configuring serial ports, controlling PDU power outlets, managing users, or navigating the ACS CLI.
license: © 2020 Vertiv Group Corp. All rights reserved. 
---

# ACS800/8000 CLI operator guide
Use this workflow to translate natural language requests into proper CLI commands for Avocent ACS800/8000 Advanced Console Systems over SSH.

## Apply this operating pattern
1. Determine user context first.
   - Confirm whether the user is an administrator or regular user (available top-level paths differ).
   - Confirm connection method: local console, SSH, Telnet, or Web Manager Appliance Viewer.
2. Discover before changing.
   - Use `pwd`, `ls`, and `show` to inspect current location and values.
   - Use `cd <space><tab><tab>` to discover valid path elements at any level.
   - Use `<tab><tab>` to discover command options elements at any level.
   - When `-- MORE --:` is displayed at the end of a partial `ls` listing:
     - <cr> causes the page to advance
     - `q`<cr> causes the `ls` command to exit
   
3. Make configuration changes explicitly.
   - Use `set <parameter>=<value>` for parameter updates.
   - Use `add <path>` to add a node.
   - Use `delete <path> <parameter>` to remove a node.
4. Finalize every edit session.
   - Use `save` or `commit` depending on the type of configuration change
     - `save` is used to save port and power management changes, user name changes, and network host additions
     - `commit` is used to delete network hosts and for all other configuration changes.
   - Use `revert` to discard pending changes.
   - Use `cancel` to leave an edit session and discard pending changes.
5. Verify after each change.
   - Re-run `show` and/or `list_configuration`.
   - Confirm prompt state is clean (no pending changes indicator).

## Explain prompt and navigation semantics
- Treat CLI as a tree of paths.
- Use `/` separators, and translate Web Manager spaces to underscores.
  - Example: `Users - Local Accounts - User Names` becomes `cd /users/local_accounts/user_names`.
- Use `cd ..` to move up one level, `cd ../..` for multiple levels, `cd /` for root.
- Explain prompt markers when relevant:
  - `--:- ... cli->` indicates normal state.
  - `**:- ... cli->` indicates unsaved changes pending commit/revert.
  - Wizard prompts may appear as `--:#- [path] cli->`.
- Treat commands as case-sensitive.
- Use `references/navigation.md` for a navigation map of all paths and settings

## Provide command help in this order
1. Start with a minimal command sequence for the task.
2. Show placeholders explicitly (for example `<port_number>` or `<pdu_name>`).
3. Add verification commands (`show`, `ls`, `list_configuration`).
4. Add rollback/fallback (`revert` or `Ctrl+z` for session suspend) when applicable.

## Core Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `cd <path>` | Navigate to location | `cd /network/settings` |
| `cd ..` | Go up one level | `cd ../..` (two levels) |
| `cd /` | Go to root | |
| `pwd` | Show current path | |
| `ls` | List subnodes at current location | `ls authentication` |
| `show` | Display parameters/values | `show` |
| `set <param>=<value>` | Set a parameter | `set status=enabled` |
| `add` | Add a new node (user, host, etc.) | `add network/hosts` |
| `delete <path> <param>` | Delete a node | `delete users/local_accounts/user_names/fred` |
| `revert` | Undo unsaved changes | |
| `list_configuration` | Export replayable config for current node } }
| `cli` | Launch CLI from Linux shell | |
| `help` | Show help | |
| `exit` or `quit` | Exit CLI | |

## Navigation Tree (Top Level)

```
/
├── access/              # Connect to devices, view status
├── active_sessions/     # View/kill active sessions
├── authentication/      # Auth servers (RADIUS, TACACS+, LDAP, Kerberos)
├── change_password/     # Change your password
├── events_and_logs/     # Logging configuration
├── monitoring/          # System monitoring
├── network/             # Network settings, devices, firewall
├── pluggable_devices/   # USB devices, SD cards
├── ports/               # Serial and auxiliary port configuration
├── power_management/    # PDU management
├── sensors/             # Temperature and environmental sensors
├── system/              # Security, date/time, boot config
├── system_tools/        # Firmware upgrade, backup/restore
└── users/               # User accounts and groups
```

## Use these high-value command patterns

### Serial console access
- Enter access level: `cd /access`
- Connect to port: `connect <port_name>`
- Authenticate if prompted.
- Suspend target session back to CLI: `Ctrl+z`

### Multi-session access (at `cd /access/<port_name>`)
- View-only attach: `sniff`
- Read/write attach: `share`
- List shared users: `list_shared_session`
- Terminate shared user: `kill_shared_session <username>`
- Send message: `sendmsg <username> <message>`
- Show/clear port data buffer: `show_databuf`, `clean_databuf`
- Show/clear appliance data log: `show_appliance_databuf`, `clean_appliance_databuf`

### Configure Serial Port with CAS Profile
```
set_cas ports/serial_ports/ <port_number>
set status=enabled
save
```

### Power control
- From `cd /access`, control all outlets by PDU:
  - `on <pdu_name>`
  - `off <pdu_name>`
  - `cycle <pdu_name>`
- From `cd /access/<pdu_name>`, control specific outlets:
  - Single: `on <outlet_number>`
  - Range: `off <start_outlet_number>-<end_outlet_number>`
  - List: `cycle <outlet_number>,<outlet_number>,...
- From `cd /power_management/pdus/<pdu_name>/outlets`, also use:
  - `lock <outlet_number>,...`
  - `unlock <outlet_number>,...`

### Toggle IPv6 (checkbox-equivalent flow)
1. `cd /network/settings`
2. `show`
3. `set enable_ipv6=yes` or `set enable_ipv6=no`
4. Optionally set:
   - `set get_dns_from_dhcpv6=yes`
   - `set get_domain_from_dhcpv6=yes`
5. `show`
6. `commit`

### Configure a serial port for CAS
1. `set_cas ports/serial_ports/ <port_number>`
2. `show`
3. `set status=enabled`
4. `show`
5. `save`

### Configure a serial port for Power profile
1. `set_power ports/serial_ports/ <port_number>`
2. `show`
3. `set status=enabled`
4. `save`/`commit`
5. `show` to verify profile and status.

## Escalate to bundled reference when needed
- Start with `references/core-commands.md` for detailed command semantics, prerequisites, and admin path mapping.
- Use `references/section-index.md` to jump to exact manual sections.
- Use `references/acs8000-command-reference-transcript.md` for near-complete guide text when handling uncommon, advanced, or edge-case operations.
- Use `references/navigation.md` for a navigation map of all paths and settings
- Use `references/examples.md` for a collection of use case examples
- For uncommon operations not covered in recipes, derive navigation path via `cd <tab><tab>`, then use `show` and `help` before proposing destructive actions.

### Transcript search patterns
- Search by command section: `2.1.<n>`, `2.2.<n>`, `2.3`
- Search by operation name: `set_power`, `set_dial-in`, `clone_ports`, `kill_shared_session`, `show_databuf`
- Search by admin path: `/system/`, `/network/`, `/ports/`, `/users/`, `/events_and_logs/`, `/power_management/`
- Search edge cases: `Appendix A`, `Appendix B`, `Appendix C`, `Migration CLI`, `recover-flash.sh`, `sudo`

