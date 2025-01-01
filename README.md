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
* Broadcom 4360 requires the "wl" module to be functional. To download the module you may temporarily use mobile phone tethering, as this Mac lacks an Ethernet port. The relevant package is `broadcom-wl-dkms` on Arch Linux and `kmod-wl` on Fedora.
* The webcam requires [facetimehd](https://github.com/patjak/facetimehd).
# Fixes and workarounds
* **Random Wi-Fi disconections:** If you use 2.4 GHz, Wi-Fi may dropout at a random time and fail to reconnect until wl kernel module is unloaded and reloaded. To avoid this situation, either use 5 GHz or stay within proximity of a 2.4 GHz access point. To force 5 GHz on a network, type `nm-connection-editor` on a terminal, choose your network and in "Wireless" section set band to "A (5 GHz)". If you cannot use 5 GHz, when a disconnection occurs, run `sudo rmmod wl && sudo modprobe wl`.
* **MacBook wakes up when lid is closed:** 
