Now that we’ve performed manual OS Command injection attacks, let’s take a look at using an automated tool called Commix. The reason that we started with manual attacks is twofold:
1. To get a better grasp of how injections work beyond just the concepts
2. While automated tools are great, sometimes they’re not available or they don’t behave the way that we need them to
But, automated tools can drastically speed up tests, find vulnerabilities that we may not find manually, and in general, help find the low-hanging fruit so that humans can focus in other areas.

## Getting started with Commix
That’s why, in this lesson, we’re going to look at a tool called [Commix](https://commixproject.com/). Commix is short for [comm]and [i]njection e[x]ploiter that can be used by web developers, penetration testers or even security researchers in order to test web-based applications with the purpose of finding bugs, errors or vulnerabilities related to command injection attacks.

It’s an easy tool to use, it’s compatible with other popular tools & frameworks like SQLMap, Metasploit, and others; it’s modular meaning that you can develop your own modules for it, it’s cross-platform and you really only need Python to run it, and it’s open source! It’s actually even featured in major pentesting distributions like Kali, BlackBox, Parrot, and others.

So if we go over to our Kali installation and search for it, we’ll see that it’s already included. At the time of this recording, though, I had recently gone through and performed quite a few updates on my system and also updated to Kali 2020.3 since it just came out. I’m not sure if that’s what caused issues or not, but when I tried to use the natively installed Commix again, I kept getting a Critical 13 error. 

A fix I found is to download the source code and run it using python. So I downloaded v3.2-dev Commix instead of v3.1-stable. This is not the default that comes with Kali, and it is a dev version, so keep that in mind. If you’d rather use the stable version, you could try downloading that instead (under Releases in [GitHub](https://github.com/commixproject/commix)).

Keep in mind that there may be different versions by the time you go through this lesson anyway, so your numbers might be different.

In any case, to start, download it from GitHub:

```wget https://github.com/commixproject/commix/archive/master.zip```

Next:

```unzip master.zip```

Let’s remove that zip file:

```rm master.zip```

Go into our new directory

```cd commix-master```

From there, we can run commix directly with

```python commix.py```

By the way, as an alternative, I tried installing this new version as the main version, but for some reason that still caused the Critical 13 error. So, to make sure that you have a smooth experience in this course, let’s stick to using it this way.

## Pulling up our Commix Testbed environment
Alright now let’s pull up the commix-testbed app instead of the DVWA. If you don’t already have it running, type this command:

```docker run --rm -d -p 3000:80 cybrcom/commix-testbed```

Then navigate to ```http://localhost:3000/commix-testbed```

From there, you will see a number of different scenarios and categories for us to test. Starting from the left, we have our classic and simple examples, but each with its own restrictions which serves to mimic examples we might find in the real world when checking our own applications.

The reason that they are broken out like this is because it outlines different potential entry points for our OS Command injection attacks.

With GET requests, we’ll try to inject URLs. With POST requests, we’ll try to inject inputs via the POST method, and then with HTTP headers, we’ll try to exploit cookies, referrer HTTP headers, and user-agent HTTP headers.

The regex filters section shows us different security controls that try to implement regular expression filtering to prevent malicious input, with varying success. This is helpful to see realistic code examples of attempts to defend against OS Command injections that are still vulnerable.

## Our first attack with Commix
Let’s start with the first one to make sure everything works as expected, and then we’ll move on to some other scenarios.

We’ll click on the GET version. Your URL should now look like this: ```http://localhost:3000/commix-testbed/scenarios/regular/GET/classic.php```

From there, we see an input box further down on the page which lets us ping an ip address. But, instead of pinging an address, let’s test with a very basic command injection:

```; pwd```

OK great, that worked. We now know the path structure of this application, and more specifically, this endpoint:

```/var/www/example.com/public_html/commix-testbed/scenarios/regular/GET```

We also know that this input is vulnerable to OS Command injections, so let’s bring out the big guns.

## Commix command and options
In order to understand how to use Commix, we have a few options. The first and quickest is to type -help:

```python commix.py -help```

This will bring up the list of commands and options we can use with Commix, starting with:

- General options
- Target options, to define a target URL
- Request options, to define how to connect to the target URL
- Enumeration options, which is used to find possible entry points and vulnerable information in the target system
- File access options, which can be used to access, write, or upload files — an important option when trying to create backdoors on the target system
- Modules options, which can be used to increase the detection and/or injection capabilities
- Injection Options, which can be used to customize injection payloads and specify which parameters to inject
- Detection options, which can be used to customize the detection phase of injection vulnerabilities
-  and then Miscellaneous options

Let’s put these options to work with our first attack against this input.

## Structuring a Commix attack
Going back to the testbed app, we can look at the URL and since it was a GET request, we see how the URL is structured which will help us build our attack and identify where to inject payloads.

Now we can type this:

```python commix.py --url="http://127.0.0.1:3000/commix-testbed/scenarios/regular/GET/classic.php?addr=INJECT_HERE"```

The ```INJECT_HERE``` placeholder tells Commix where to inject its payloads.

Let’s submit this. We will see:

```[info] The GET parameter 'addr' seems injectable via (results-based) classic command injection technique.```

## Pseudo-Terminal shell
It asks us if we want a Pseudo-Terminal shell, which will give us access to perform commands on our target host. Type ```Y``` for yes.

If we input a question mark, we’ll get a list of available options, and we can see things like ```reverse_tcp``` and ```bind_tcp``` which would set up a reverse TCP connection or set a bind TCP connection.

The Bind TCP option opens up a port on the target host, while Reverse TCP tries to connect to you from the target machine back to your machine.

We briefly saw the concepts of reverse shells in a prior lesson, and this is very similar, although Bind shells and Reverse shells do have differences, and typically reverse shells work best. I won’t go into the differences here, but feel free to look it up or ask us in our Discord server or Forums if you’re curious to learn more.

In any case, let’s see what kind of information we can get form this pseudo-terminal.

Just like we’ve seen in a prior lesson, we can type commands like:

```whoami```

To gather information about what privileges we may have with our current user. In this case, we are the user ```www-data```.

Although we already know the directory for this endpoint, we can confirm here:

```pwd```

Alright, let’s exit out of here for now with ```Ctrl + C```.

## Automatically gathering system information with Commix
Now, let’s collect the same kind of data that we collected through the pseudo-terminal, but instead we’ll let commix do that for us by passing in enumeration options:

```python commix.py --url="http://127.0.0.1:3000/commix-testbed/scenarios/regular/GET/classic.php?addr=INJECT_HERE" --current-user --hostname --is-root --sys-info```

It now returns information about the current user running this application, the hostname, whether the user has root privileges, and system information like the operating system and hardware platform.

And now let’s take a look at a User-Agent HTTP Header injection vulnerability.

## User-Agent HTTP Header injections

We’ll go to the Classic user-agent-based example.

At this endpoint, the testbed application prints out our User-Agent information from the HTTP headers, and if we cheat and [look at the source code](https://github.com/commixproject/commix-testbed/blob/master/scenarios/user-agent/ua(classic).php): we will see that the code grabs the ```HTTP_USER_AGENT``` from the ```$_SERVER``` variable and then runs the exec command to echo out that information:

```
<?php  
$user_agent = $_SERVER['HTTP_USER_AGENT'];  echo exec("echo '".$user_agent."'");
?>
```

So at first glance, this definitely looks injectable because I am seeing no security mechanism, and we can modify the ```HTTP_USER_AGENT``` to be whatever we want it to be!

Let’s set up our attack with this command, we can use this command:

```python commix.py --url="http://127.0.0.1:3000/commix-testbed/scenarios/user-agent/ua(classic).php" --level=3 --technique="c"```

- --level=3 – option 3 tests User-Agent and Referer values, whereas –level=2 tests Cookie values
- --technique="c" – uses the classic, results-based, command injection technique

This one can take quite a while, so we’ll fast-forward.

After a little while, I think maybe 5-10 minutes, we find a successful payload:

```echo JFGKXK$((43+25))$(echo JFGKXK)JFGKXK```

It again asks us if we want a Pseudo-Terminal shell, but I’ll go ahead and say no this time. I also won’t continue the attack.

Conclusion
At this point, we’ve looked at a few different examples, but there are many more in the commix application, including cookie-based, referer-based, and more, so feel free to play around with those! There are also some that may not be exploitable, so you can try to figure out which one those are and how they’re defending against Commix’s automated attacks!

Otherwise, let’s go ahead and complete this lesson, and I’ll see you in the next!
