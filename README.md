# Security-shutdown
Two ways to automatically shutdown your Linux PC/Laptop if there is has no power supply attached.

The reasoning is simple: You dont want anyone to be able to touch your running system if they dont actually have to stay in your home.

LUKS encryption is pretty strong, but if your Laptop is on and someone could take it somewhere, this may be weakened.

Using these little tools you can ensure they would always have to fight with LUKS and probably lose.

## Method 1: udev rule

If your system supports this, use that. Its way simpler and should work immediately.

```
sudo cat <<EOF > /etc/udev/rules.d/99-poweroff-no-power.rules
SUBSYSTEM=="power_supply", ATTR{online}=="0", RUN+="/usr/bin/shutdown -h now"
EOF
```

## Method 2: upower

```
sudo cat <<EOF > /etc/systemd/system/poweroff-no-power.service
[Unit]
Description=Shutdown on power loss
DefaultDependencies=no
After=syslog.target

[Service]
Type=oneshot
ExecStart=/usr/bin/upower -i /org/freedesktop/UPower/devices/battery_BAT0 | /bin/grep -q "state: discharging" && /sbin/shutdown -h now

[Install]
WantedBy=multi-user.target
EOF
```

## Method 3: /bin/on_ac_power

This service executes every 10sec. Should be secure enough.

```
sudo cat <<EOF > /etc/systemd/system/poweroff-no-power.service
[Unit]
Description=Detect if there is no power and shut down the system
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c "while true; do
    if ! /usr/bin/on_ac_power; then
        /usr/bin/systemctl shutdown -h now
    fi
    sleep 10
done"
EOF
```

## Method 4: ACPI
This one might be a bit specific and not work everywhere

```
sudo cat <<EOF > /etc/systemd/system/poweroff-no-power.service
[Unit]
Description=Detect if there is no power and shut down the system
After=network.target

[Service]
Type=simple
Environment=DISPLAY=:0
ExecStart=/usr/bin/acpi_listen | /usr/bin/grep --line-buffered "ac_adapter ACPI0003:00 00000080 00000000" | /usr/bin/xargs -I{} /usr/bin/shutdown -h now

[Install]
WantedBy=multi-user.target
EOF
```
