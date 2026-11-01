---
title: "Using Keychron Launcher on Linux: fixing device permissions"
date: '2026-11-01'
draft: false
description: "Setting up Keychron Launcher for a Q4 HE on Omarchy, with an explanation of udev and Linux device permissions."
featured_image: '/posts/keychron-launcher-on-linux/cover.png'
tags:
  - linux
  - omarchy
  - keychron
  - udev
---

I use a Keychron Q4 HE with Omarchy, an Arch-based Linux distribution. Typing on a keyboard and configuring it through a browser sound like they should go together. If Linux already recognizes the keyboard, what else does it need?

Turns out, permission to type is not enough for [Keychron Launcher](https://launcher.keychron.com/) to configure it.

I found the fix in [Blaise Alleyne’s guide for Debian](https://blaise.ca/blog/2025/12/31/howto-get-the-keychron-launcher-working-in-debian-gnu-linux/). But before copying the rule and moving on, I took a detour to learn about udev. What does that line actually do, and why does my keyboard need it? That’s what I want to walk through here.

### TL;DR:

> Connect the keyboard in Cable mode, use a supported browser, and add a udev rule so your user can access the keyboard’s raw HID interface.

<!--more-->

## Check the connection first

Before getting into udev, check the simple things:

1. Connect the keyboard using a USB data cable.
2. Set its connection switch to **Cable** mode.
3. Open [Keychron Launcher](https://launcher.keychron.com/) in a current version of Chrome, Edge, or Opera.

These are the browsers recommended in [Keychron’s setup instructions](https://www.keychron.com/blogs/news/how-to-use-launcher-to-program-your-keyboard).

If Launcher still cannot connect, it’s time to look at Linux device permissions.

## Why typing works while configuration fails

Normal typing goes through Linux’s input handling. Launcher needs to send configuration commands to the keyboard, and that uses a different interface.

Linux exposes this through device files such as `/dev/hidraw0`. HID stands for *Human Interface Device*, and the raw interface lets applications talk directly to the hardware. The [Linux hidraw documentation](https://docs.kernel.org/hid/hidraw.html) goes into the details.

Launcher uses **WebHID** to do this from the browser. There are two permission checks here: you allow the website to access the keyboard in the browser’s connection dialog, and Linux has to let the browser open the device file.

Clicking **Connect** only takes care of the browser side. Linux still gets a say! Chrome’s [WebHID documentation](https://developer.chrome.com/docs/capabilities/hid) mentions that Linux devices may need a udev rule for this reason.

## A little bit of udev

udev handles device events on Linux. When you plug something in, it checks rules against the device’s properties and applies settings such as ownership and permissions.

Local rules live in `/etc/udev/rules.d/`. For this keyboard, we need a rule that grants access to its raw HID device files. If the distribution or an installed package already provides a suitable rule, there’s nothing to add. Otherwise, we can create one ourselves.

Why not just run `chmod` on `/dev/hidraw0`? Because that file can be recreated when you reconnect the keyboard, and next time it might be `/dev/hidraw1`. A udev rule matches the keyboard by its USB properties and applies the permissions whenever it connects.

If you want to dig into the rule syntax, here’s the [udev documentation](https://www.freedesktop.org/software/systemd/man/latest/udev.html).

## Find the keyboard’s USB IDs

First, find the keyboard:

~~~bash
lsusb | grep -i keychron
~~~

Look for the keyboard’s entry. My Q4 HE uses:

~~~text
3434:0b40
~~~

The first value, `3434`, is the vendor ID. The second, `0b40`, is the product ID.

I also have a Keychron Link-KM receiver connected, so there’s more than one Keychron device in the list. Make sure you pick the keyboard’s entry.

> **Check your own IDs before copying the rule below.** Another Keychron model may have a different product ID.

## Create a device rule

Check your primary group name:

~~~bash
id -gn
~~~

Mine is `milad`. Replace it with your group name in the command below, and adjust the USB IDs if yours are different:

~~~bash
sudo tee /etc/udev/rules.d/70-keychron-q4-he.rules >/dev/null <<'EOF'
SUBSYSTEM=="hidraw", KERNEL=="hidraw*", ATTRS{idVendor}=="3434", ATTRS{idProduct}=="0b40", MODE="0660", GROUP="milad", TAG+="uaccess"
EOF
~~~

That’s a long line for a keyboard permission. Here’s what each part does:

| Setting | Purpose |
| --- | --- |
| `SUBSYSTEM=="hidraw"` | Matches raw HID devices. |
| `KERNEL=="hidraw*"` | Matches device names beginning with `hidraw`. |
| `ATTRS{idVendor}=="3434"` | Matches the USB vendor ID. |
| `ATTRS{idProduct}=="0b40"` | Matches this keyboard’s product ID. |
| `MODE="0660"` | Grants read and write access to the owner and group. |
| `GROUP="milad"` | Assigns the device to the specified group. |
| `TAG+="uaccess"` | Enables access management for the active local user on a typical systemd desktop. |

The `70-` in the filename matters too. It puts this rule before systemd’s [seat access rule](https://github.com/systemd/systemd/blob/main/rules.d/73-seat-late.rules.in), so the `uaccess` tag is there when systemd checks it.

One detail to keep in mind: this gives local processes running with those permissions access to the device. It isn’t a permission just for the Keychron website. Matching both USB IDs keeps the rule limited to this keyboard model.

## Reload and reconnect

Reload the udev rules:

~~~bash
sudo udevadm control --reload-rules
~~~

Then unplug and reconnect the keyboard.

Reloading tells udev to read the new rule; reconnecting the keyboard gives it a device event to apply the rule to.

Return to [Keychron Launcher](https://launcher.keychron.com/), click **Connect**, and select **Keychron Q4 HE**.

With the permissions in place, Launcher should be able to connect. The rule will still be there after a reboot, so you can get back to configuring your keyboard.
