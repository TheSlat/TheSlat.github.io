---
layout: post
title: Rookie Mistake - Arduino Strings
excerpt: "after a few re-flashes and counting LED blinks, I realized I had been away from embedded for too long and had made a rookie mistake"
modified: 2016-03-16

categories : [blog]
tags: [C++, Embedded, Troubleshooting]
comments: false
---
### Project Overview ###
  My girlfriend, a mutual friend who is also a biologist, and I were out enjoying a hike recently and they were talking about how a cheap and easy data logger would be nice. The commercial ones that scientists usually use can be very expensive and hard to use. They also seem to have a fairly high failure rate based on the conversations I have overheard and the field expeditions I have tagged along on. Data loggers are a pretty basic embedded project, pretty common for beginners and there are a lot of examples out there on the internet. Seems like a great project to work on, and though its not super complex it is a good opportunity to learn about making a device that is easy to use. If it ever pans out a biologist would need to be able to use it without coding or compiling, and it would have to survive the field! Spring break just started and my girlfriend was at a bird banding workshop all day so I thought I would take a stab at it.

### Microcontroller ###
  I chose the [esp8266](http://esp8266.net/) since it has a low power sleep mode, built in WiFi features I might use, is cheap and tiny, and has program memory eliminated the need for a separate microcontroller to run my logic. It also is a supported platform in [platformIO](http://platformio.org/), which is my new favorite thing. Specifically i used the [Sparkfun Thing](https://www.sparkfun.com/products/13231) since it has all the pins accessible, a built in antenna, and a built in LiPo charger. This will make limited field testing much easier before I eventually lay out my own board. The adafruit [feather HUZZAH](https://www.adafruit.com/products/2821) also looks SUPER cool but was out of stock, and still is as of today. The feather has ["wings"](https://www.adafruit.com/feather) which are stackable board with other modules like real time clocks and SD cards which is really nice. But *I do* have more than a few RTC modules and SD card breakouts laying around so an old school jumper wire and breadboard nest will work for now.

### First Issue (simple) ###
  With [platformIO](http://platformio.org/) I was up and blinking in about 5 minutes using a project in [Clion](www.jetbrains.com/clion/). Hooking up my SD card breakout to the esp8266 and running a test program gave me issues though. The SD library complained that my "Architecture or board not supported", specifically from the defined Arduino board architectures in Sd2PinMap.h in the SD card library. Since the esp8266 Arduino libraries come with their own version of the SD library, and even provide an example this error was a little confusing at first. I thought maybe my libraries were out of date so I ran `platformio update` in BASH (*so cool*) and all my libraries managed by platformIO were updated. This did not fix the issue but I did notice that I had an additional SD library I had installed for a previous project (more specifically #161 the Adafruit SD library). I uninstalled this library `platformio lib uninstall 161` and everything worked immediately. I will look into this more later but obviously an esp8266 project should use its SD.h *not* the one in the other library. Probably need to look at my cmake file, but with platformIO uninstalling and installing libraries is so easy this fix was quick and kept me moving for now.

<figure>
	<img src="/images/datalogger_rookie-mistake.jpg">
	<figcaption>The setup as thrown together</figcaption>
</figure>

### The Main Problem ###
  As an initial defense I have not done any serious embedded development in a year. I am taking a crazy class load to transfer to university ASAP, and even thought I program a LOT of C++ it is all for Compute Science classes. Writing a skip list and binary search tree for a data structures class or developing a Qt app is *very* different from the kind of code I need for a microcontroller. The rookie mistake I am about to describe is a direct result of the style I developed in regular C++ classes. But hey, this is why personal projects are so important and why I was excited to get a little time to work on this :)

  When You want to write a function that prints some text, you generally want to avoid just printing inside the function (with cout for example). We want our code to be more generic, so it can be reused in more situations without modification. In this project I am creating a JSON array, and initially I am outputting the text over the serial line so I can read and debug the output in my terminal, and then once it is nice I want to write that text to a file on the SD card. I shouldn't have to re-write the whole function just to change the destination of the text! the normal was to do this in C++ is to either return a string or pass in a reference to a stream, as shown below:

	{% highlight C++ %}
	// Return a string
	std::string print()
	{
		std::ostringstream output;
		output << "some text";
		output << "Some more text!" << number << std::endl;
		return ouput.str();
	}

	// Print via a stream reference
	void print(ostream &out)
	{
		out << "some text";
		out << "Some more text!" << number << std::endl;
	}
	{% endhighlight %}

  The second option looks a little cleaner in the function and requires less code/includes but I generally like the first because getting a string back in the client code give me more options I think. The Arduino libraries provides a nice "String" class with overloaded += operators that makes concatenating easy so I mindlessly went with my normal style, modified slightly:

	{% highlight C++ %}
	// Create JSON string
	String getJSON()
	{
		String output = "";
		output += "Some JSON Stuff";
		output += "More JSON Stuff";
		return output;
	}
	{% endhighlight %}

  Which worked great! At first... it seemed to only work intermittently, so I knew I was in for puzzle. I would get a few JSON outputs on my SD card, then... nothing. The built in LED blinks for each loop so I knew it was working, but even leaving it running for a long time I only got a few outputs, more than the first few but I had no idea why so few. I should have been writing a new line to the file every 15 seconds and I would have 3-5 after 5 minutes, and not always the same number! I created a quick `void blink(const int num);` function to blink my led for troubleshooting, and threw it in my code at key points with an incremental number of blinks. This was a lot easier than the serial port output since I was [using the sleep function which uses the DTR pin](learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/example-sketch-goodnight-thing-sleep-mode), preventing my from having my FTDI cable hooked up during tests. Using the LED also makes me feel *super hardcore* which is also important.

  Anyways I sat down and started counting..... and noticed that very rarely it would make it through the entire loop, but *usually* it would make it to my string returning print function and loop, without actually printing the string to the file. This made no sense, it was almost as if my main loop was cut in half. Even if the SD card failed to open, or the print function failed to get any info from the WiFi module, there was plenty of blinks and simple code *below* the print function that should be running! Long story short I moved around and added more `blink()` functions, even into the print function itself. I then saw that it does enter the print function but fails sometime during the for loop that fills the JSON with data from the wifi chip.....

  **Wait... AM I CRASHING / STACK OVERFLOW / SEG-FAULT??!?!?! :sweat: **

  So yea, rookie mistake but a good learning opportunity. Basically, even though the esp8266 has much more program memory than the average Arduino thanks to its use of an external flash chip it still only has **[96Kb](github.com/esp8266/esp8266-wiki/wiki)** of Data RAM to work with, and the WiFi stack takes some amount of that at any given time. Compared to an ATMEGA328 which has 2Kb, 96Kb is *huge* which is why I was able to get a few string out and not just crash every time. What threw me at first was the fact that an error like this would bring a desktop program to a screeching halt, but on a microcontroller it caused an instant reset, causing the appearance of a working, if incorrect, loop. While this is a total rookie mistake I learned a lot about the [memory structure](en.wikipedia.org/wiki/Harvard_architecture) of [microcontrollers](www.learningaboutelectronics.com/Articles/Microcontroller-memory-types.php) as well as troubleshooting and how to [better style my code](miscsolutions.wordpress.com/2011/10/16/five-things-i-never-use-in-arduino-projects/). 

  In the end I used the function below, and it worked great. This writes the JSON to the SD card as it is created, since the entire JSON array is usually too big to store in a string. The `File` object is from the SD library, and means the function will only work for this one thing. I may refactor later and investigate a way to make it more general. It would be nice to change between SD card, serial port, or something else by just changing the call in the client code. But for now... I went for a drive and came back with a *HUGE* JSON file I can figure out how to parse tomorrow :smile:

	{% highlight C++ %}
	// Create JSON string
	void getJSON(File &out)
	{
		out << "WORKS!!!"
	}
	{% endhighlight %}
