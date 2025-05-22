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
* **Random Wi-Fi disconections:** If you use 2.4 GHz, Wi-Fi may dropout at a random time and fail to reconnect until wl kernel module is unloaded and reloaded. To avoid this situation, either use 5 GHz or stay within proximity of a 2.4 GHz access point, connection drop is less likely to happen if signal is strong. 5 GHz can also drop if signal is weak, but it can recover automatically. To force 5 GHz on a network:
  * Using NetworkManager: Type `nm-connection-editor` on a terminal, choose your network and in "Wireless" section set band to "A (5 GHz)".
  * Using iwd: Add this to `/etc/iwd/main.conf`:
  ```
  [Rank]
  BandModifier5GHz=100.0
  BandModifier2_4GHz=0.0
  ```
  * If you cannot use 5 GHz: When a disconnection occurs, reload `wl` module: `sudo rmmod wl ; sleep 1 ; sudo modprobe wl`.
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
* **Desktop environment / mouse freeze:** Use `intel_idle.max_cstate=1` kernel parameter. [Source](https://github.com/M4he/Linux/blob/master/Hardware/MacBookAir7%2C2.md#limiting-cstate) WARNING: This will greatly increase battery consumption.
* **Shutdown due to low battery:** Your Mac may not exhibit this issue so you can skip this step. On my Mac, high load can cause the battery to report a lower remaining charge than its actual remaining charge. This becomes a problem if reported capacity is low enough to trigger a shutdown. There are at least 2 mechanisms that can trigger a shutdown: 
  * Hidden suspend button: If battery is at %0, Mac firmware will "push" a virtual suspend button. `systemd-logind` will interpret the button push and Linux will suspend... except the Mac will do a cold boot instead of resuming. To override this behaviour, ~~open `/etc/systemd/logind.conf` and set `HandleSuspendKey` to `ignore`, don't forget to uncomment the line.~~ run `sudo systemctl mask suspend.target`.
  * UPower: To disable automatic shutdown by UPower, open `/etc/UPower/UPower.conf`, set `AllowRiskyCriticalPowerAction` to `true` and set `CriticalPowerAction` to `Ignore`. 
# Worse battery life compared to macOS 
It is possible to reduce battery life and nearly match macOS. For best battery savings, CPU Package C-State C7 is needed.
* **Checking current package state**: Open `powertop`, switch to "Idle Stats" tab. Pkg(HW) shows package states used by Linux. If a state is not %0, it is currently utilized.
* **C2 is not used:** Remove `intel_idle.max_cstate=1` from your kernel parameters. If your Mac is on low battery, let it charge then reboot it.
* **Only C2 and C3 are used, C6 not available:** Whether C6 is available or not depends on factors I could not fully determine so these instructions may or may not work. You are still recommended to apply them as some of them can reduce power consumption regardless of available C-State.
1. Add this to your kernel parameters: `acpi_osi=!Darwin`. These kernel parameters *may* also save power: `i915.enable_dc=4 i915.enable_psr=2`
2. Open `powertop`, turn on every power saving option in "Tunables" tab, turn off WakeUp for everything. Alternatively enable powertop's systemd service. Enabling power saving for Wi-Fi will save 0.5-1W.
3. Blacklist `facetimehd` module and reboot. This is not needed on low firmware versions (at least Mojave and older).
4. Use a distribution based on Arch Linux. I could not enable C6 on Nobara however switching to Arch immediately enabled C6.
5. Disconnect power supply, repeatedly (5-10 times) suspend and resume the Mac. Check if C6 is available after every resume.
6. Repeatedly reboot (3-5 times) the Mac.
7. Go back to step 5.
* **C6 is available, C7 is not**: Unload and reload `wl` module: `sudo rmmod wl ; sleep 1 ; sudo modprobe wl`. This needs to be performed every boot and resume from suspend. Also enable power saving for Wi-Fi module *again* using powertop's Tunables tab, reloading the module resets power saving option.
* **Other changes**:
  * Disable periodic Wi-Fi access point scanning: Apparently NetworkManager does not support disabling AP scanning (*please* file an issue if this is not the case) therefore the only way of disabling it is disabling NetworkManager entirely and using a standalone supplicant such as `wpa_supplicant` or `iwd`. This greatly increases Wi-Fi stability and reduces power consumption by 0.5-1W.
  * Brightness matching: Brightness levels on macOS do not directly map to brightness levels on Linux, Linux may be brighter than macOS even if brightness settings are set to same percentage. 
  * (Obsolote, left for reference) Power off the MacBook after charging it and power it back on. This power save trick will not work with a reboot, the system must start from a cold boot after being charged.
