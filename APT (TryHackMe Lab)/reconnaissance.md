# RECONNAISSANCE
---
### Finding the IP Scanning Your Web Server
````
index=botsv1 imreallynotbatman.com
````
- Identify sourcetypes Associated with Search Values 
- Identify and select Source Addresses

### IP Validation
````
index=botsv1 imreallynotbatman.com src=40.80.148.42 sourcetype=suricata
````

Because we can see what appear to be web server signatures firing, we can infer that this IP address is indeed scanning our website. Using the data returned from another data source helps us validate our findings.

 ### Another Option: Search Stream sourcetype and Count the src_ip
````
index=botsv1 imreallynotbatman.com sourcetype=stream* | stats count(src_ip) as Requests by src_ip | sort - Requests
````

Keep in mind there are a number of ways to conduct these searches so there isn’t a single right way to find the answer. Using our answers from another data source helps us validate our findings.

### Identifying The Web Vulnerability Scanner
 - ### Looking at src_headers
````
index=botsv1 src=40.80.148.42 sourcetype=stream:http
````

If we zoom in on the source headers, we can see information that says (Acunetix Web Vulnerability Scanner - Free Edition), so our antenna is up based on that clue.

 - ### Looking at http_user_agent strings
````
index=botsv1 src=40.80.148.42 sourcetype=stream:http
````

We can take this a step further and look in other fields of the HTTP wire data. If we look at http_user_agent, we see some anomalous/unusual agent strings and in a few of them we again see Acunetix.

Unfortunately, I may not be well versed in web application vulnerability scanners. What’s an analyst to do? Use Google.

### Determining Which Web Server is the Target
 - ### Digging into the URI
````
index=botsv1 dest=192.168.250.70 sourcetype=stream:http
````

Now that we know the sourcetype AND we know the IP of the web server, we can start looking at URLs or URIs to get an idea of the kind of files and directory structures being requested. As we look through the list, we see the word joomla on a number of these directory trees. I don’t know a person named joomla in the office, so where should we go to learn more?

Google for Joomla

It's an open source CMS system. Sounds like we are on the right path.
- Examine URIs On the Web Server IP

 - ### Looking for Confirmation
````
index=botsv1 dest=192.168.250.70 sourcetype=stream:http status=200
````

Let’s take our original search to find our URIs and improve it a bit. We are going to add a status of 200 to our search to indicate successful page loads.
````
| stats count by uri
````

We can also add a transformational search command called stats that will allow us to count the number of events grouped by URI. stats is fantastic and we will be using it throughout this workshop in a number of ways. In fact, we have already been using stats in our initial searches when we looked at counts of fields that we have been inspecting.
````
| sort - count
````

Finally, we are going to issue another transformational command called sort and return results in descending order based upon the count so that our result set is formatted in the manner we would like to see it.

When we get our results back, we see the vast majority of URIs on the server start with "joomla" in the URI.

 - ### Finding the Answer With IIS
````
index=botsv1 sourcetype=iis sc_status=200 | stats values(cs_uri_stem)
````

There are many ways to arrive at this answer. As we found earlier, imreallynotbatman.com is hosted on 192.168.250.70. We have IIS logs for that host (we1149srv). We could search the IIS logs and examine the URI strings being accessed to find indicators and verify that the http response code equals 200.
