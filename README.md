# Raspberry Pi NVR Configuration

This repository contains Raspberry Pi NVR configurations so a Pi 4 or CM4 can be used as an NVR, or Network Video Recorder, for capturing and managing CCTV/IP camera streams.

Currently I'm experimenting with many different DVR applications.

See the 'NVR Solutions' section below for my thoughts on different applications, and read through the GitHub issues to see current progress in testing.

## Raspberry Pi Setup

For an NVR, you should use storage other than built-in eMMC (CM4 only) or microSD (Lite CM4 or Pi 4 model B). For my own purposes, I'm booting a Pi off of an NVMe drive, using the new [native NVMe boot option](https://www.jeffgeerling.com/blog/2021/raspberry-pi-can-boot-nvme-ssds-now) on the CM4.

You also need a lot of storage—multiple TB of storage is best if you want any form of long-term archive.

And finally, some applications like Frigate work great if you add on something like Google Coral TPU via USB.

To prep the Pi, make sure you are running the latest version of Raspberry Pi OS, can reach the Pi over SSH, and can log into it with something like `ssh pi@dvr.local` (that's the default address I'm using to test).

## Installation

Make sure you have Ansible installed (I install with Pip: `pip3 install ansible`).

Copy the `example.inventory.ini` to `inventory.ini` and change the IP address under the `[dvr]` section to the IP or hostname of your Pi, and the username after `ansible_user` to your Pi username.

Run the Ansible playbook to prepare the Pi for NVR applications:

```
ansible-playbook main.yml
```

> Ideally you will have set up an [SSH key pair](https://www.raspberrypi-spy.co.uk/2019/02/setting-up-ssh-keys-on-the-raspberry-pi/) to access the Pi without entering a password. If you need to enter a password to SSH into the Pi, add `-K` after the `ansible-*` commands and Ansible will prompt you for the password when it runs.

Then run specific NVR playbook, e.g.

```
ansible-playbook frigate/main.yml
```

Be sure to have storage settings configured (e.g. any network or local mounts) prior to starting any of these applications.

### Frigate setup

The Frigate `docker-compose` configures the Frigate storage volume to be synced to `/mnt/frigate`, so you should either mount a network share in that path, or create a local volume there.

In my case, I either set up a RAID volume or a single disk (NVMe, SSD, or HDD), and made sure it was mounted at the path `/mnt/frigate` before running the playbook.

> There are a number of great guides for Frigate out there, but I am indebted especially to [this Frigate guide from Simplepush](https://www.simplepush.io/blog/frigate-nvr-push-notification-guide#run-mosquitto-mqtt-in-docker) for a broad understanding of all Frigate's moving parts.

### Shinobi setup

After the playbook completes, visit the URL of your NVR, at the `/super` path, e.g. `http://dvr.local:8080/super`. The default login credentials are:

  - Email: `admin@shinobi.video`
  - Password: `admin`

## Raspberry Pi NVR Solutions

Here are my notes on different options for DVR/NVR/CCTV solutions that could run on the Pi:

  - Shinobi (https://shinobi.video)
    - Guide: https://www.heystephenwood.com/2018/08/shinobi-on-raspberry-pi-3-b.html
    - Seems popular, well supported
    - Docker image only available for arm32v7 right now (see: https://github.com/geerlingguy/pi-nvr/issues/1)
  - Frigate (https://docs.frigate.video)
    - Use with Google Coral TPU... https://docs.frigate.video/hardware
    - Runs in Docker, though storage configuration can be annoying
    - Seems to integrate *really* well with Home Assistant
    - Requires an MQTT server elsewhere?
  - ZoneMinder (https://zoneminder.com)
    - Pi guide: https://wiki.zoneminder.com/Raspberry_Pi_4_-_Raspbian
    - Old and trusted... but also harder to configure
    - No official Docker images for arm64v8, but [this one](https://registry.hub.docker.com/r/nardo86/zoneminder) seems okay.
  - MotionEyeOS (https://github.com/motioneye-project/motioneyeos)
    - Latest release from October 2020... which is kinda old. Dev work to get on Python 3...
  - Orchid Core from IPConfigure (https://www.ipconfigure.com/download)
    - Unlimited cameras, but they do like you to use licensed version maybe. Not open source?
  - PiNVR (https://www.pinvr.net)
    - Cool logo, but beta not open, Forums look spammed out
  - On-NAS Surveillance Center (ASUSTOR)
    - Even though it's proprietary, I wanted to see if it was plug-and-play
    - Safari plugin doesn't seem to work with latest Safari

## Author

This project was created in 2022 by [Jeff Geerling](https://www.jeffgeerling.com).
