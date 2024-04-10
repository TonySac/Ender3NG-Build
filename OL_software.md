---
title: "Software Configuration :building_construction:"
layout: default
nav_order: 7
has_children: true
---

:construction: 
I haven't got this far yet...
:building_construction: 

# Software Configuration
{: .no_toc }

<details closed markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

I already use Klipper with Mainsail on my current printers so it was a natural choice for this project. My installation is running on a [Raspberry Pi 3A+](https://www.raspberrypi.com/products/raspberry-pi-3-model-a-plus/). As I prefer to start with a bare bones system and cherry pick what I need, I flashed Raspberry Pi OS Lite then used [KIAUH](https://github.com/dw-0/kiauh) for a "manual" install.

{: .tip }
:bulb: I have a tutorial for [Baking a Raspberry Pi](https://themakermedic.com/posts/Pi-Baking_a_Pi/) in my other blog.

Since so much information already exists between the [KIAUH](https://github.com/dw-0/kiauh) docs along with the [Klipper](https://www.klipper3d.org) and [Mainsail](https://docs.mainsail.xyz) pages I suggest referring to them for more details on the installation process and methods. Rather than rewrite existing good documentation, I will focus this section on configurations specific to the Ender 3 NG and my personal machine.


# Config File (printer.cfg)

