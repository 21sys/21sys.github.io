+++
title = "Connecting an external display to Asahi Linux with DisplayLink"
date = "2025-08-18T03:40:08+02:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["asahi-linux","displaylink","dell","dell-da100","evdi","gpu","display","fedora","kde-plasma","linux","macbook","wayland"]
+++

As of now (August 2025) Asahi Linux does not support USB-C display output on the M1/M2 Macs that it supports. This leaves those Mac models that don't have a dedicated HDMI port, such as the Macbook Air M1/M2, without an option to connect an external display.

I'm on a Macbook Air M2 and this sucks. It's my only real pain point. So time for a workaround.

How about an **external USB graphics card**? (Not a passthrough adapter, a full GPU). It turns out this niche thing exists, most famously in certain Dell laptop docking stations that, as a feature, can support more displays via the docking station than the laptop's GPU is capable of outputting to.

The chipset behind this is called **DisplayLink**, [by Synaptics](https://www.synaptics.com/products/displaylink-graphics).

## DisplayLink driver on Linux

Synaptics offers [proprietery drivers for Ubuntu](https://www.synaptics.com/products/displaylink-graphics/downloads/ubuntu) and there is a community effort to [re-package these into rpms](https://github.com/displaylink-rpm/displaylink-rpm), with support for Fedora 42 aarch64 in the latest [releases on GitHub](https://github.com/displaylink-rpm/displaylink-rpm/releases).

Installation is easy:
- download the rpm (`fedora-42-displaylink-1.14.10-1.github_evdi.aarch64.rpm` worked for me on Asahi Linux Fedora 42)
- make sure you have the free + nonefree [rpm fusion repos enabled](https://rpmfusion.org/Configuration#Installing_Free_and_Nonfree_Repositories) for dependencies
- `dnf install ./fedora-42-displaylink-1.14.10-1.github_evdi.aarch64.rpm`

After installation check that the `displaylink-driver.service` is enabled (`dnf enable --now displaylink-driver.service`) and that the evdi driver is loaded (`lsmod | grep evdi` and if needed `modprobe evdi`).

You should now see additional evdi cards:

```text
$ ls -la /dev/dri/by-path/
platform-206400000.gpu-card -> ../card1
platform-206400000.gpu-render -> ../renderD128
platform-evdi.0-card -> ../card0
platform-evdi.1-card -> ../card3
platform-evdi.2-card -> ../card4
platform-evdi.3-card -> ../card5
platform-soc:display-subsystem-card -> ../card2
```

After attaching the display, you should be able to press `SUPER` + `P` and extend or mirror the display.

(Just to say, this worked for me on the default Wayland/Kwin/KDE Plasma 6 setup of Asahi Fedora Linux 42. No changes needed).

## The hardware choice

For me, with the wrong DisplayLink adapter nothing worked despite extenisve troubleshooting - and with the right hardware everything worked instantly.

- **bad hardware choice**:  This is the adapter that I first tried and that I did **NOT** get to work: *[ABLEWE B0D1C7RM1G usb2hdmi-DE-FBA](https://www.amazon.co.uk/dp/B0D1C7RM1G)* - **don´t buy**
- **good hardware choice:** This is the adapter that **WORKS WELL** for me: *[Dell DA-100](https://dl.dell.com/Manuals/all-products/esuprt_electronics/esuprt_docking_stations/dell-universal-dongle-da100_User's%20Guide_en-us.pdf)*
  - only the old DA-100 variant with USB-A connector seems to have DisplayLink. Don't buy the successor version with USB-C connector.

![Dell DA-100](/images/connecting-an-external-display-to-asahi-linux-with-displaylink/dell_da100.png)

## So, how is it?

With the `Dell DA-100` (attached to a regular USB-A to USB-C adapter) and the `displaylink-rpm`, I'm now getting FullHD HDMI output on Asahi Linux on my Macbook Air M2.

The performance isn't too awesome, but it's better than I expected and certainly good enough for day to day work in non-graphics/3D applications or when I need to attach to a projector for a presentation.

(I still have to try and see how well it works for a 3D heavy application like Blender - probably not too good?).

While certainly not made for gaming, it's actually good enough to play [SuperTuxCart](https://supertuxkart.net/) in FullHD on an external monitor.

![supertuxcart game on an external display with glxgears on the internal display](/images/connecting-an-external-display-to-asahi-linux-with-displaylink/dispaylink_demo.gif)

