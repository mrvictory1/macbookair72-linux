# macbookair72-linux
Tweaks and workarounds for Linux on MacBookAir7,2. Also known as MacBook Air (Early 2015) or MacBook Air (13 inch, 2017). Notable hardware includes:
* Intel Core 5th generation i5 or i7 processor
* Intel HD 6000
* Intel Broadwell-U audio
* 1440x900 glossy screen
* Broadcom 4360 Wi-Fi / BT card
* 720p FaceTime HD camera

Yes, Apple was selling a 5th gen Intel Core [until 2019](https://everymac.com/systems/apple/macbook-air/specs/macbook-air-core-i5-1.8-13-2017-specs.html). Since this is a rather old MacBook with an Intel processor I expected the Mac to work well with Linux, the reality is this Mac requires several tweaks which are mentioned in different websites to be fully functional.
# Getting hardware to work
* Broadcom 4360 requires the "wl" module to be functional. To download the module you may temporarily use mobile phone tethering, as this Mac lacks an Ethernet port. The relevant package is `broadcom-wl-dkms` on Arch Linux and `kmod-wl` on Fedora. You may need to blacklist any other driver for wl to take control.
* The webcam requires [facetimehd](https://github.com/patjak/facetimehd).
* Keyboard backlight doesn't work on Nobara Linux 40 but it did on winesapOS, I haven't investigated why.
# Fixes and workarounds
* **Random Wi-Fi disconections:** If you use 2.4 GHz, Wi-Fi may dropout at a random time and fail to reconnect until wl kernel module is unloaded and reloaded. To avoid this situation, either use 5 GHz or stay within proximity of a 2.4 GHz access point, connection drop is less likely to happen if signal is strong. 5 GHz can also drop if signal is weak, but it can recover automatically. To force 5 GHz on a network, type `nm-connection-editor` on a terminal, choose your network and in "Wireless" section set band to "A (5 GHz)". If you cannot use 5 GHz, when a disconnection occurs, run `sudo rmmod wl && sudo modprobe wl`.
* **MacBook wakes up when lid is closed:** Disable lid wakeup with the following systemd unit:
```
[Unit]
Description=MacBook suspend bug fix
[Service]
Type=oneshot
ExecStart=/bin/sh -c "echo XHC1 > /proc/acpi/wakeup && echo LID0 > /proc/acpi/wakeup"
[Install]
WantedBy=multi-user.target
```
Closing the lid to suspend the Mac will work if it is enabled on your system, for waking up you will need to press the power button.
* **Hissing / screeching sound:** When playing back voice or a high pitch sound, you may hear sound distortions. To get rid of distortions, set an equalizer with EasyEffects. I am using 32 bands; bands 19, 20 and 21 (1.1 kHz, 1.4 kHz and 1.7 kHz respectively) have -12.56 db gain.
* **Desktop environment / mouse freeze:** Use `intel_idle.max_cstate=1` kernel parameter. [Source](https://github.com/M4he/Linux/blob/master/Hardware/MacBookAir7%2C2.md#limiting-cstate)
* **Worse battery life compared to macOS:** On Linux with %70-80 battery health, battery lasts 4 and a half hours with light usage. On idle with %15 screen brightness, keyboard backlight off, Wi-Fi enabled, power profile set to  "Power Save", energy usage is 8W as reported by powertop. On macOS Mojave under same conditions, battery usage is 3W. Here are some modifications that greatly reduce power draw on idle:
  * ~~Blacklist kernel module b43~~ UPDATE: This does nothing
  * Use kernel parameters `i915.enable_dc=4 i915.enable_psr=2 acpi_osi=!Darwin`.
  * ~~Use powertop, turn on every power saving option in "Tunables" tab, turn off WakeUp for everything. Alternatively enable powertop's systemd service.~~ UPDATE: Don't do this either, this decreases 5 GHz network stability and speed even though there are noticeable battery savings.
  * Power off the MacBook after charging it and power it back on. This power save trick will not work with a reboot, the system must start from a cold boot after being charged.

acpi_osi=!Darwin reduces power usage by 1-1.5W, ~~blacklisting b43 reduces by 1-2W~~, powertop tweaks reduce usage by 0.4-0.5W. i915 parameters are there to enable aggressive power saving, they may or may not have an effect. Power cycle trick saves 0.5-1W.
These modifications are not enough to bring Linux battery life on par with macOS but help with bridging the gap. Most notably CPU package states C6 and above is not available, highest state is C3. 
