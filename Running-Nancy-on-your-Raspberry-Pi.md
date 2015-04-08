This page describes how to get up and running with Nancy using Mono and selfhosting on your Raspberry Pi.
The install/configuration described below was successfully performed on April 8th 2015.

# 1. Install OS on your Raspberry Pi

These instructions assume you are using the Raspbian, however they should work with Ubuntu as well, if you have a Raspberry Pi 2. You can check the [Raspberry Pi Downloads[(http://www.raspberrypi.org/downloads/ page for OS images.

# 2. Update OS to latest

Perform the following commands in a shell on your Raspberry Pi:

    sudo apt-get update
    sudo apt-get upgrade

After upgrade you should have something similar to the following:

    pi@raspberrypi ~ $ uname -a

    Linux raspberrypi 3.18.7+ #755 PREEMPT Thu Feb 12 17:14:31 GMT 2015 armv6l GNU/Linux

You might also want to update the firmware on your Pi to the latest version:

    sudo rpi-update

# 3. Install Mono

Mono is required for running Nancy, so go ahead and perform the following commands in a shell in order to get Mono installed:

    sudo apt-get update
    sudo apt-get install mono-complete

Note: Last step can take a long time.

After installation, type `mono -V`. You should see a result like 

    Mono JIT compiler version 3.2.8 (Debian 3.2.8+dfsg-4+rpi1)
    Copyright (C) 2002-2014 Novell, Inc, Xamarin Inc and Contributors. www.mono-project.com
            TLS:           __thread
            SIGSEGV:       normal
            Notifications: epoll
            Architecture:  armel,vfp+hard
            Disabled:      none
            Misc:          softdebug
            LLVM:          supported, not enabled.
            GC:            sgen
    

# 4. Create /var/www

Run the following command:

    sudo mkdir /var/www
    sudo chmod 755 /var/www

#5a Follow These Directions

You should now be able to [follow the directions here](https://github.com/NancyFx/Nancy/wiki/Hosting-Nancy-with-Nginx-on-Ubuntu#create-nancy-website), skipping the step in installing mono (you've already done that).

#5b Compile and Run Nancy Locally

The above instructions will walk you through creating the Nancy app on Windows and deploying to your Pi. However, if you'd rather do everything on the Pi start to finish, you can do the following instead:

This final section describes how to get up and running with Nancy on your Raspberry Pi.

## Compile Nancy:

First we need to install dependencies, retrieve Nancy source and finaly compile Nancy in order to get the dll's we are going to reference in our application later on. So in a shell perform the following commands:

    mkdir /home/pi/code (or any other directory you prefer)
    cd /home/pi/code
    sudo apt-get install git ruby rake
    sudo gem install albacore --version "1.0.0.rc2"
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

