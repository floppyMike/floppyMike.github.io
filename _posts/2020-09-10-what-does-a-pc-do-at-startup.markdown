---
layout: post
title:  "What does a PC do at startup?"
date:   2020-09-10 11:42:34 +0200
categories: Random-Facts
---

What a PC gets powered up it goes through a lot of processes before becomes ready for use. This is called the `Boot Process`. The `Boot Process` is controlled by the `BIOS`.

# Terms
---
## BIOS
The `BIOS` (Basic Input/Output System) is saved on the `ROM` component. The `BIOS` is integrated with the `motherboard`.
In the `BIOS` you can configure the hardware components of the system. For example it's possible to change the order at which the `drivers` are loaded or the clocking speed at which the processors should run at.
{:refdef: style="text-align: center;"}
![ROM BIOS Chip](/assets/rom-bios.jpeg)
{: refdef}

## Driver
The `drivers` is a group of files which allow the `Operating System` to communicate with connected hardware components. Without the `drivers` the `Operating System` won't be able to correctly control the devices attached to the PC.

## POST
The `POST` (power-on self-test) is a small program that checks if there are any system errors. A beep from the PC indicated that everything is in order. Other beep sequences mean that something went wrong. All sequences translate to a specific error.

## CMOS
The `CMOS` (complementary metal-oxide semiconductor) is a microchip designed to store information about the PC. It stores the system time, date and the chosen configurations of the `BIOS`.

## MBR
The `MBR` (master boot record) is the first sector of the hard drive. A sector is most of the time 512 bytes large. It holds information on how the `Operating System` should load and how the hard drive is divided.
{:refdef: style="text-align: center;"}
![MBR Sector](/assets/mbr.png)
{: refdef}

## Boot Medium
The `Boot Medium` also called the `Boot Disk` is the hard drive containing the `MBR`.

## Boot Loader
The `Boot Loader` is a program designed to find and load the `Operating System`. It is located inside the `MBR`

# The Sequence
---
1. The power button activates the `power supply`. The `motherboard` followed by the other components are powered on.
2. The `BIOS` is loaded and executed.
    1. The `POST` program is executed.
    2. The information of the `CMOS` gets loaded.
    3. Other information such as the `BIOS` manufacturer, processor features, amount of installed `RAM` and amount of `drivers` are displayed on the monitor. These days tho a lot of PCs rather display the logo of the manufacturer.
    4. The `Boot Medium` with the `MBR` is searched.
    5. The `MBR` is loaded.
    6. The `Boot Loader` is loaded to `RAM`.
3. The `BIOS` gives execution control to the `Boot Loader`.
    1. The `Operating System` is loaded.
4. The `Operating System` takes control over execution.