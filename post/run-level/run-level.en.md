---
title: What is a Run Level and why do I need one?
date: 2025-08-11T12:35:55.661Z
tags:
  - Linux
  - RunLevel
  - SystemAdmin
  - SysVinit
  - Systemd
category: develop
draft: false
lastmod: 2025-08-11T22:22:39+09:00
layout:
CSS summary: The concept of runlevels in Linux systems, the characteristics of each level, and the transition to the modern systemd target system.
series:
SeriesOrder:
banner:
---

## Intro

While preparing for my Linux Master certification, I came across a rather unfamiliar word: Run Level.  
As a regular user of Ubuntu or CentOS, I took it for granted that sometimes the system boots into the GUI and sometimes into the terminal, but I learned that these are governed by rules called runlevels, so I thought I'd summarize them.

## What is a Run Level?

**Run Level refers to the operational state of the `init` process in Unix/Linux based operating systems. It is responsible for defining what services and processes the system will start with when it boots up. In other words, you can think of it as a set of scenarios that predetermine 'what to do' when you first turn on your computer.

A runlevel is a numbered representation of what mode the system will operate in. Each runlevel runs a different combination of services and allows the system administrator to control the behavior of the system.

In Linux systems, seven runlevels are traditionally defined, numbered 0 through 6. Let's take a look at the characteristics of each runlevel

### Run Level 0 - Shutdown.

- Purpose: Shut down the system safely
- Features: Clean up all processes and shut down the system completely
- Example Usage**: `init 0` or `shutdown -h now`.

### Run Level 1 - Single User Mode

- Purpose: System administration and maintenance
- Features**:
  - Only the root user can log in
  - Disable network services
  - Run only minimal system services
- Example of use**:
  - Recover forgotten root password
  - Recover system files
  - Modify important configuration files

This is a similar concept to safe mode in windows.

### Run Level 2 - Multi-user mode (no NFS)

- **Purpose**: Local multi-user environment.
- Features**:
  - Allows multiple user logins.
  - Network File System (NFS) services are disabled
  - Supports local networking only

> [!note] Network File System (NFS)
> A system for sharing files over a network.

### Run Level 3 - Full multi-user mode

- Purpose: Mainly used in server environments
- Features:
  - Enables all network services.
  - Provides only a command line interface (CLI)
  - No graphical interface
- Examples of use: Web servers, database servers, etc.

### Run Level 4 - Customize

- Purpose: Arbitrarily defined by the user
- Features: not commonly used, can be customized for special purposes

### Run Level 5 - Graphical Multi-User Mode

- **Purpose**: Desktop environment.
- Features**:
  - Enables all network services.
  - Provides a graphical user interface (GUI)
  - X-based login, such as X Window.

### Run Level 6 - System Reboot

- Purpose: Restart the system
- Features: Clean up all processes and reboot the system
- Example Usage**: `init 6` or `reboot`.

## Systemd and the Target System

Modern Linux distributions (RHEL 7+, Ubuntu 16.04+, CentOS 7+, etc.) use **systemd** instead of the traditional SysVinit. systemd uses the concept of **Target** instead of runlevels.

### Main Systemd Targets

| Traditional Run Level | Systemd Target | Description | Additional Features |
| -------------- | ----------------- | ------------- | ----------------------------- |
| 0 | poweroff.target | system shutdown | perform a safe shutdown procedure |
| 1 | rescue.target | single-user mode | for recovery operations, run only minimal services |
| 2,3,4 | multi-user.target | Multi-user CLI mode | Network included, no GUI |
| 5 | graphical.target | Graphical multi-user mode | multi-user.target + display manager |
| 6 | reboot.target | Reboot the system | Perform a safe restart procedure |

### Hierarchy of Targets

systemd targets are organized hierarchically, with parent targets containing child targets:

Modern Linux distributions (RHEL 7+, Ubuntu 16.04+, CentOS 7+, etc.) use **systemd** instead of the traditional SysVinit. This represents a fundamental change in the way systems are initialized, not just a tool replacement.

### Why the change?

#### Limitations of SysVinit

The traditional SysVinit system has been around since the 1980s and while it is a stable system, it has shown some limitations to modern requirements:

1. **Inefficiency of sequential booting**.

   ```bash.
   # SysVinit: services start sequentially.
   S01network -> S02firewall -> S03database -> S04webserver
   # wait until each service is fully started before starting the next service
   ````

2. **Complexity of managing dependencies**.
   - Manage dependencies between services only with script numbers
   - Unable to resolve dynamic dependencies
   - Difficult to debug when errors occur in complex dependency trees

3. Lack of response to hardware changes
   - Lack of support for dynamic hardware such as USB, hotplug devices, etc.
   - Difficult to respond flexibly to network interface changes

4. Decentralization of log management
   - Different log formats and locations for each service
   - Difficulty identifying system-wide status

#### Systemd's innovative approach

systemd solved these problems in the following ways

1. **Parallel booting**.

   ```bash
   # systemd: start non-dependent services at the same time
   network.service + firewall.service + basic.target
   └─ database.service + webserver.service (after network is done)
   ```

2. **Sophisticated Dependency Management**.

   ```ini
   # Example systemd unit file
   [Unit]
   Description=Web Server
   After=network.target database.service
   Wants=network.target
   Requires=database.service
   ```

3. **Event-based system
   - Real-time status monitoring via D-Bus
   - Instantly react to hardware changes
   - Enable socket-based services

4. Integrated logging system

   ```bash
   # Log all system logs to one interface
   $ journalctl -u nginx.service
   $ journalctl --since "1 hour ago"
   ```

### Run Level vs Target: Conceptual Differences

| Aspects | SysVinit Run Level | Systemd Target |
|------|-------------------|-----------------|
| **Structure** | Linear, single-state | Parallel, multi-state possible |
| **Dependencies** | Numeric ordering based | Explicit dependency declarations |
| **Scalability** | 7 fixed levels | Unlimited number of targets can be created |
| **Activation** | One runlevel at a time | Activate multiple targets simultaneously

```bash
# SysVinit: only one runlevel active
$ runlevel
N 3

