In this lesson, we walk through setting up our environment in order to follow along with the hands-on demonstrations throughout the course. This is an important lesson to complete if you want to apply what you’re learning hands-on, so if you get stuck at any point in time, please reach out and we’ll help you resolve the issue so that you can move on.

The first thing we need to configure is Kali Linux, which is a free Linux distribution that’s often used for digital forensics and penetration testing. The reason we want to use Kali is because it comes pre-installed with many of the tools we’ll be using throughout the course, which will help us get going and avoid issues that can come from running different operating systems.

## Creating a Kali VM with VirtualBox
Don’t worry, this step is not difficult and it doesn’t take too much time. And again, this is all free. If you don’t already have VirtualBox or VMWare, go ahead and download whichever one you prefer, but I’ll be using VirtualBox. All you have to do is go to virtualbox.org and download the latest version for your current operating system. I’m on a mac, so I’ll download the OS X version, but if you’re on Windows you would download that version.

Then, follow the steps to install VirtualBox. At this point, if you have any issues during the installation and you can’t figure out a solution, please reach out in our [forums](https://cybr.com/forums/) or on [Discord](https://cybr.com/discord) and we’ll be glad to help.

Once you have VirtualBox installed and running, it’s time to set up Kali Linux. There’s a great tutorial for using Kali ISOs [located at this URL with instructions](https://phoenixnap.com/kb/how-to-install-kali-linux-on-virtualbox), so I won’t go into too much depth if you want to install Kali using an ISO which provides a bit more customizability but takes longer and requires more configuration:

Instead, I’ll use an OVA version. The main difference between OVA and ISO is that OVA will import Kali into VirtualBox instead of installing it as if we were inserting a CD version of Kali. This is a very simple way of getting Kali up and running without having to configure a lot of settings, and it will work just fine for this course.

First, we’ll want to download Kali at this URL: https://www.kali.org/downloads/

If you want an ISO image of Kali to be able to boot from it, then you can download from that page, but since we’re using the OVA version and VirtualBox, we’ll need to click on this link: https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/

And we’ll download the 64-Bit version. This can take a few minutes depending on your internet connection. Once you’ve downloaded the OVA, go to VirtualBox and Import the Appliance (File -> Import Appliance), or double-click the OVA file.

Before importing, you’ll want to double-check settings to make any modifications necessary. Then, start the import process. This can take a few minutes. After importing the appliance, we can check Settings again to make further modifications. Some of the settings to take a look at include CPU and Memory allocation, and this will depend on your system and how much you are willing to allocate to this virtual machine, so I’ll leave that up to you.

One setting I ran into issues with, however, is the USB 2.0 settings. In our case, we will need to disable USB 2.0, or install a package that enables this functionality. Since we won’t be needing USB 2.0, I chose to disable it. Select the virtual machine in VirtualBox and then click on Settings. Go to “USB” (if doing this on Windows) or “Ports” (if doing this on Mac). You can then uncheck the box “Enable USB controller” (you will need to click on the “USB” tab on Mac after clicking on Ports) and save settings.

We’re now ready to start the machine.

Log in using ```kali/kali``` as username/password (we will change this in a moment).

## Changing the default password
```passwd```

Make sure you read the instructions because people oftentimes blow through those steps and wonder why it doesn’t work :-). The system will ask you to put in your current password first, then your new password twice.

## Installing Docker in Kali
We’re now ready to install software that we will use throughout the course. Let’s start by installing Docker. 

### Step 1: Add a Docker PGP key
```curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -```

We do this for privacy and also for file integrity to help make sure no one is tampering with our download.

### Step 2: Add and configure the Docker APT repository
```echo 'deb [arch=amd64] https://download.docker.com/linux/debian buster stable' | sudo tee /etc/apt/sources.list.d/docker.list```

Now we can update our package manager:

```sudo apt-get update```

### Step 3: It’s time to install Docker
```sudo apt-get install docker-ce```

We can test our install with:

```sudo docker run hello-world```

At this point, docker service is started but not enabled. Run:

```sudo systemctl start docker```

If you want to enable docker to start automatically after a reboot, which won’t be the case by default, you can type:

```sudo systemctl enable docker```

I prefer not to do that, since I don’t always use Docker when launching this Kali virtual machine.The last step is to add our non-root user to the docker group so that we can use Docker:

```sudo groupadd docker```

```sudo usermod -aG docker $USER```

We now need to reload settings so that this permissions change applies.

```newgrp docker```

The best way to reload permissions, though, is to log out and back in. If that doesn’t work, try to reboot the system. Otherwise, you may found that other terminal windows haven’t reloaded settings and you may get “permission denied” errors. But, if you’d rather not log out or reboot at this time, you can use the above command.

## Running our target environments with Docker
With docker installed, we can now pull in different environments as we need them, without having to install any other software for those environments.

### The Damn Vulnerable Web Application (DVWA)
For example, if we want to run the Damn Vulnerable Web Application, we can do that with this simple command:

```docker run --rm -it -p 80:80 vulnerables/web-dvwa```

If that doesn’t work, try running this command first:

```docker pull vulnerables/web-dvwa```

and then re-run the docker run command above. You’ll have to wait until it downloads the needed images and starts the container. After that, it will show you the apache access logs so you can see requests going through the webserver. 

You can navigate to 127.0.0.1 in your browser in order to access the web application.

It will ask you to login, and you can use the username admin and password password. Initially, you will be redirected to localhost/setup.php where you can check configurations and then create the database.

### The Commix Testbed
Another environment we’re going to use in the course is a commix-testbed which was created specifically to test a tool called commix which we will explore later on. While there are instructions on the main GitHub for how to install this application, I much prefer using something like Docker to spin up and spin down environments, especially if I’m only temporarily using those environments. 

Since I couldn’t find a dockerized version of this app, I created one. Using it is very simple, just type:

```docker run --rm -d -p 3000:80 cybrcom/commix-testbed```

And since we already have the DVWA app running on port 80, we’re running the testbed app on port 3000. This time, we passed in the -d symbol in order to run this container in detached mode to free up our terminal window for other things.

It can be helpful to remain attached to the DVWA since it prints out apache logs that we can look at, but here there won’t be anything to see.

Open up your favorite browser and go to ```localhost:3000``` where you will see the commix-testbed directory. Click on that, and you will be in the testbed app. From there, we have multiple scenarios to walk through, but we’ll keep that for another lesson.

## Shutting down Docker containers
Once you’re done with your container environments, you’ll probably want to shut them down. To do this, type:

```docker ps```

Grab the ID(s), and use them in this command:

```docker kill <id>```

This will shut down, or kill, the container(s). You can verify that by doing another docker ps which should show no more running containers.

## Conclusion
Feel free to explore these applications if you’d like, and then go ahead and move on to the next, where we will start to review important concepts to understand before we can perform command injections.
