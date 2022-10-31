# Raspberry Pi Setup

I'm using an Pi 4B, with the Raspberry Pi OS (64-bit) image.

This includes the LXDE desktop environment.

SSH into the PI and run `sudo raspi-config`.
We want to enable auto-login for the desktop environment.

Select `1: System Options`, `S5: Boot / Auto Login`, `B4: Desktop Autologin`.

Select `2: Display Options`, `D4: Screen Blanking`, `No`.

Install unclutter to hide cursor, and xdotool for fake key/mouse events: `sudo apt-get install unclutter xdotool`.

Run `sudo nano boot.sh`, and add the following script:

```bash
#!/bin/bash

sed -i 's/"exited_cleanly":false/"exited_cleanly":true/' /home/pi/.config/chromium/Default/Preferences
sed -i 's/"exit_type":"Crashed"/"exit_type":"Normal"/' /home/pi/.config/chromium/Default/Preferences

unclutter &
chromium-browser --noerrdialogs --disable-infobars --kiosk https://assets.mattcowley.co.uk/voice/ &

sleep 15
eval `xdotool search --onlyvisible --name chromium getwindowgeometry --shell`
xdotool search --onlyvisible --name chromium mousemove --sync $(( $WIDTH / 2 )) $(( $HEIGHT / 2 ))
xdotool search --onlyvisible --name chromium click 1
```

Save, and then run `sudo chmod +x boot.sh`.

_This clears the last exit state of Chromium, and then starts it in kiosk mode. After 15s of waiting for Chromium to start, it moves the mouse to the centre of the screen and clicks._

Run the following to create our auto-start script:

```bash
mkdir -p /home/pi/.config/lxsession/LXDE-pi
cp /etc/xdg/lxsession/LXDE-pi/autostart /home/pi/.config/lxsession/LXDE-pi/
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

Add the following config, leaving the default config before it:

```bash
@xset s off
@xset -dpms
@xset s noblank

@/home/pi/boot.sh
```

_This disabled the screensaver and energy saving features, and then runs our boot script._

Run `sudo nano /home/pi/.config/lxpanel/LXDE-pi/panels/panel` to open the LXDE panel config.
Within the `Global {}` block, ensure the following options are set:

```ini
autohide=1
heightwhenhidden=0
notifications=0
```

_This hides the LXDE menu panel, and disables notifications from it._

Run `/home/pi/.config/pcmanfm/LXDE-pi/desktop-items-0.conf` to open the LXDE desktop config.
Ensure the following options are set:

```ini
desktop_bg=#000000
desktop_shadow=#000000
wallpaper_mode=color
show_documents=0
show_trash=0
show_mounts=0
```

_This sets the desktop background to be black, and hides the desktop icons._