# systemd: multiple targets active at the same time
$ systemctl list-units --type=target --state=active
basic.target loaded active active Basic System
multi-user.target loaded active active Multi-User System
network.target loaded active active Network
```

### Actual performance difference

Boot time comparison (typical server environment):

- **SysVinit**: approx. 60-120 sec.
- **systemd**: approx. 15-30 seconds

This performance improvement is due to parallelization and techniques like lazy loading and socket activation.

systemd uses the concept of **Target** instead of runlevels. Targets are not just a replacement for runlevels, they are a more flexible and powerful way of managing system state.

### Main Systemd Targets

| Traditional Run Level | Systemd Target | Description | Additional Features |
|-------------------|----------------|------|-----------|
| 0 | poweroff.target | system shutdown | performs a safe shutdown procedure
| 1 | rescue.target | single-user mode | for recovery operations, run only minimal services |
| 2,3,4 | multi-user.target | Multi-user CLI mode | Network included, no GUI |
| 5 | graphical.target | Graphical multi-user mode | multi-user.target + display manager |
| 6 | reboot.target | Reboot the system | Perform a safe restart procedure |

### Hierarchy of Targets

systemd targets are organized hierarchically, with parent targets containing child targets:

```bash
graphical.target
├── multi-user.target
│ ├── basic.target
│ │ ├── sysinit.target
│ │ └── sockets.target
│ ├── network.target
│ │ └── remote-fs.target
└── display-manager.service
```

Thanks to this structure:

- Enabling `graphical.target` automatically enables `multi-user.target` as well
- Necessary dependencies are automatically resolved
- Partial state transitions are possible

### Managing Target from Systemd

```bash
# Check the current default target
$ systemctl get-default

# Change the default target
$ sudo systemctl set-default multi-user.target

# Change target immediately
$ sudo systemctl isolate graphical.target

# check target's dependencies
$ systemctl list-dependencies graphical.target

# Boot into a specific target (from GRUB)
# add systemd.unit=rescue.target to kernel parameters
```

## Practical use case

### 1. Troubleshooting the graphical interface

When you have a problem with the GUI and can't log into the system:

```bash
# Enter edit mode by pressing the 'e' key in GRUB at boot time.
# Add to the end of the linux line:
systemd.unit=multi-user.target

# or the traditional way:
3
```

### 2. Optimize your server

Remove unnecessary GUI services from the server environment:

```bash
# Remove GUI and use CLI only
$ sudo systemctl set-default multi-user.target
$ sudo systemctl disable gdm # Disable Display Manager.
```

### 3. System Recovery

Repair corrupted configuration files or reset your password:

```bash
# Boot into single user mode
# Add systemd.unit=rescue.target from GRUB
# or traditional runlevel 1
```

## Security considerations

Security considerations when using runlevels:

### 1. The principle of least privilege

- Use runlevels that run only the services you need
- Use runlevel 3 on servers that don't require a GUI

### 2. Single-user mode security

- Can boot into single-user mode when physically accessed
- Requires GRUB password for added security

### 3. Service Management

- Regularly check services running at each runlevel
- Disable unnecessary services

## Troubleshooting

### Booted to the wrong runlevel

```bash
# check the current runlevel
$ runlevel

# Change to the desired runlevel
sudo init 5 # Change to GUI mode
$ sudo init 3 # Change to CLI mode
```

### If it doesn't boot

1. select recovery mode in GRUB
2. boot into single user mode
3. check and modify configuration files
4. reboot to normal runlevel

## Impact of the change in practice

### Changes that system administrators should be aware of

1. **Change script location

   ```bash
   # SysVinit environment
   /etc/init.d/nginx start
   /etc/rc.d/rc3.d/S80nginx
   
   # systemd environment
   systemctl start nginx
   /etc/systemd/system/nginx.service
   ```

2. **Change the way you check logs

   ```bash
   # The old way
   $ tail -f /var/log/messages
   $ cat /var/log/nginx/error.log
   
   # systemd way
   journalctl -f
   $ journalctl -u nginx.service
   ```

3. **Manage service dependencies
   - SysVinit: manually renumber scripts
   - systemd: specify dependencies in the unit file

### Migration Strategy

Considerations when converting an existing system to systemd:

1. **Phased transition**.
   - Create systemd unit files starting with critical services
   - Set up a period of parallel operation with existing init scripts
   - Full transition after sufficient testing

2. **Documentation and training
   - Conduct systemd training within the team
   - Document new operating procedures
   - Update troubleshooting guide

3. Adjust monitoring tools
   - Modify existing monitoring scripts
   - Establish journald log collection system

## Finalize

Run Levels and systemd targets are key concepts in Linux system administration. The change from SysVinit to systemd represents an **evolution in system philosophy**, not just a tool replacement.

### Key messages.

1. **Understand both systems**: If you're a practitioner who has to work with both legacy and modern systems, you need to know both.

2. understand the reason for the change: Adopting systemd is a necessary choice to improve performance, dependency management, and system monitoring

3. apply it in practice: know how to utilize single-user mode when troubleshooting and how to manage service dependencies

Especially in DevOps environments, containers and orchestration tools are borrowing heavily from the concepts of systemd, making these fundamentals even more important. Managing the health of systems and resolving service dependencies are key elements of modern infrastructure management.
