---
layout: post
title: Arduino/AVR and platformIO
excerpt: "Getting AVR code to play nice with Clion"
modified: 2016-02-04
categories : [setup]
tags: [IDE, Arduino, Clion]
comments: false
published: false
image:
  feature: sample-image-5.jpg
  credit: EthanSlattery
---
### Motivation
One of the biggest struggles in working with hardware is getting code to play nice with an IDE, especially in Windows. Linux of course is much easier, but I spend half my time on a windows computer. At school we have only windows, and eclipse is the default IDE. I had been using the AVR plugin for eclipse successfully for a bit, but it has several things I dislike. Mainly that the settings for libraries and tools is linked to the project not the workspace, so you either need a blank project to copy or re-do the settings for every new project. It also seems to randomly break the entire workspace, meaning I have to re-create all settings from scratch.

This all came to a head last night as I opened a project for the first time since before break, and the eclipse AVR workspace was corrupted. I only had about 2 hours to work on my project and instead of re-creating an eclipse workspace and going through all that struggle I decided to look for an alternate solution...

**Other Solutions:**
 * **Text editor and command line:** Great for small things, but honestly for large projects with libraries I don't know well or other people writing code it is nice to have some sort of code completion, highlighting, and parameter suggestion. The main project I am working with now is also for computer club, and I *really* want to make it accessible to entry level students, so finding a development environment that is both friendly but doesn't hold back advanced students is a plus.
 * **Arduino IDE:** almost the same as above but no command line. It's great for what it is, but it makes it really hard for the other C++ students in the club to write code the way we learned how to (breaking up into multiple files, classes, functions, etc)
 * **Eclipse:** Drawbacks mentioned above, it is the default IDE at our school but I think the drawbacks make it too much for entry level students.
 * **MakeFile:** There are makefile projects out there for arduino, AVR, etc. This is great but again not great for beginners.
 
Last semester a fellow student showed me clion, and I *really* liked it. It is unfortunately not free... but students can get a free copy. The code completion and suggestions are almost magical, it has git integration, code analysis and suggestions, and uses makefiles as a base. All the project and workspace settings play nice with git, which is nice as well since eclipse definitely lacks this ability. I started using it for my class assignments for these reasons, especially an easy break in for makefiles. The only problem is that it doesn't support non-standard toolchains easily, and it seemed like too much work to modify it.

### Enter PlatformIO
I had seen platformIO before but it advertises itself as an "open source ecosystem for IoT development", and since IOT is not somethign I am into I never looked into it. But it came up as a way to get embedded harware working in clion in some google searches so I dug deeper. It was great! I was able to get my AVR project up and running in clion within 30 minutes.

### What you need:
 * *[Python 2.6-2.7](https://www.python.org/)* : I use 3.5, so i had to install 2.7.11
 * *[Clion](https://www.jetbrains.com/student/)* : Free for students
 * Thats it!
 
### Installing
[Here is the official quickstart guide, if my steps don't work for you](http://docs.platformio.org/en/latest/quickstart.html)

Install platforio using pip (python package manager)
~~~BASH
pip install -U platformio
~~~

