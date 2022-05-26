# INSTALLATION
---
### Identifying the Executable Uploaded
- ###  Search for EXEs in stream:http
````
```
index=botsv1 sourcetype=stream:http dest="192.168.250.70" *.exe
````

Starting with stream:http, we know the destination is the web server and we know its IP address. If we add the string "*.exe" to our search, we can narrow our search and then look into the part_filename{} field and see 2 filenames referenced; one exe and one php file. Note that we don't yet know the field name hrtr, but Splunk's full text search capabilities will find the file extension regardless of what field it is in.

- ###  Search for EXEs in Suricata
````
```
index=botsv1 sourcetype=suricata dest_ip=192.168.250.70 .exe
````

With Suricata, we can also look for .exe files with the destination IP of our web server. The additional field that we are looking for is called fileinfo.filename. In this case, we see two exe files. One has the same name that we saw with stream:http, the other looks like something we would often see on an IIS webserver.

- ###  Hostnames v IPs
````
```
index=botsv1 sourcetype=suricata (dest="192.168.250.70" OR dest_ip="192.168.250.70") .exe
````

Fields may not have the same values based on the logging structure. For example, dest and dest_ip could have different values. Try using OR and parenthesis to bracket the search (dest=1.1.1.1 OR dest_ip=1.1.1.1)

- ###  When Destination and Destination IP are Different
````
```
index=botsv1 sourcetype=suricata (dest=imreallynotbatman.com OR dest="192.168.250.70") http.http_method=POST .exe
````

We can broaden our search to include the hostname of the server and the IP address. We can get results from both and see the associated filenames as well. While we may end up with some additional values, we also ensure that we are getting broader coverage and we can narrow our search from there if desired.

Using our Suricata search from earlier and then adding the http_method of POST further narrows the data down to three events. We have two files and by looking at them, one appears to be associated with Microsoft Front Page extensions. (If you don't believe me, Google "_vti_bin/shtml.exe".)

- ###  Taking the Extra Step and Capturing the Source of the Executable
````
```
index=botsv1 sourcetype=suricata dest_ip="192.168.250.70" http.http_method=POST .exe
````

Because the question asked about the executable uploaded by P01s0n1vy, we should probably check the source address. When we look at the source address, we find that the source of the .exe found is 40.80.148.42. This address was associated with other stages of the attack which helps drive our confidence that we have the file of interest.

### Determining the Hash of the Uploaded File
- ###  What sourcetype Should I Start With?
````
```
index=botsv1 3791.exe
````

If we don’t know which sourcetype to start with, we could run a search just looking for our file, 3791.exe. When we look at the sourcetypes, it appears we have a choice of five. We could either use a process of elimination and look at each sourcetype and attempt to determine which ones contain hash values or we could us the internet. Which one would you choose?

To short circuit this a bit, we know a little something about Sysmon data. If we click on the link in the resources about Microsoft Sysmon, we can see that one of the cool things that Sysmon can do is record the hash of the process image using a number of different hashing functions. Based on that, we can pivot into sysmon data and look there.

- ###  What Can I Find With Sysmon? Part 1 of 2
````
```
index=botsv1 3791.exe sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
````

As we inspect the interesting fields in Sysmon, we can see event descriptions that include image load, process create, network connect, all of which can give us insight into things happening on a specific Windows system, not only from a program execution perspective, but also from a network communication perspective. We can also see hashes for files that include MD5, SHA1 and SHA256.
	PIVOTS:
	- EventDescription
	- Hashes

- ###  What Can I Find With Sysmon? Part 2 of 2
````
```
index=botsv1 3791.exe sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
````

Sysmon can also return values like the command that was issued to start a process execution as well as the parent command line. This can be helpful to see chained processes where one process starts and then spawns subsequent processes. There’s a little foreshadowing here. 
	PIVOTS:
	- CommandLine
	- ParentCommandLine

- ###  Isolating on MD5
````
```
index=botsv1 3791.exe sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational EventCode=1
````

We can uncover a bunch of interesting data from Sysmon, but we need to find the MD5 of the file. Scrolling through our results we can select fields of interest that contain MD5, SHA1 and SHA256 values. We can also find CommandLine and ParentCommandLine values within Sysmon. These values are specific commands that were executed. If we focus on the EventCode field in Sysmon and search where that value is 1, we are able to see all of the Process Execution events. While this narrows down our result set, to identify the correct MD5, we need to search for 3791.exe in the command line field, as that would capture the process starting.

- ###  Exploring the Sysmon Event
````
```
index=botsv1 3791.exe CommandLine=3791.exe
````

If we expand our event, we can see a number of values including EventDescription which is "process create," the MD5 value which we were looking for, as well as all of the other hashes, the directory where the command was executed, the parent command line which tells us which process spawned 3791.exe, as well as the host that Sysmon was run on.

These nuggets of information are useful so keep them in mind as we continue our investigation. This is one technique that can be used to identify hashes. Again, this can be very helpful when we don’t have direct access to the file itself.

- ###  Putting it Together
````
```
index=botsv1 3791.exe CommandLine=3791.exe
````

If we scroll up, we can see from our earlier search that there is only one event where CommandLine has a value of 3791.exe.
````
```
| stats values(MD5)
````

We can use the stats command with values function to return all the matches to a specific field, in this case the md5 field. Because there is only one md5 hash for this specific .exe in our data set, one value is returned. Is it possible to get multiple md5 hashes for the same file name?

If there are different versions of executables compiled, you can certainly have different hashes for those files. If we are looking for something more insidious like a malicious file called explorer.exe we could get a number of md5 hits, but using values allows us to get a single list of all hashes to then work with. Whitelisting could be used to isolate the known good hashes from the rest.
