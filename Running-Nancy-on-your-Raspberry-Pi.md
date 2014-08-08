This page describes how to get up and running with Nancy using Mono and selfhosting on your Raspberry Pi.
The install/configuration described below was successfully performed on 12th-14th of April 2013.

**Update Aug 8th 2014:**
Hard float is now supported - which should have a major impact on the performance of running mono apps on the Raspberry Pi. [Click here for details](http://c-mobberley.com/wordpress/2013/12/27/raspberry-pi-mono-3-2-installation-hard-float-compatible/)

# 1. Install OS on your Raspberry Pi

Due to the need for also running Mono, you currently need to install the soft-float Debian “wheezy” (2012-08-08-wheezy-armel.zip) on your Raspberry Pi. This is due to the fact that Mono currently does not work correctly with the hard-float version of the Raspbian OS (which is the default and latest one). Read more on this issue [here](https://bugzilla.xamarin.com/show_bug.cgi?id=7938). The soft-float OS can be downloaded from [here](http://www.raspberrypi.org/downloads).

You might also get it up and running with hard-float Raspbian using [these instructions](http://www.raspberrypi.org/phpBB3/viewtopic.php?f=34&t=37174), so if you feel like experimenting, do go ahead and try it out :)

To install the soft-float OS to your SD card, use the instructions on:

http://elinux.org/RPi_Easy_SD_Card_Setup

The "Linux + ImageWriter" way of writing the image to SD card has worked excellent, but any method should work just fine.

# 2. Update OS to latest

Perform the following commands in a shell on your Raspberry Pi:

    sudo apt-get update
    sudo apt-get upgrade

After upgrade you should have something similar to the following:

    pi@raspberrypi ~ $ uname -a

    Linux raspberrypi 3.2.27+ #250 PREEMPT Thu Oct 18 19:03:02 BST 2012 armv6l GNU/Linux

You might also want to update the firmware on your Pi to the latest version:

    sudo rpi-update

# 3. Configure OS to optimize performance

In a shell run the command: 

    sudo /usr/bin/raspi-config

Set the following settings:

* Memory-split: 16MB

* Overclock: Modest 800 MHz (or just leave it at 700 MHz if you're afraid of burning off your Pi)

* Boot to GUI: No (uses extra resources, but might still work if booting to GUI)

* Upgrade raspi-config

Then perform a reboot from within raspi-config after setting the above settings.

After reboot verify you have the full 512 MB of RAM available by running the command:

    top

and verify you have almost 500 MB of RAM (on the top left part of page).


# 4. Install Mono

Mono is required for running Nancy, so go ahead and perform the following commands in a shell in order to get Mono installed:

    sudo apt-get update
    sudo apt-get install mono-complete

Note: Last step takes a long time, 25+ min.

# 5. Compile Nancy, Create & Run Nancy apps

This final section describes how to get up and running with Nancy on your Raspberry Pi.

## Compile Nancy:

First we need to install dependencies, restrieve Nancy source and finnaly compile Nancy in order to get the dll's we are going to reference in our application later on. So in a shell perform the following commands:

    mkdir /home/pi/code (or any other directory you prefer)
    cd /home/pi/code
    sudo apt-get install git ruby rake
    sudo gem install albacore
    git clone https://github.com/NancyFx/Nancy.git
    cd Nancy
    rake mono

Note: final step takes a long time. (optionally you might be able to only run 'rake compilemono')

Note: The gem `albacore` version 0.3.5 has a depency bug with `rubyzip` ([Issue #70](https://github.com/Albacore/albacore/issues/70)). Creating the `albacore` gem from the current dev branch (0.4.0) works (takes about 45min on the RPi...):

Alternatively you can install rubyzip 0.9.9 which saves you that time.

``` sh
sudo aptitude install ruby1.9.1-dev
cd ~/code/
git clone https://github.com/Albacore/albacore.git
cd albacore
gem build albacore.gemspec
sudo gem install albacore-0.4.0.gem
```

## Create Nancy app file using Nancy Selfhosting

Now we are ready to create our first self-hosted Nancy app, so go ahead and save the following code:

    using Nancy;
    using System;
    
    namespace Example
    {
            public class Program:Nancy.NancyModule
            {
                    static void Main(string[] args) {
                            Console.Write("Starting server...");
                            var server = new Nancy.Hosting.Self.NancyHost(new Uri("http://localhost:8282"));
                            server.Start();
                            Console.WriteLine("started!");
                            Console.WriteLine("press any key to exit");
                            Console.Read();
                    }
    
                    public Program()
                    {
                            Get["/"] = _ => { return "Nancy says hello!"; };
                    }
            }
    }

in a file mains.cs:

1. Write the Nancy app file using

    nano main.cs

2. And then paste (right-click) + exit (CTRL-X) + Save

## Compile and run Nancy app

Finally we can compile our Nancy app using the following command:

    dmcs -r:/home/pi/code/Nancy/src/Nancy/bin/MonoRelease/Nancy.dll -r:/home/pi/code/Nancy/src/Nancy.Hosting.Self/bin/MonoRelease/Nancy.Hosting.Self.dll main.cs

In order to run the app we must either copy the two Nancy dll's (Nancy.dll and Nancy.Hosting.Self.dll) into the directory of the executable so that they are placed beside the application or we must make sure they are all added to the path searched when resolving assemblies. As a start just place the Nancy dll's beside the executable file (the location of the Nancy dll's can be found in the previous command executed above). 

Now run the Nancy application by issuing the following:

    mono main.exe

And finally your should be able to invoke the service by pointing your favorite browser to the address http://localhost:8282 and you should be greeted by a "Nancy says hello!".

# Useful resources

https://docs.google.com/document/d/1bzbRZFmrYOLh-ldUAcd3_3-o5ISAN7GtrLfOhJoj0qI/edit?usp=sharing

http://hubcitylabs.org/unlocking-your-new-raspberry-pis-512mb-of-memory/

http://mongopi.wordpress.com/2012/11/25/installation/

http://andyfelong.com/2013/02/raspberry-pi-meets-mongodb/

http://stackoverflow.com/questions/7948789/mongodb-mongod-complains-that-there-is-no-data-db-folder

http://sourcecodebean.com/archives/mongodb-c-and-mono/1408



