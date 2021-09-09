---
separator: <!--s-->
verticalSeparator: <!--v-->
revealOptions:
transition: 'fade'
---

# How to Build a Drum Machine with Nerves

### Hassan Khan-Shaikley

Twitter: @hassanshaikley

Github: @hassanshaikley

---

# Me

- plants
- food
- milk tea
- tinkering
- spouse

---

# What is a Drum Machine

<!-- ![](img/drum_machine_mine.png) -->
![](img/drum_machine_old.jpg)

---

# Hardware

- RPI3
- Official 7" Touchscreen
- Audio Jack Mic
- Power Supply
	- 5.25V / 3A 
- Micro SD Card

---

![](img/drum_machine.png)

---

# Software

- Nerves
- Scenic
	- Supports cross platform compilation of UI
	- Has an RPI Driver
- aplay (ships with scenic), afplay (local to mac)

---

# Meatware

- Yourself

---

# Getting Started

- Getting Started With Nerves in Scenic docs
- Plug and play

---

0. Plug in your SD card
1. `mix scenic.new.nerves rpi_drum_machine_nerves`
2. `cd rpi_drum_machine_nerves`
3. `export MIX_TARGET=rpi3`
4. `mix deps.get`
5. the firmware burn command 

---

![](img/scenic_starter.png)

---

# Sound output to jack

`:os.cmd('amixer cset numid=3 1')`
- 0: automatic
- 1: analog (headphone jack)
- 2: HDMI
- 3: None 

---

# Changing volume

`:os.cmd('amixer cset numid=1 #{percent}%')`

---

# Using static assets

`priv/static` is accessed with `Path.join(:code.priv_dir(:drum_machine_nerves), "static")`

---

# Optimization

- Benchee

---

# Questions
