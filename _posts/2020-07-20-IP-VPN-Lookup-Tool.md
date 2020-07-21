---
title: "Tool: VPN IP lookup tool"
excerpt: "Is that IP in your logs a VPN?"
header:
  teaser: "/assets/images/iphub.png"
tags: 
  - VPN
  - IP
  - Threats
---

*Use this tool to check for the existence of VPNs in your logs. Use cases Office 365, SFTP providers and so on.*

#### Introduction

<figure>
	<a href="/assets/images/iphub.png"><img src="/assets/images/iphub.png"></a>
</figure>

Head to https://iphub.info/api and grab yourself a free API key, remember to adhere to the rate limiting policies. 

I tend to use Excel to create a list of IPs in quotes eg:
"1.1.1.1" "2.2.2.2" "3.3.3.3." and paste into the script, then output into a format that will allow you to search across whatever you are using for analysis. 

Standard output by default is json. 

#### Code

clone from <a href="https://github.com/SecRedOps/VPN-Finder">here</a>

Create a bash script with nano/vim  chmod +x and run

```html
#!/bin/bash
## -a = declare an array variable
## change the IP in brackets to what you want to search for, no need to comma separate multuple IPs, just use qoutes
echo "check if an IP is a VPN with iphub - get your API key here:https://iphub.info/api"
echo "use with caution if blocking IP addresses"
echo
echo "block: 0 - Residential or business IP (i.e. safe IP)"
echo
echo "block: 1 - Non-residential IP (hosting provider, proxy, etc.)"
echo
echo "block: 2 - Non-residential & residential IP (warning, may flag innocent people)"
echo
declare -a arr=("1.1.1.1")

## now loop through the above array
for i in "${arr[@]}"
do
   curl -s -S http://v2.api.iphub.info/ip/$i -H "X-Key: place your key here" | json_pp && sleep 3
done
```

And you'll get something that looks like this:

#### Output

```html
check if an IP is a VPN with iphub - get your API key here:https://iphub.info/api
use with caution if blocking IP addresses

block: 0 - Residential or business IP (i.e. safe IP)

block: 1 - Non-residential IP (hosting provider, proxy, etc.)

block: 2 - Non-residential & residential IP (warning, may flag innocent people)
{
   "countryCode" : "AU",
   "asn" : 13335,
   "isp" : "CLOUDFLARENET",
   "hostname" : "1.1.1.1",
   "countryName" : "Australia",
   "ip" : "1.1.1.1",
   "block" : 1
}
```

Happy threat hunting

Note: if you find a false positive and the ASN is not a proxy/VPN provider, please report it to iphub and they will update their records. 
