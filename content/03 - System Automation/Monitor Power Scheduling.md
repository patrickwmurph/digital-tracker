---

title: Monitor Power Scheduling  
description: Automating daily monitor activation and deactivation using Linux.  
tags:

- linux
    
- automation
    
- bash
    
- systemd
    

---
# Monitor Power Scheduling

## Overview

The Digital Tracker is designed primarily for morning task review rather than functioning as an always-on display. To support this, I configured the monitor to automatically activate at **6:00 AM** and deactivate at **9:00 AM** each day.

These actions control the display rather than shutting down the Raspberry Pi. This allows Obsidian and the underlying operating system to remain active while minimizing unnecessary screen usage.

The automation also preserves manual control, allowing the display to be activated or deactivated outside its scheduled operating window.

## Design Considerations

Three requirements shaped the implementation:

1. **Scheduled operation.** Automatically activate and deactivate the monitor at predetermined times.
    
2. **Independent display control.** Manage the monitor without shutting down the Raspberry Pi or terminating the Ubuntu session.
    
3. **Manual override.** Allow the monitor to be used outside the scheduled window without modifying or disabling the automation.
    

## Implementation

The solution separates **display control** from **scheduling**, allowing each component to operate independently.

### Display Control Using D-Bus

Ubuntu's GNOME desktop uses Mutter as its window manager and Wayland compositor. Rather than relying on traditional X11 utilities such as `xrandr`, I used `busctl` to communicate with Mutter through D-Bus.

D-Bus is an interprocess communication system that allows applications and services to communicate. The `busctl` utility provides a command-line interface for interacting with these services.

The display-control script is located at `~/scripts/monitor-power.sh`:

```
#!/bin/bash

case "$1" in
  on)
    MODE=0
    ;;
  off)
    MODE=1
    ;;
  *)
    echo "Usage: monitor-power.sh {on|off}"
    exit 1
    ;;
esac

busctl --user set-property \
  org.gnome.Mutter.DisplayConfig \
  /org/gnome/Mutter/DisplayConfig \
  org.gnome.Mutter.DisplayConfig \
  PowerSaveMode i "$MODE"
```

The script accepts a single argument, `on` or `off`, and translates it into the appropriate `PowerSaveMode` value.

|Value|Behavior|
|---|---|
|`0`|Display on|
|`1`|Display standby|

Although the script refers to the second action as `off`, it requests standby rather than physically disconnecting power from the monitor.

The `--user` option ensures the command communicates with the user's D-Bus session, where GNOME manages display configuration.

### Scheduled Execution

Two systemd user timers trigger the display-control script through their corresponding services.

|Timer|Schedule|Action|
|---|---|---|
|`monitor-on.timer`|Daily at 6:00 AM|Executes `monitor-power.sh on`|
|`monitor-off.timer`|Daily at 9:00 AM|Executes `monitor-power.sh off`|

Both timers are enabled and registered with systemd.

The associated services use `Type=oneshot`, executing the script once rather than maintaining a continuously running background process.

The timer configurations also specify `Persistent=false`, preventing missed scheduled events from being automatically replayed when an inactive timer is subsequently activated.

The complete service and timer configurations, including setup instructions, are documented in [[Systemd Timers]].

### Manual Operation

The monitor can be controlled independently of its scheduled operation.

To deactivate the display:

```
~/scripts/monitor-power.sh off
```

To reactivate the display:

```
~/scripts/monitor-power.sh on
```

These commands can also be assigned to GNOME keyboard shortcuts, providing convenient control without modifying the schedule.

See [[Keyboard Shortcuts]].

## Testing and Troubleshooting

### Initial Validation

Before configuring the recurring schedule, I validated the display-control commands using a short 15-second test.

This confirmed that the monitor could enter standby while the Raspberry Pi continued operating normally.

### Display Reactivation

During testing, reactivating the monitor did not always immediately restore the visible Ubuntu desktop, despite the Raspberry Pi remaining operational.

Pressing `Ctrl + Alt + T` restored the display.

This highlighted the importance of testing display-management commands within the actual GNOME/Wayland environment rather than assuming behavior would match traditional X11-based systems.

### Timer Verification

The systemd timer configuration was verified using:

```
systemctl --user is-enabled monitor-on.timer monitor-off.timer
```

Both timers returned `enabled`.

The upcoming scheduled executions were also confirmed using:

```
systemctl --user list-timers --all
```

At the time of verification, systemd showed the next activations scheduled for 6:00 AM and 9:00 AM on November 17, 2026.

These checks confirm the timers are enabled and scheduled. Successful unattended execution should also be verified through service logs after a scheduled run.

## Outcome

The resulting configuration provides a dedicated morning planning window without requiring the Raspberry Pi to be shut down or restarted each day.

By separating display-control logic from scheduling, the implementation remains reusable for keyboard shortcuts and future automation while preserving manual access to the dashboard.

## Related Documentation

- [[System Automation]]
    
- [[Systemd Timers]]
    
- [[Keyboard Shortcuts]]
    
- [[Display Configuration]]