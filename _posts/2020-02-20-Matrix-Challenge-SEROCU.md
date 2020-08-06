---
title: "CTF: The Matrix challenge provided to SEROCU + Met Police"
excerpt: "This is the CTF that would have been used for the Matrix challenge 2020"
author_profile: false
header:
  teaser: "/assets/images/MCTheMet.PNG"
tags: 
  - CTF
  - Met Police
  - Write Up
---

*CTF*

#### Matrix Challenge CTF write up 

<figure>
	<a href="/assets/images/MCTheMet.PNG"><img src="/assets/images/MCTheMet.PNG"></a>
</figure>

Here is the CTF challenge provided to the London Metropolitan Police for 2020. 

### Task

As the lead consultant you have been tasked with identifying Super Secure Startup's data is being accessed. 

You managed to pull some interesting files off one of Super Secure Startup's anonymous FTP servers. Via some OSINT work(a torrent or online Password breach site) you have also procured a recent data breach dump. 

Can you unlock the file and retrieve the key? 

Download the first clue here

### Answers 

Opening the 'web-developer-needed.docx' informs the player of the document author and the companies domain by checking document info, meta-data or just reading. 

Looking at the breach data file a password can be found, however, this is not correct due to the super secure savvy HR lady changing her password according to the seasons, a helpful hint is to note when the job advert doc was created. 

Once the key.doc is opened, the number 13 and a block of text appears. If the user gives the answer as 13 at this stage, they get some points/recognition, but have more to do. The key is in fact encoded in base64 13 times. 

There are two ways to decode this:

1. very simply google base64 decoder and decode 13 times
2. If no Internet is available then the user can try the following command on Windows with PowerShell by placing the base64 block into a foo.txt file
```sh
for($i = 0; $i -lt 13; $i++){$j=$i+1; certutil -decode foo0$i.txt foo$j.txt} 
```
Doing either of these reveals the actual code showing the challenge has been completed. 

For all of the files check here(link coming soon.)