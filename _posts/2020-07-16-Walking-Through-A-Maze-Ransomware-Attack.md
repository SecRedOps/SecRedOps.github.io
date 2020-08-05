---
title: "Blog: Walking Through a Maze(Ransomware Attack)"
excerpt: "Break down of the Maze gang(TA2101) TTPs."
header:
  teaser: "/assets/images/Maze.png"
tags: 
  - Malware
  - Ransomware
  - Maze
---

*Coming Soon my write up on Maze TTPs*

### Walk-through of a recent breach by the Maze gang(TA2101)


<figure>
	<a href="/assets/images/Maze.png"><img src="/assets/images/Maze.png"></a>
</figure>

The sudden realisation of being a victim of a ransomware attack is a feeling not many hopefully will have to experience, however, for the few that do, I can only imagine the heart sinking feeling of turning a screen on and seeing the following message:

> Attention!
>
> What happened?
>
> We hacked your network and now all your files, documents, photos, databases, and other important data are safely encrypted with reliable algorithms.
> You cannot access the files right now. But do not worry. You can get it back! It is easy to recover in a few steps.
>
> We have also downloaded a lot of private data from your network, so in case of not contacting us as soon as possible this data will be released.
> If you do not contact us in a 3 days we will post information about your breach on our public news website and after 7 days the whole downloaded info.
>
> To see what happens to those who don't contact us, google:
> * Southwire Maze Ransomware
> * MDLab Maze Ransomware
> * City of Pensacola Maze Ransomware
>
> After the payment the data will be removed from our disks and decryptor will be given to you, so you can restore all your files.

The text above is saved as a desktop background for any PC that has been encrypted by the Maze Ransomware attackers. The text continues and mentions about installing TOR to access a URL that will allow you to speak to the ‘support staff’ at Maze. It also contains a public key that is relevant to the specific attack by Maze and how you will need to upload the file to the Maze site in order to identify yourself and discuss paying the ransom. 
The current tactics of Maze are to not only encrypt your data, but ex-filtrate it to their servers where they will hold it until you make the requested ransom payment. 

Whilst not strictly sticking to the specific time frames, Maze are true to their word and regularly update their site of companies who have failed to make contact or payments.

So, how would a company arrive to be in this situation? Well, I was recently tasked with answering that exact question, and through forensic acquisition and analysis I can provide a walkthrough of a Maze attack, and importantly what to do to avoid it happening to your and your organisation. Or if you have already fallen victim to Maze, what to do after the incident and how to respond accordingly. 

So to paint the scene, we have arrived on site, secured the systems and network, taken forensic images of all the affected devices and where required ensured that there is no possibility of persistence by resetting all passwords of any corporate information systems, such as emails, desktops and so on. 
Now that we have recovered the infected images to our labs, we can begin the investigation. 

### The Foothold 

Every attack that involves some form of remote code execution has to start somewhere, and that is what is known as ‘The Foothold’ - hackers have to gain this crucial first step in order to move laterally within a network and perform any reconnaissance of further exploit avenues. 

Maze tactics are to aim for low hanging fruit, and most often this is an open Remote Desktop Protocol that is accessible from anywhere on the Internet. 
Looking through the logs of one device that was exposed to the Internet with the RDP port open, we can instantly see there has been a sustained brute-force attack against the server for several months. Below you can see the number of failed logins for only one month of logs. 

<figure>
	<a href="/assets/images/RDP-BF.png"><img src="/assets/images/RDP-BF.png"></a>
</figure>

Then after several months of trying the attackers are successful at logging into the server. The logs show our first entry point, and no prizes for guessing where the attacker is from:

<figure>
	<a href="/assets/images/RU-IP.png"><img src="/assets/images/RU-IP.png"></a>
</figure>

An interesting point to highlight is that the attackers are not wasting time with low privilege accounts are only attempting to log in with Administrator style user names. As you can see from the failed logins, they are only using several variations. 

So, we are now at the point where the attackers have access to a server on an internal network. Their next task is to carry out reconnaissance of the victim server and other servers they might want to access. 

### Lateral Movement 

Now the attackers have access to the internal network, with a Domain Administrator account no less, they will attempt to enumerate other hosts and services running on the network. To do this the attackers download a network scanning tool to the server on the Administrator desktop, this also includes their licence and configuration for the tool of what they want to check across the network.

The attackers are also clearly automating all of this, as the logs show an connection and the command 'nslookup' being used to find the Domain Controller, then instantly disconnecting. 

Thankfully, these servers are backing up to a separate cloud storage container quite frequently, meaning we can see the files that have been captured by the backup, even though the attackers try and clean up after themselves by periodically deleting files. 

The three files the attacker place on the infected PC are:

<figure>
	<a href="/assets/images/net64.png"><img src="/assets/images/net64.png"></a>
</figure>

As mentioned, this the network scanning tool, licence, and configuration file, running the net64.exe also confirms the language of the attackers:

<figure>
	<a href="/assets/images/tool.png"><img src="/assets/images/tool.png"></a>
</figure>

The executable seen in the image above, ‘net64.exe’ is a portable network scanner that can be deployed by an automated process with a licence and configuration file. The configuration file included in this case shows that the tool is setup to search across the network for the following IP ranges and ports:

<figure>
	<a href="/assets/images/config.png"><img src="/assets/images/config.png"></a>
</figure>

The tool will scan everything on the given network range from 192.168.0.0 to 192.168.255.255 for live hosts. Once a live host has been found it will check for open port numbers in the above image. 
If the tool identifies open ports such as remote terminal services, it will attempt to use stored credentials and authenticate to the service, reporting if successful or not.  

### Further Exploitation 

Once the attackers became aware that they had access to the network via their automated tools, they proceeded to carry out further exploitation techniques, such as password dumping. To do this the attackers placed known exploitation tools such as Mimikatz and NPRW on the DC. Attackers commonly use Mimikatz to steal credentials and escalate privileges: in most cases, endpoint protection software and anti-virus systems will detect and delete it. 
NPRW stands for Network Password Recovery Tool and is used to scan the system for user stored credentials. This is a commercially available piece of software that appears to originate from Russia. 

<a href="https://www.passcape.com/network_password_recovery">Checkout NPRW here</a> 

When the attackers attempted to run this tool, one of the processes it uses triggered the Cylance end point protection as a known malicious tool. 

In total, thirty-seven attempts were detected of the process ‘loader64.exe’ 

During the investigation process it was also noticed that the attack team were systematically removing tools in attempt to remain undetected. Various backups where showing that tools being installed were quickly deleted not long after use, so daily backups are essential!

### Cylance Detection

It is worth noting that whilst Cylance has detected multiple instances of various threats, it did allow for attacks to get through. Which will be detailed shortly. 

### Data Exfiltration 
