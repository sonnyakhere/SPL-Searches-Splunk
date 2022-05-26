# COMMAND AND CONTROL
---
### Identifying the Fully Qualified Domain Name of the Attacker
- ###  Using Fortigate Firewall Events We Just Found
````
```
index=botsv1 sourcetype=fgt_utm "poisonivy-is-coming-for-you-batman.jpeg"
````

If we look at our firewall/UTM events and expand the result we can see the destination IP address, the defaced file and the URL. That URL likely provides us the FQDN that we are looking for. Verification is always good though.

- ###  What Other Data Sets Saw This File?
````
```
index=botsv1 dest=23.22.63.114 "poisonivy-is-coming-for-you-batman.jpeg" src=192.168.250.70
````

Because we know the file in question, we can add it to our search and see if any other sourcetypes contain data with that file name. If you recall from the earlier question, multiple sources saw it but let’s double check.

- ###  Using stream:http
````
```
index=botsv1 dest=23.22.63.114 "poisonivy-is-coming-for-you-batman.jpeg" src=192.168.250.70 sourcetype=stream:http
````

If we drill into the stream:http sourcetype and then scroll down to the URL field, we can see two events have the same URL with the same FQDN that we saw with Fortigate, so we likely have our domain.

- ###  What if I Don’t Have the File Name?
````
```
index=botsv1 answer=23.22.63.114 sourcetype=stream:dns | stats values("name{}")
````

We can still find the FQDN even in the absence of the filename. Because we know the IP address of 22.23.63.114 is of concern, we can use Splunk to search DNS and look for DNS events where 22.23.63.114 was the answer. From there, we can use the stats command to return values of the name{} field to find the domain.
