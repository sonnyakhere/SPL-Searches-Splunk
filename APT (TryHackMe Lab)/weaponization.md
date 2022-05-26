# WEAPONIZATION
---
### Using OSINT to Identify Attacker Infrastructure
- ### [Robtex](http://www.robtex.com/)
	Let’s start by navigating to www.robtex.com and entering in the domain that we know was associated with the defacement attack.
	
	Robtex will return a number of pieces of information around the domain in question to include values like IP addresses and associated domains. Historical information, whois and graphical representations are available as well but other OSINT sites may have greater fidelity on individual components so don’t just use one site. Find the ones that work the best for you.
	
	As we can see in this search, we immediately see an IP address that is associated with our FQDN. Let's drill into the IP to see what information is associated with it.
	
	When we pivot into the IP address, we can see that this IP has a number of other domain names associated with it as well. In fact, a quick glance shows a few that look a bit like Wayne Enterprises. We should note these domains for future investigation.

- ### [ThreatCrowd](http://www.threatcrowd.org/)
	Now that we have the relationship between the IP address and domain, we could leverage a tool like ThreatCrowd.org to visualize the relationship between the IP address and other domains. Having multiple OSINT tools to leverage is key. This one does a nice job of visualizing relationships, whereas Robtex.com provides lots of good information, but their visualizations aren’t as rich.

- ### [VirusTotal](http://www.virustotal.com/)
	VirusTotal is another source for information. We could enter our domain into VirusTotal to see if there is Passive DNS resolution back to an IP address. If there is an IP, we could search VirusTotal for that IP address to find domains the IP is associated with as well as any associated URLs that have AV detections.
	
	When we search VirusTotal for our IP address, we see all of the additional domains associated with it. We saw similar information in robtex.com and threatcrowd.org, but this gives us resolution dates and more.


### Using OSINT To Create Linkages Between Email and Infrastructure
- ### [VirusTotal - Domain Information](http://www.virustotal.com/)
	As we inspect the whois records, we notice that we have registrant contact information including an email address. While there are a lot of things you can lie about within a DNS record, the email is something that is real. The bill for the domain goes to that address and the registrar's primary means of communication is via this address. We will probably want to look at other associated domains and see if there are similar email addresses or contact information located in them.

- ### [DomainTools for Validation](http://whois.domaintools.com/)
	To validate this, we can go to whois.domaintools.com and look up the po1s0n1vy.com domain and we immediately see an email address that is associated with eight domains.

### Using OSINT to Identify Associated Malware
- ### [TheatMiner](www.threatminer.org)
	We can start by going to threatminer.org and put our known IP address into the tool and see what we get. ThreatMiner provides the ability to input a number of threat attributes into the site to include domains, IP addresses, hashes, email addresses and more and can search for instances of those values across a number of data sources to include Malwr.com, VirusShare, VirusTotal and more. If we scroll through our results, we find related malware samples with their MD5 hashes and the various AV engines that detected them. Let’s click on that MD5 hash.

- ### [VirusTotal - File Hash](http://www.virustotal.com/)
	We are partial to VirusTotal, but you could take that hash and just Google it if you wanted to. We could open VirusTotal through the ThreatMiner interface or we can search directly in VirusTotal by searching for the SHA256 hash.

- ### [Hybrid-Analysis.com](https://www.hybrid-analysis.com/)

	In this case, we used the initial attack infrastructure to pivot from that IP address to identify any malicious samples that are out in the wild and then corroborated our findings on other threat intel sites. The screenshot shows that validation by leveraging Hybrid-Analysis.com in a manner similar to what we did with VirusTotal.
	
	Malwr.com and VirusShare.com are also good sites to check out.
