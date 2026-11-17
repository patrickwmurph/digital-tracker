---

title: System Automation  
description: Automating display management and scheduled operation of a Raspberry Pi digital calendar.  
tags:

- linux
    
- automation
    
- raspberry-pi
    

---

# System Automation

## Overview

The Digital Tracker is designed to function as a dedicated productivity display rather than a conventional desktop computer. Although the system runs a full Ubuntu desktop environment, routine interactions with the operating system should require minimal manual intervention.

To accomplish this, I implemented several Linux automations to manage display orientation, power scheduling, and manual controls. Display rotation was particularly important because the VESA-mounted monitor can be physically repositioned between landscape and portrait orientations.

These features allow the Raspberry Pi to remain operational while the external monitor is activated only when needed.

## Design Objectives

The automation system was developed around four objectives:

1. **Support flexible display orientation.** Allow the monitor's software orientation to match its physical position without navigating Ubuntu's display settings.
2. **Minimize manual interaction.** Automate repetitive actions such as activating and deactivating the monitor during a scheduled morning planning session.
3. **Preserve manual control.** Allow the monitor to be activated, deactivated, or rotated independently of its scheduled operation.
4. **Separate display operation from system operation.** Keep the Raspberry Pi running when the monitor is inactive, avoiding unnecessary system shutdowns and restarts.

## System Environment

|Component|Configuration|
|---|---|
|Computer|Raspberry Pi 5 (4 GB)|
|Operating system|Ubuntu 26.10|
|Desktop environment|GNOME|
|Display protocol|Wayland|
|External display|LG monitor (`HDMI-1`)|
|Mounting|VESA-mounted adjustable arm with swivel and rotation|
|Automation|Shell scripts, system scheduling, GNOME keyboard shortcuts|

## Implemented Features

### [[Monitor Power Scheduling]]

The monitor is configured to activate automatically at **6:00 AM** and deactivate at **9:00 AM** each day.

This creates a dedicated morning planning window without requiring the Raspberry Pi itself to shut down.

The implementation uses a shell script for display control and scheduled execution to trigger the appropriate commands.

### [[Monitor Rotation]]

The external monitor can switch between landscape and portrait-left orientations using a keyboard shortcut.

Because the monitor is mounted on a rotating VESA arm, its physical orientation can be changed independently of the operating system. The rotation shortcut allows the digital display to match the physical orientation without manually navigating through display settings.

### [[Keyboard Shortcuts]]

Custom keyboard shortcuts provide manual control of the display independently of its scheduled operation.

These include commands for display rotation, sleep, and activation.

### [[Systemd Timers]]

Systemd provides a mechanism for running services according to defined schedules.

This section examines service and timer configuration, command execution, and testing as applicable to the implemented display schedule.

## Architecture

The automation separates user interaction, scheduling, and display control.

- **Scheduled actions** initiate commands at predetermined times.
    
- **Keyboard shortcuts** provide manual control.
    
- **Shell scripts** encapsulate reusable display operations.
    
- **GNOME/Wayland** manages the display session in which those operations execute.
    

This separation makes it possible to modify the schedule or keyboard shortcuts without redesigning the underlying display-control commands.

## Related Documentation

- [[System Architecture]]
- [[Dashboard Overview]]
- [[Display Configuration]]
- [[Ubuntu Installation]]