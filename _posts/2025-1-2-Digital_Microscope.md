---
title: How I Built a DIY Digital Microscope
tags: Raspberry-Pi Microscope 3D-Printed Tool
layout: article
show_edit_on_github: false
license: true
---

Like most of my projects, I got my inspiration for this one by through Youtube. Specifically, I saw this video ([Brauns CNC](https://www.youtube.com/watch?v=jzcHGjFiR0o)), and I was immediately hooked on the idea of building my own digital microscope. Although I had never experimented with using a camera module on a Raspberry Pi, I dove in headfirst.

---
## Materials
(Required)
 - Raspberry Pi Zero
 - Raspberry Pi Camera
 - Macro lense
 - 3D Printer + Filament
 - 5 volt,  1-2 amp Micro USB power supply
 - HDMI Mini to HDMI adapter

<br>
Like always, .step and all other files will be provided on this project's [Github Page](https://github.com/Ballistyxx/digital-microscope) so you can print and create your own!

---

## The Build

This whole project revolves around the Raspberry Pi Zero. Its a pocket-sized (65x30mm) SBC with 512MB of ram and the BCM2835, a SOC made by Broadcom. But the part that makes this SBC essential for this project, besides having a massive community online for debugging (which we'll be using plenty later on) is the presence of a CSI connector on the bottom of the Pi. This lets us connect a camera directly to the processor, rather than have to rely on a less secure USB connection, which in my experience is susceptible to disconnections and reliability issues. The Raspberry Pi Zero 2 can also run either a 32-bit or 64-bit distribution of Raspberry Pi OS right out of the box, which comes with built-in support for using a Raspberry Pi camera. 

Everything starts with booting up the Pi and being able to access it. First, I flashed a 32-bit version of Raspberry Pi OS to an 8GB SD card, ensuring that I selected the 'Buster' version. This version of Raspberry Pi OS is based off of Debian 10 'Buster', which still comes pre-installed with support for `raspivid`, the command that I used to start the camera stream. Additionally, I made sure to specify my home network's SSID and password, as well as explicitly enable SSH.

After successfully flashing and mounting the filesystem to the SD card, I connected my Raspberry Pi Camera to my board using a CSI to Mini-CSI ribbon adapter, and plugged in my Micro USB power supply to the PWR-in plug **(NOT the USB plug, as that will fry the board upon too much power draw.)**

### Software

After letting the Pi boot for the first time, I opened the command line on my laptop to begin a SSH session. Since both my laptop and the Raspberry Pi are connected to the same wifi network, we can run a simple command to access our Pi from our laptop. Using [Angry IP Scanner](https://angryip.org) to scan my network for the ip address of my Pi, I ran the command `ssh pi@192.168.1.xxx`, where `xxx` was replaced with the IP address of the Pi, prompting a first-time signin prompt.  After confirming the authenticity of the host by responding `yes` to the message on the screen, I was prompted one last time to enter my password, which was still the default `raspberry`. With that, I was greeted with the command line of the raspberry pi.

Now that my Pi was up and running, it was relatively simple to enable the camera using `raspivid`. The command that I settled on using has six arguments:

- `-t 0`: sets the time until starting the camera stream to zero seconds
- `-w 1920`: sets the horizontal resolution of the camera stream to 1920 pixels
- `-h 1080`: sets the vertical resolution of the camera stream to 1080 pixels
- `-fps 30`: caps the frames per second of the stream to 30
- `-sa 10`: sets the saturation to 10 (on a scale from 0 to 100)
- `-rot 180`: rotates the image 180°; this was necessary on my setup, but depending on the orientation of your camera/screen, this may not be necessary

Alltogether, the final command ended up like this:

`raspivid -t 0 -w 1920 -h 1080 -fps 30 -sa 10 -rot 180`

The file that runs this command is included in this project's github repository, linked at the end.

Troubleshooting the camera connection took the majority of my time on this project. Thankfully, there is a plethora of resources online that helped me troubleshoot my problems. Although I can't go into all the nuanced detail of troubleshooting my camera here, I can offer some advice to anyone attempting to do the same:

- Make sure that the legacy camera stack is enabled; On Debian Buster and earlier distros, it is required for `raspivid`, but on later distros, it may not be required with newer libraries like `libcamera` or `rpicam`. Run the command `sudo raspi-config` to access the configuration CLI for your Pi, navigate to `"3 Interface Options"`, and enable the legacy camera stack.
- Check for any scratches on the ribbon connector; Those connectors are nothing more than glorified flexible PCBs, and as such are extremely delicate. 
- Use the command `vcgencmd get_camera` to get some details on the connection status of your camera
    - A status of `supported=1` means that the camera is recognized as a camera sensor that is compatible with the Raspberry Pi Camera Stack
    - A status of `detected=1` means that the camera is detected by the Raspberry Pi Camera Stack
    - A status of `libcamera interfaces=1` means that the camera is being properly communicated with by the Raspberry Pi Camera Stack.
    - If all three statuses return values of 1, then the camera will work as intended.

### Hardware

Once I verified that my camera worked, I moved on to designing the 3D model to mount the Pi to. After mulitple major revisions, I settled on a design that isn't perfect, but works well enough. It consists of a rack and pinion mechanism, with a locking knob to keep the distance between the camera and the target constant. The Raspberry Pi and the camera assembly were both mounted to the rack, so that the focus of the camera could be adjusted for objects of different height.

Perhaps the hardest part of the entire design to get correct was the geometry of the camera + lense assembly. I went through nearly a dozen revisions, tweaking the tolerances and dimensions of the model until I got one that let the lense friction-fit snugly in front of the camera sensor, which sat directly above it, held down by some M2 screws threaded into the plastic directly.

In addition to a knob that adjusts the height of the camera, I also implemented a locking mechanism using a 3D-printed thread. As a bolt was threaded through a hole in the side of the base, it pressed up against the gear, adding friction and making it impossible for the microscope to lose its focus.

My hardware implementation didn't stop at just 3D prints, however. I also implemented two electrical features, powered off of the GPIO of the Raspberry Pi.
1. Soldering a 2-pin male DuPont connector to pins 4 and 6 (5V and GND, respectively) let me power a LED to shine down on the object being inspected. Having a bright workspace is important, especially since zoom lenses allow less light to pass through them to the camera sensor.
2. Soldering another 2-pin male DuPont connector to pins 39 and 40 (GND and GPIO21, respectively) let me connect a button that could be detected using software on the Raspberry Pi. This button is used to safely shut down the microscope when it isn't in use before I disconnect the power.

At long last, my project was completed. Using the 15x zoom lense inserted in front of the camera sensor, I'm able to see incredible detail on my custom circuit boards. Using my microscope, I've been able to easily spot the wire-bonds on 0402 SMD LEDS, and inspect my solder joints for continuity. This project has been one of my most-used tools in my lab, and it has helped me in ways I didn't know it could until I had it at my disposal. 

---

## Conclusion

Overall, I'm very pleased with how this project turned out, but thats not to say that there isn't lots of room to improve. This was my first foray into building something like this, after all.

Looking back after using this scope for over a year, I've come up with a few things I would certainly do differently if given the chance to do this again:

1. I could have made the maintenance process **much** easier by using heated inserts and a multilayered design to sandwich the camera and lense between slabs of plastic to keep them secure, rather than using a slip fit for the lense, and screwing the camera directly into the plastic.

2. The base that I built for the microscope is functional, but much too flexible for my taste. Since the focus of the camera is heavily dependent on the distance of the lense from the top of the object, as the base flexes from the weight of my hand, it can make the final image blurry after the position is locked. I'll definitely be redesigning the base to prevent it from flexing in the future.

Despite those issues, my microscope is still extremely versatile and I've used it more than most tools in my lab. I'm excited to continue using it in the future, and I've already gotten some ideas to improve the durability and functionality of my current design. Look out for another blog post on another tool I've made for my lab, and thank you for reading!

<!--more-->
