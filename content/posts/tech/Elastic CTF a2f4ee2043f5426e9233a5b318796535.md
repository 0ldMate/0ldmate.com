---
title: "Elastic CTF"
date: "2020-07-06"
tags: ["elastic","blueteam"]
---

The Elastic CTF is a capture the flag competition that I built based on the Elastic Stack (formerly ELK Stack). I created it for the Sectalks Ninja Night as a way to give back something to the community that has given me so much. It was designed to give people a chance to play with a platform that is used quite often in security teams in many companies. This was my first time developing a CTF challenge and I hope I get the chance to do it again another time.

![/img/Elastic%20CTF%20a2f4ee2043f5426e9233a5b318796535/Untitled.png](/img/Elastic%20CTF%20a2f4ee2043f5426e9233a5b318796535/Untitled.png)

Spinning up the Elastic Stack and CTF platform using docker on digital ocean was a pretty simple task to get up and running. I used the base configs on both docker-elk and CTFd, however I did have to change the JVM heap of Elasticsearch to get it to boot properly. 

Creating this CTF wasn't a super difficult task if you generated it without any scenarios, you could just bring up the stack using docker then base questions off the sample data provided by Elastic. This would still allow users to manipulate and analyse data and learn the basics of Kibana, however, I found the scenario added a bit more fun to the challenge that allowed teams to use the knowledge they learnt from the basic challenges to solve a harder challenge. In the end, there was 36 challenges (21 beginner and 15 scenario).

![/img/Elastic%20CTF%20a2f4ee2043f5426e9233a5b318796535/Untitled%201.png](/img/Elastic%20CTF%20a2f4ee2043f5426e9233a5b318796535/Untitled%201.png)

Building the scenarios was the harder part, coming up with the idea and figuring out how to structure it was a challenge. This was my first time playing around with Docker and Digital Ocean, which learning that was difficult in itself. I had to learn about the way auditd and filebeat communicate to get the logs to come through how I wanted them.

I set up three additional systems separate to the elastic stack.

- a front facing machine that hosting ssh externally
- an internal machine (connected to the external box via VPC)
- an attacker box

Once all these servers were ready to go with the appropriate beats installed on them for logs there was just a few things I needed to set up before I was ready to simulate the attack. I installed an older version of the Elastic Stack (6.5.4) that was vulnerable to remote code execution ([https://github.com/LandGrey/CVE-2019-7609](https://github.com/LandGrey/CVE-2019-7609)) on the internal server and opened up SSH on the external facing server. I then simulated the attack from the attacker box making sure all logs are being sent to the Elastic Stack.

Scaling the stack was as simple as giving Elasticsearch more resources and increasing the JVM heap to allow more searches to be completed at a faster rate. In the end giving the Elasticsearch about 8gb of ram was enough to host all the users, with a max load of 85% of usage on Elasticsearch. If needed, you could create multiple nodes to create an Elasticsearch cluster for more users if you have a very large scale CTF.

Overall the CTF took me about 15-20 hours altogether to build it, a lot of that time was learning and refreshing my knowledge with Docker and Elastic Stack. I'm super happy with how it turned out and from the positive feedback, I think other players enjoyed it too. I think it worked pretty well as a way to introduce people to the UI and power of Kibana and would like to see other CTFs like it. Perhaps later down the line, I'll expand on it or do another similar ctf. I have made the CTF available to anyone on github if you'd like to spin it up yourself, it should be relatively simple.

## Running your own version

If you'd like to spin up the CTF it's quick and simple using Docker. It can be run locally or on a public VPS like Digital Ocean. To set it up, install Docker and Docker-compose and use the following Github links to set up the appropriate containers.

- [https://github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk) - easily spin up an Elastic stack quickly
- [https://github.com/CTFd/CTFd](https://github.com/CTFd/CTFd) - Used to host the CTF platform

Download and install [elasticdump](https://github.com/elasticsearch-dump/elasticsearch-dump) using either docker or npm (I used npm) which is used to import  logs. Clone [my ElasticCTF repo](https://github.com/0ldMate/ElasticCTF) that has all the elasticsearch logs. There are three different indicies that are included in the repo that need to be imported to complete all the challenges. Note the auditbeat data was too large to store in one compressed file so I split it into multiple verisons.

Once you have the stack set up, you can import the data with a few Elasticdump commands that look like the following.

```jsx
elasticdump --input=file.json --output=http://username:password@localhost:9200/index-name
```

Input the index data from the json files to the Elasticsearch server so it can be analysed and viewed in Kibana. You need to import the mapping files first before importing the data files. If you hadn't changed any of the config files when starting up docker-elk, you should be able to run the following commands to import the kibana sample mapping and data.

```jsx
elasticdump --input=kibana-sample-mapping.json --output=http://elastic:changeme@localhost:9200/kibana-sample
elasticdump --input=kibana-sample-data.json --output=http://elastic:changeme@localhost:9200/kibana-sample
```

Then go through and import the other two indexes, to import the auditbeat index you need to import the data according to the numbers of the files, from 1-13. After that the stack should have all the information required you just need to import the ctfd data in the repo using the admin panel.

Now you're all ready to go and play the CTF.
