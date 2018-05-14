# 0x60's 385 Audio Tools

## Introduction

This repo is just a collection of tools and a semi-guide to help students in ECE 385 at UIUC to integrate audio into their projects. The tools were built for the DE2-115 Development and Education Board and worked for me during Spring of 2018. I cannot guarantee that it will work on other boards but hopefully it helps.

## The Audio Codec

The audio driver in the DE2 board I had is the WM8731 from Cirrus Logic. A datasheet is available online (I found mine through [mouser](https://www.mouser.com/datasheet/2/76/WM8731_v4.9-1141834.pdf)). Given that the chip is already embedded and wired up for you, the page(s) I found useful were:

* Page 41/42 (AUDIO DATA SAMPLING RATES): for configuring the chip to work with sounds that aren't 48kHz
* Pages 49-54 (REGISTER MAP): for configuring other options

## Just tell me how to use everything

I'm assuming you have a sound file that you want to play that's in .wav form. If it isn't, use Audacity or some other software to convert into a signed 16-bit PCM Microsoft .wav file. In this example, I'm going to use ROM for my audio file, however you could any time of memory.

1. Convert your audio from .wav to .mif files. Using `python wavToMif.py input.wav output.mif`
2. Create a "On-Chip Memory (RAM or ROM)" in Qsys. Configure it as so:
	- Memory Type
		- Type: ROM (Read-only)
	- Size
		- Data width: 16
		- Total memory size: DEPTH * 2
	- Memory Initialization
		- [x] Initialize memory content
		- [x] Enable non-default initialization file
			- Select the .mif you just created.
3. Export the s1 port via a wire (For this example, I'll call mine `sound_s1`)
4. Create an "Avalon ATPLL" in Qsys. Configure it as so:
	- Frequency in: 50 MHz
	- Output freq: 0.048 MHz
5. Export the c0 port on the PLL you created via a wire (I'll call mine `Sound_clk`)
6. In your top level (or wherever you instantiate your qsys module), connect the ROM as so:
	```SystemVerilog
	.sound_s1_address(sound_address),
	.sound_s1_debugaccess(1'b0),
	.sound_s1_clken(1'b1),
	.sound_s1_chipselect(1'b1),
	.sound_s1_write(1'b0),
	.sound_s1_readdata(sound_data),
	.sound_s1_byteenable(2'b11),
	```
7. Instantiate and connect the `audio_controller_simple` module from `/simple_example` in your top level.

