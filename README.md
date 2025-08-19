Bluetooth Spotify Connect with Librespot
=======================================================================

This guide sets up a Raspberry Pi Zero 2W with DietPi OS (aarch64) to stream Spotify audio 
to a Bluetooth speaker using librespot (Spotify Connect client) and BlueALSA 
(Bluetooth → ALSA audio bridge). Spotify Premium account required for Spotify Connect.

Installation Guide
==================

1. Add Bookworm repo (with safe priority pinning)
-------------------------------------------------
```bash
echo "deb http://archive.raspbian.org/raspbian/ bookworm main" | sudo tee /etc/apt/sources.list.d/armbian.list
printf 'Package: *\nPin: release n=bookworm\nPin-Priority: 100\n' | sudo tee --append /etc/apt/preferences.d/limit-bookworm
```

2. Update system & install dependencies
---------------------------------------
```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt install bluez bluez-tools alsa-utils bluez-alsa-utils libpulse0 libasound2 libgcc-s1 -y
```

3. Install librespot
--------------------
This librespot build is for aarch64:

```bash
curl -L -o librespot.tar.gz https://github.com/TomasTNunes/librespot-bluetooh-guide/releases/latest/download/librespot_aarch64.tar.gz
tar -xzvf librespot.tar.gz
chmod +x librespot
sudo mv librespot /usr/local/bin/
sudo rm -rf librespot.tar.gz
```

4. Disable PulseAudio & PipeWire
--------------------------------
To avoid conflicts, disable PulseAudio and PipeWire:

```bash
# Kill PulseAudio
pulseaudio --kill
sudo systemctl --user stop pulseaudio.socket pulseaudio.service
sudo systemctl --user disable pulseaudio.socket pulseaudio.service
sudo systemctl stop pulseaudio.socket pulseaudio.service
sudo systemctl disable pulseaudio.socket pulseaudio.service
```

```bash
# Kill PipeWire
sudo systemctl --user stop pipewire.socket pipewire.service pipewire-pulse.service
sudo systemctl --user disable pipewire.socket pipewire.service pipewire-pulse.service
sudo systemctl stop pipewire.socket pipewire.service pipewire-pulse.service
sudo systemctl disable pipewire.socket pipewire.service pipewire-pulse.service
```

5. Enable Bluetooth & BlueALSA
-------------------------------
```bash
sudo systemctl enable bluealsa
sudo systemctl start bluealsa

sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```

6. Pair Bluetooth Speaker
--------------------------
Enter the bluetoothctl shell and pair your device  
(replace XX:XX:XX:XX:XX:XX with your device’s MAC address):

```bash
sudo bluetoothctl
power on
agent on
default-agent
scan on
pair XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
quit
```

7. Test Audio
--------------
Play the ALSA test sound through your Bluetooth speaker
(replace XX:XX:XX:XX:XX:XX with your device’s MAC address):

```bash
sudo aplay -D bluealsa:DEV=XX:XX:XX:XX:XX:XX,PROFILE=a2dp /usr/share/sounds/alsa/Front_Center.wav
```

8. Run Librespot
----------------
Start Spotify Connect (replace with your device MAC address):

```bash
sudo librespot \
  --name "MySpotifyDevice" \
  --backend alsa \
  --device "bluealsa:DEV=XX:XX:XX:XX:XX:XX,PROFILE=a2dp" \
  --initial-volume 70
```
