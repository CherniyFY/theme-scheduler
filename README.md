# theme-scheduler

Switch KDE Plasma's dark and light global themes at sunset and sunrise, with your own offsets. For example, dark one hour before sunset and light one hour after sunrise.

KDE's automatic switching changes the theme exactly at sunrise and sunset, with no offsets. theme-scheduler makes two changes per day, runs as a systemd user service, and gives control back to KDE if it fails.

It is a single Python file that uses only the standard library.

## Requirements

- KDE Plasma 6 (`kreadconfig6`, `kwriteconfig6`, `plasma-apply-lookandfeel`)
- A systemd user session (`systemctl --user`, `journalctl --user`, `busctl --user`)
- Python 3.11 or newer
- Optional: a notification service on the session bus, for failure alerts

## Install

Run from the repository directory:

```sh
mkdir -p ~/.local/bin
ln -sf "$PWD/theme-scheduler" ~/.local/bin/theme-scheduler
theme-scheduler install
theme-scheduler status
theme-scheduler failure-test
theme-scheduler enable
```

- `install` writes the config and the systemd units, and saves your current KDE settings once. It enables nothing.
- `status` shows the next changes, the service state, and any alerts.
- `failure-test` crashes and hangs a dry-run copy of the service to check recovery. Run it while the service is stopped.
- `enable` starts the service now and at every login.

## Configuration

`install` writes `~/.config/theme-scheduler/config.toml`:

```toml
[location]
# Example: central London. Replace with your location.
# latitude = 51.5
# longitude = -0.12

[schedule]
dark_minutes_before_sunset = 60
light_minutes_after_sunrise = 60
sun_zenith = 90.833

[themes]
# dark = "org.kde.breezedark.desktop"
# light = "org.kde.breeze.desktop"
```

- Without `latitude` and `longitude`, the location comes from KDE Night Light (`~/.config/knighttimerc`).
- Negative offsets move the change to the other side of sunrise or sunset.
- `sun_zenith` sets what counts as sunrise and sunset: 90.833 is official, 96 civil twilight, 102 nautical, 108 astronomical.
- Without `[themes]`, the themes come from KDE's default dark and light themes (`kdeglobals`).
- After editing, run `systemctl --user restart theme-scheduler`.

## Commands

| Command | What it does |
| --- | --- |
| `install` | Write the config and systemd units, save current KDE settings once. Enables nothing. |
| `enable` | Start now and at every login. |
| `status` | Show times, service state, KDE state, and alerts. |
| `times [-n N]` | Show the next theme changes. |
| `failure-test` | Crash and hang a dry-run copy of the service to check recovery. |
| `restore` | Stop and disable the service, put back your original KDE settings. |

## How it works

- While running, it turns off KDE's automatic switching and sets the dark or light theme itself.
- On a crash or a hang, the recovery unit turns KDE's automatic switching back on, sets the theme for plain sunrise and sunset times, sends a critical notification, and writes an alert that `status` shows. Your offsets stay off until you restart the service.
- Stopping the service, including at logout, hands KDE's switching back too, without offsets.
- systemd runs `theme-scheduler run` and `theme-scheduler recover`. You don't run those by hand.

## Files

- `~/.local/bin/theme-scheduler` (a symlink to this repository, per the install steps)
- `~/.config/systemd/user/theme-scheduler.service` and `theme-scheduler-recover.service`
- `~/.config/theme-scheduler/config.toml`
- `~/.config/kdeglobals` (the KDE settings it changes)
- `~/.local/state/theme-scheduler/` (your saved original settings and the alert file)

## Uninstall

```sh
theme-scheduler restore
rm ~/.local/bin/theme-scheduler ~/.config/systemd/user/theme-scheduler*.service
rm -r ~/.config/theme-scheduler
systemctl --user daemon-reload
```

## Limits

- Two changes per day, at the times computed from your location.
- No handling for polar day or night. Without a sunrise or sunset nearby, the service exits with an error.
- KDE Plasma only.

## License

MIT. See [LICENSE](LICENSE).
