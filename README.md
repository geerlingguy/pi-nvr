# Raspberry Pi NVR Configuration

[![CI](https://github.com/geerlingguy/pi-nvr/actions/workflows/ci.yml/badge.svg)](https://github.com/geerlingguy/pi-nvr/actions/workflows/ci.yml)

This repository contains Raspberry Pi NVR (Network Video Recorder) configurations, for capturing and managing CCTV/IP camera streams.

Currently supported Raspberry Pi models:

  - Raspberry Pi 5
  - Raspberry Pi 4
  - Raspberry Pi Compute Module 4 (CM4)

## Current Status

I'm experimenting with many different DVR applications. From time to time, I create videos about NVR solutions, and I'll update this list with all the related videos:

  - [I spy, with my little Pi...](https://www.youtube.com/watch?v=7wkVGcdI2vk) (on Frigate + Axzez Interceptor + CM4)

See the 'NVR Solutions' section below for my thoughts on different applications, and read through the GitHub issues to see current progress in testing.

## Cameras

Currently, I use a mix of two different types of cameras in my setups:

  - [ANNKE C800 4K PoE Security Camera](https://amzn.to/48N3VXT)
  - [HIKVISION DS-2CD2122FWD-IS HD Outdoor PoE Security Camera](https://amzn.to/3S8o2Kv)

The C800 does H.265-only (at 4K), while the DS-2CD2122FW does H.264-only (at 1080p), so between the two, I have tested a variety of recording scenarios.

Both of these cameras also offer a substream at 640x360, which is more suitable for motion detection and object recognition, since you don't need full resolution for that purpose.

Setup of PoE or other IP security cameras is outside of the scope of this README. The main goal is to get cameras covering as much or as little of your property as you would like.

## Raspberry Pi Setup

For an NVR, you should use storage other than built-in eMMC (CM4 only) or microSD (Pi 5/4 or Lite CM4). For my own purposes, I'm booting a Pi off of an NVMe drive. See my articles on how to do this:

  - [NVMe Boot on Raspberry Pi 5](https://www.jeffgeerling.com/blog/2023/nvme-ssd-boot-raspberry-pi-5)
  - [NVMe Boot on Compute Module 4](https://www.jeffgeerling.com/blog/2021/raspberry-pi-can-boot-nvme-ssds-now)

You also need a lot of storage. Multiple TB of storage is best if you want any form of long-term archive.

And finally, some applications like Frigate are much more powerful with the addition of a [Coral TPU](https://amzn.to/3HlXlMp) (note: these are hard to get at list price currently).

To prep the Pi, make sure you're running the latest version of Raspberry Pi OS, can reach the Pi over SSH, and can log into it with something like `ssh pi@dvr.local` (that's the default address I'm using to test).

## Installation

Make sure you have Ansible installed (I install with Pip: `pip3 install ansible`).

Run `ansible-galaxy install --force -r requirements.yml` to make sure all dependencies are satisfied.

Copy the `example.inventory.ini` file to `inventory.ini` and change the IP address under the `[dvr]` section to the IP or hostname of your Pi, and the username after `ansible_user` to your Pi username.

Copy the `example.config.yml` to `config.yml` and modify the settings to your liking.

Run the Ansible playbook to prepare the Pi for NVR applications:

```
ansible-playbook main.yml
```

> Ideally you will have set up an [SSH key pair](https://www.raspberrypi-spy.co.uk/2019/02/setting-up-ssh-keys-on-the-raspberry-pi/) to access the Pi without entering a password. If you need to enter a password to SSH into the Pi, add `-k` after the `ansible-*` commands and Ansible will prompt you for the password when it runs.

Then run specific NVR playbook, e.g.

```
ansible-playbook frigate/main.yml
```

Be sure to have storage settings configured (e.g. any network or local mounts) prior to starting any of these applications.

### Frigate setup

The Frigate `docker-compose` configures the Frigate storage volume to be synced to `/mnt/frigate`, so you should either mount a network share in that path, or make sure `/mnt` exists locally.

For example, set up a RAID volume or a single disk (NVMe, SSD, or HDD), and made sure it was mounted at the path `/mnt/frigate` before running the playbook.

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
