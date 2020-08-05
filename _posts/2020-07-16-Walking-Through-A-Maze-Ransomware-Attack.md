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