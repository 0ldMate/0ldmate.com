---
title: "Elastic Stack CTF Scenario Walkthrough"
date: "2020-06-28"
tags: ["elastic","blueteam"]
---

This is a walkthrough for the Elastic Stack CTF scenario that was run for the [Sectalks Ninja Night 0x08 (9th)](https://www.meetup.com/en-AU/SecTalks/events/271153250/).

The CTF is still live and is available [HERE](http://elasticctf.0ldmate.com:8000/) for the next few days. Feel free to hop in and give it a go.

Scenario:

> Overnight we've had an attack on our network, we have two devices in the cloud and it appears both have been compromised.
> The attack appears to have taken place on the 25th of May between 9pm and 11:30pm.
> Our network is composed of one box that is front facing with an SSH port open to the web and a second server behind it running an old Elastic Stack.
> We believe the ssh password was bruteforced and then exploited the vulnerable elastic stack version.
> Both boxes have logs sent to our new Elastic Stack.
> Please recover the information requested in these challenges so we can piece together what happened.


1. What IP did the bruteforce attack come from?

Select the SIEM Menu > Hosts > Authentications and set the time frame given in the scenario, the first few challenges can be solved within this menu.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled.png)

This shows us the IP address of the "Last failed source" 134.122.125.130 is the flag.

2. What username did they crack?

Staying in the same menu, we can see the username that was attacked

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%201.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%201.png)

The username "johnny" was the target of the bruteforce. 

3. What host was attacked?

Staying in the same menu, we can see the name of the host that was attacked

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%202.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%202.png)

The host "sshbox" was the host that was attacked.

4. How many failed attempts were made on the machine?

Staying on the same menu, we can see the amount of failed password attempts on the machine

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%203.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%203.png)

There were "611" failed attempts to authenticate to the server.

5. What time was the last failed attempted login?

Although the answer is on this page, I swapped over to a dashboard to get further information on the login. Go to Dashboard > Login Dashboard ECS.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%204.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%204.png)

We can see the last failed log with the time of "21:39:31".

6. What time did the attacker successfully login?

Using this Dashboard we can also view the successful login time.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%205.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%205.png)

We can see the successful login time was "21:50:13".

7. What is the first command the attacker ran on the box?

Now we will have to look deeper into the logs to view what commands were run, a good point to note is to alter the log filter start time from the original scenario time to the time of successful login (as they couldn't run commands without being logged into the device). Go to Discover and change the Index Pattern to "filebeat-*"

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%206.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%206.png)

Now we need to start filtering out logs that aren't useful. You can filter by using the search bar and/or using the selected fields.

Start off by filtering by "agent.hostname" to the box that the attacker bruteforced - immediately we drop the logs drastically from over 457,000 to 434 logs

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%207.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%207.png)

Comparing logs or by looking at the hints you can then filter the logs for the "event.action" field so it only displays docs that contain "execve" - the original logs were made using auditd which tags commands executed on the command line as "execve"

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%208.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%208.png)

Now, we could manually go through these logs, but there's a much easier way to filter them so they're easier on the eyes. After viewing a few of these logs, you should notice that the "auditd.log.a0" field is the command that's run and every consecutive number after 0 is arguments run along with it. If we select add on a few of those fields we should be able to piece together commands that were run by the attacker.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%209.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%209.png)

Keeping in mind, these aren't clean logs as there are some system processes that are still appearing in this list so we can filter further by selecting "auditd.log.a0" and exclude anything matching "/bin/sh", "/usr/bin/dpkg" and "apt-config" as these are most likely system noise.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2010.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2010.png)

Now we've got a reasonable amount of logs to look through that is easy on the eyes. Assuming the attacker didn't type commands within milliseconds of authenticating to the machine we can see the first command was "whoami"

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2011.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2011.png)

This gives us the flag.

8. What tool did the attacker use to get the exploit onto the machine?

Now we have all the work done in challenge 7, the next few should be easy to spot. Viewing the commands run on the box, we can see there was some kind of connection made to github - [https://github.com/LandGrey/CVE-2019-7609.git](https://github.com/LandGrey/CVE-2019-7609.git) 

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2012.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2012.png)

The CVE was cloned from github using "git" which was the flag.

9. Shortly after getting the exploit on the machine, the attacker used vim to create a file, what is the name of that file?

Staying on the same page, we can see vim was used

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2013.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2013.png)

The file name was "ElasticCTFisFun!" which was the flag

10. What is the filename of the exploit that was run?

Staying on the same page, we can see the exploit was run using python

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2014.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2014.png)

The file run was "CVE-2019-7609-kibana-rce.py" which was the flag

11. What is the ID of the log that shows the exploit being run?

If we expand this log we can find the ID

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2015.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2015.png)

The ID was "tSHdS3IBCEolQs9lI0Hb" which was the flag

12. What parameter turned the script from testing to exploiting?

Now we need to take a look at what the exploit did, looking at the github page, we can view what parameter triggers the shell.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2016.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2016.png)

The parameter "—shell" causes the exploit to create a reverse shell which was the flag

13. What IP was the shell sent to?

Looking again at the exploit on GitHub, we can see what parameter identifies the reverse shell IP

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2017.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2017.png)

We can see what follows the -host parameter is the IP the shell is sent to which is "10.116.0.2"

14. After running the exploit, they accessed the /etc/passwd file, what is the ID of the doc that shows this?

Now we need to change up the filters as we are now looking for the commands run on a different machine. Altering the "agent.hostname" to "elkstack" allows us to see the logs of the exploited computer. 

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2018.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2018.png)

There are a lot of logs so we need to filter further. We can exclude out some of the "auditd.log.a0" fields that aren't going to help us like "/usr/share/kibana/node/bin/node" which removes most of the logs.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2019.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2019.png)

Now we can view and expand the log that mentions "/etc/passwd"

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2020.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2020.png)

This gives us the ID "mSPtS3IBCEolQs9l-ekW" which was the flag

15. We think they created a new user, what was the name of that user?

Finally, using the same filters we can see the log that has created a user.

![/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2021.png](/img/Walkthrough%2043f8146f64ee495ea39e69e02d73ff58/Untitled%2021.png)

The username of the account created by the attacker was "Thanks4Playing" which was the flag.

I hope you enjoyed the Elastic CTF and I hope it gave you a chance to play around and learn the Kibana GUI.
