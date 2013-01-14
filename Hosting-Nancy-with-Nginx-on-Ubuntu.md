This tutorial describes how to install and run a NancyFx powered website on [Ubuntu 12.04][ubuntu]

# Install mono on your ubunu machine
Go to the [mono download page][getmono] to retrieve the the latest mono version. In our case, we build the latest source from github:

    $ sudo apt-get install git autoconf automake libtool g++ gettext
    $ mkdir ~/src
    $ cd ~/src
    $ git clone git://github.com/mono/mono.git
    $ cd mono
    $ ./autogen.sh --prefix=/usr/local
    $ make get-monolite-latest
    $ make
    $ sudo make install

Now we are ready to run .NET applications under linux.

# Install mono on your development machine
Go to [mono][getmono] to retrieve the the latest mono version for windows. In our case 3.0.2.

# Create nancy website
Open VS2012 and start a new console app NancyDemo.sln
![nancy nuget](https://dl.dropbox.com/u/19401194/NancyNuget.png "nancy nuget")

Edit the program.cs file and add the following code:

```c-sharp 
namespace NancyDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            var uri = "http://localhost:8888";
            Console.WriteLine(uri);
            // initialize an instance of NancyHost (found in the Nancy.Hosting.Self package)
            var host = new NancyHost(new Uri(uri));
            host.Start();  // start hosting

            //Under mono if you deamonize a process a Console.ReadLine with cause an EOF 
            //so we need to block another way
            if (args.Any(s => s.Equals("-d", StringComparison.CurrentCultureIgnoreCase)))
            {
                while (true) Thread.Sleep(10000000);
            }
            else
            {
                Console.ReadKey();
            }

            host.Stop();  // stop hosting
        }
    }
}

```

Create a new folder Modules and add the class HelloModule.cs

```c-sharp
namespace NancyDemo
{
    public class HelloModule : NancyModule
    {
        public HelloModule()
        {
            Get["/"] = parameters => "Hello World";
        }
    }
}
```

To make sure all code at leasts builds under mono, we build the sln with xbuild.
    
    D:\Development\Mono\NancyDemo>"C:\Program Files (x86)\Mono-3.0.2\bin\xbuild.bat" NancyDemo.sln 

Once the build succeeds copy NancyDemo.exe, Nancy.dll and Nancy.Hosting.Self.dll to the linux machine. We can test if it works by running the console app.

    $ mono NancyDemo.exe

Now open a browser and go to http://localhost:8888. We should see "Hello World".

# Install nginx
[nginx][nginx] is the webserver we're using. We configure it to forward all requests to the nancy self hosted application. The content folder with static files will be handled by nginx.

    $ sudo apt-get install nginx

Create the website configuration file in /etc/nginx/sites-available/nancydemo with the following content. The server_name is the domain on which the request will be handled. Change this to your own value.

    server {
        listen       80;
        server_name  yourdomainname.com;
        root /var/www/nancydemo;

        location /Content/ {
            alias /var/www/nancydemo/Content/;
            location ~*  \.(jpg|jpeg|png|gif|ico|css|js|ttf)$ {
                expires 365d;
            }
        }

        location / {
                proxy_pass http://127.0.0.1:8888;
        }
    }

To enable the website, create a symbolic link from the sites-available to the sites-enabled folder. This will make it easy to temporary disable sites in the future.

    $ sudo ln -s /etc/nginx/sites-available/nancydemo /etc/nginx/sites-enabled/nancydemo

The configuration is completed, reload Nginx to apply.

    $ sudo /etc/init.d/nginx reload

# Install supervisor
To make sure our nancy self hosted website never stops, we use [supervisor][supervisor]. This program makes sure that NancyDemo keeps running.

    $ apt-get install supervisor

Configure supervisor by creating a new file /etc/supervisor/conf.d/nancydemo.conf

    [program:nancydemo]
    command=/usr/local/bin/mono NancyDemo.exe -d
    user=www-data
    stderr_logfile = /var/log/supervisor/nancydemo-err.log
    stdout_logfile = /var/log/supervisor/nancydemo-stdout.log
    directory=/var/www/nancydemo/

And start the control manager of supervisor

    $ sudo supervisorctl

And update the configuration. You should see that there is a new process added. Now start nancydemo.

    $ supervisor>update
    $ supervisor>start nancydemo

# Put your hands in the air
Go to http://yourdomain.com and see our baby saying "Hello World".

# Notes
Linux is very critical about uppercase and lowercase letters, so if a view can't be found, make sure you've used the exact name with the exact uppercase and lowercase letters.

[getmono]: http://mono-project.com/Obtaining_Mono
[ubuntu]: http://www.ubuntu.com/download/desktop
[supervisor]: http://supervisord.org/
[nginx]: http://nginx.org/
[nancy-nuget]: https://www.dropbox.com/s/8fyp5725ghqiwte/NancyNuget.png "nancy nuget"