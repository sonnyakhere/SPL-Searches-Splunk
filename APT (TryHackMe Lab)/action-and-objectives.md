# ACTION AND OBJECTIVES
---
### Identifying the File that Defaced Our Web Server
- ###  Looking at Directional Flow of Data - Part 1
````
```
index=botsv1 dest=192.168.250.70 sourcetype=suricata
````

Let’s start by taking a look at our web server to determine who is communicating with it. Using our Suricata data and the IP address that we identified earlier for our web server (192.168.250.70), we can see there are 3 internal systems communicating with the web server, but there are no external systems originating traffic and generating alerts. Let’s see what it looks like if the web server is the source of the traffic. That’s not something we would expect to see, but it’s still important to check.

- ###  Looking at Directional Flow of Data - Part 2
````
```
index=botsv1 src=192.168.250.70 sourcetype=suricata
````

In this search, we are treating the web server as the source of the traffic instead of the destination. Is there anything interesting about the results that you see?

It's interesting that the destination IP address values contain three external IP addresses that our web server is originating traffic to. Would we expect a web server to initiate traffic outbound, especially to an external IP address? Is that something that a web server does?

A web server listens for requests and sends data back based on the request. Generally, when observing network or web traffic, the client or browser would be the source IP or host while the web server would be the destination. If an administrator logged onto the web server, executed a browser and started surfing the web, that could be one reason that the web server would be the source in an event, but that action generally violates policy, so at the very least, that should be looked into.

In this case, because we see a large volume of logs that indicate the webserver is communicating out to external sites, this might indicate some sort of longer term or larger volume data connection.

- ###  Pivot Into Destination IP Addresses to View URLs
````
```
index=botsv1 src=192.168.250.70 sourcetype=suricata dest_ip=23.22.63.114
````

We can pivot into the external destination IP addresses and view other interesting fields. With 23.22.63.114 as our destination, we notice that http.url has two php files and a jpeg. We previously saw traffic associated with this address, but let's capture this new activity. Nearly all of the events have a value for http.url but there are only 3 results. Let’s jot those down and then start looking at our source for web data using sourcetype of stream:http to see if we can get any corroborating information.

- ###  Do Web Servers Start A Conversation?

With the data collected so far and what was just discussed, let’s stop and think critically about our web server traffic. Does it generally start a conversation or respond to a request?

As we start digging deeper here, one of the questions that we may want to stop and think about is how might a file end up on our web server. If we hypothesize that the file ended up there due to information in our http traffic, we know there are a couple of ways that http data is moved. The most common ways are via http GET and http POST. You can see in the link a picture of the data flow and which system initiate the communication. This is important to keep in mind, particularly if we decide to specify source IP versus destination IP in our searches.
Web Data Flow (POST v GET)
````
```
index=botsv1 imreallynotbatman.com sourcetype=stream:http
````

If we modify our search to look at stream:http wire data, we notice very few logs originating from our web server, but they are there.

- ###  Any Similarities between Suricata and stream:http?
````
```
index=main src=192.168.250.70 sourcetype=stream:http
````

Looking at the http traffic that returned those nine results, we can pivot into interesting fields like the URI field in the same way we did with suricata. If we look through the list of URIs we see in the two sourcetypes, we do see some differences, but we also see some similarities. Do you see anything in common?

### Validating Using Firewall Logs the File that Defaced Our Web Server
- ###  What About Firewall Data?
````
```
index=botsv1 sourcetype=fgt_utm "192.168.250.70"
````

If we search our Fortinet UTM for the IP of the web server, notice we get a couple of IPs of interest, both from a source and destination perspective.
	PIVOTS:
	- src
	- dest

- ###  Which Search Do I Start With?

If we specify the direction of the communication path but limit the search using the NOT command, we get result sets of 14K and nine events. Which one should we dig into first?

Let's start with the nine events. If nothing else, we can eliminate them from consideration quickly, but based on our analysis with the other sourcetypes, we should have a strong suspicion that the web server originated the traffic flow.
````
index=botsv1 sourcetype=fgt_utm "192.168.250.70" NOT dest="192.168.250.70" | stats count
````
````
index=botsv1 sourcetype=fgt_utm "192.168.250.70" NOT src="192.168.250.70" |stats count
````

- ###  Use Web Site Categorization to Filter
````
```
index=botsv1 sourcetype=fgt_utm "192.168.250.70" NOT dest="192.168.250.70"
````

UTM (Unified Threat Management) devices (or next-generation firewalls) often rate or classify various web sites much like standalone web filtering gateways (e.g. Blue Coat). In this case, if we pivot down on category, we see of the nine connections the UTM logged, 3 were to malicious websites. Those sound much more intriguing than the IT sites, so let’s drill down on them.

- ###  Firewall Gives Us Confirmation
````
```
index=botsv1 sourcetype=fgt_utm "192.168.250.70" NOT dest="192.168.250.70" category="Malicious Websites"
````

Those three logs show us that the file_path has that same jpeg that both Suricata and stream:http saw. Based on that, it appears the presence of this jpeg in the appropriate directory defaced our web site.
