# ALERTS

<img width="2248" height="488" alt="image" src="https://github.com/user-attachments/assets/cb87a897-1f0d-44fa-b7b3-4c2e4eb2dda3" />

<img width="681" height="665" alt="image" src="https://github.com/user-attachments/assets/d8be2c88-c42e-4a2f-8936-b3a96af2fbb1" />
<img width="2260" height="477" alt="image" src="https://github.com/user-attachments/assets/41bd59f5-4363-45dd-aab6-ff4de7c50277" />
<img width="700" height="730" alt="image" src="https://github.com/user-attachments/assets/70431a4a-9ef9-4c3f-8b4d-6c0df3279fc4" />

<img width="711" height="238" alt="image" src="https://github.com/user-attachments/assets/ffefbf73-8c38-4459-91ba-d829078847bd" />

### We know our suspected ip is 10.8.15.133
So we focus on that as a source.ip and destination.ip
<img width="3152" height="245" alt="image" src="https://github.com/user-attachments/assets/070e2441-f9fe-4120-8276-94d7b45b4f30" />


### Let's check out what network protocols are being used.

<img width="864" height="976" alt="image" src="https://github.com/user-attachments/assets/a98d6247-0f5b-4d4b-a1cc-e6010b70f6f0" />

We see that HTTP has the most traffic
Next we'll filter to view only HTTP traffic from the infected host

Because we we filtered on "network.protocol:http" we were not able to see fields like http.uri or http.method because those are coming from Zeek datasets and our current search was searching suricata flow records.

<img width="670" height="854" alt="image" src="https://github.com/user-attachments/assets/ccbef79a-eef1-4302-a9b8-d84c78b67aea" />
<img width="659" height="515" alt="image" src="https://github.com/user-attachments/assets/36362a3b-3fda-4fa1-aa39-2b6cbb98c8f4" />


So we removed the network.protocol tag
Now we can see data
<img width="3156" height="942" alt="image" src="https://github.com/user-attachments/assets/2f981165-89f6-4a86-93e2-231eee2171bf" />

### Question
Why does "event.dataset:zeek" not return results but "event.dataset:zeek.http" does?
<img width="687" height="854" alt="image" src="https://github.com/user-attachments/assets/b00680c8-1e99-45d0-b000-8369dd23b896" />
##########################################

Next we looked at http and took a look at what websites we being visited at the http.virtual_host 
<img width="1197" height="927" alt="image" src="https://github.com/user-attachments/assets/002cb1e2-8686-49a1-bb38-e4cd9753c92a" />

We see the IP 72.5.43.29 being visited heavily, which is suspicious.
ChatGPT walkthrough wanted us to look at 'quote.checkfedexexp.com' at first since we fed it the answers beforehand but it looks like FEDEXEXP.COM was the initial infection.

<img width="656" height="714" alt="image" src="https://github.com/user-attachments/assets/82864724-1261-4bf4-9bc1-3e740388da7b" />

Now let's look at HTTP.METHOD to see top values,

<img width="921" height="906" alt="image" src="https://github.com/user-attachments/assets/664e078e-50c8-42f6-a1ee-895c61cadcab" />

Now we can add the HTTP.METHOD as a column, we don't want to filter on a specific request because we might miss the others.

<img width="3169" height="994" alt="image" src="https://github.com/user-attachments/assets/f8ca8d23-3fc6-4337-9669-f9ad7d0ee2ec" />

Next we can add HTTP.URI AND HTTP.VIRTUAL_HOST

HTTP.VIRTUAL_HOST = tells us who the victim talked to - the domain name

HTTP.URI = what the victim requested on that server, the exact path

Also changed timestamp to ascending order

<img width="3159" height="1074" alt="image" src="https://github.com/user-attachments/assets/3a5f88fb-d9b9-4d02-9ad9-7334f5b11f75" />

Here we see a GET request for a /data/ file from the suspicious IP 72.5.43.29.
ChatGPT informed us that this was the malicious DLL download but we want to know what happened before this as the initial infection.

<img width="663" height="721" alt="image" src="https://github.com/user-attachments/assets/1d0bfd1e-52d6-46f3-affa-a705c1377d7c" />

We also remove the IP filter because attacks can occur in stages that involve multiple IP's and if we only filter on one IP, we might miss the otherstages of the attack

<img width="678" height="807" alt="image" src="https://github.com/user-attachments/assets/9cea30e9-218b-4f3a-8d01-60131ff951de" />

<img width="3151" height="987" alt="image" src="https://github.com/user-attachments/assets/c20acdbb-864e-4775-831d-50422d748721" />

So here I can see one of the earliest GET requests is to the quote.checkfedexexp.com for a uri,
Maybe I could've crossed checked this with the earlier HTTP.VIRTUAL_HOST and thought it looked suspicious

<img width="459" height="498" alt="image" src="https://github.com/user-attachments/assets/ea1fcd65-5676-4fe1-8fcf-0c94409f642e" />

<img width="843" height="1315" alt="image" src="https://github.com/user-attachments/assets/aa84ea2f-7d4b-4fea-926d-6ffa5b06411e" />

### We were attempting to understand the sequence of events 

1. Initial Infection
2. Payload Delivery
3. C2 Behavior

I had the question of if I see a document in Kibana, say the GET request for /data/0f60a3e from the 72.5.43.29 and before that is the GET request for /management?16553a25.. from the webiste "quite.checkfedexexp.com", how do I know which one triggered the alert in Security Onion with the malicious MZ executable.

ChatGPT told us to not identify the executable by guessing the URI name, but by lining up the timing + server + behavior

So in Security Onion we see the MZ alert and note 3 things:

src_ip (who sent the file) = 72.5.43.29
dest_ip (victim) = 10.8.15.133
@timestamp (when) = 2024-08-15 00:12💯

Then we go to Zeek HTTP logs (event.dataset:zeek.http) and search with:

Then we'll zoom in on our time range to just around the alert time.
Find Zeek entries that involve the smae server IP
-Look for http.virtual_host:72.5.43.29 (or destination IP showing 72.5.43.29)

Look for the HEAD then GET pattern from near that time:

That pattern strongly suggests: "check file" -> download file"

Confirm it's the likely binary transfer by adding/looking at these fields in Zeek

http.response.body.length (large-ish is more file-like)
http.resp_mime_types (sometimes shows application/*)
http.status_code (200 is common for successful download)

<img width="2475" height="936" alt="image" src="https://github.com/user-attachments/assets/74dd22e4-3f5d-4106-82b8-f095e36ee870" />

Now separate the the earlier state (lure site/initial infection)

Requests to quote.checkfedex.com with /managements? are more "webpage-ish" (parameters, long query strings). That's why we treat it as delivery/lure, not the exe itself

Had to change our time in Kibana from browser to UTC
<img width="3381" height="1087" alt="image" src="https://github.com/user-attachments/assets/50ec596d-0319-4793-a015-44e0141249bb" />

<img width="2256" he<img width="3036" height="278" alt="image" src="https://github.com/user-attachments/assets/b620bb89-5842-429c-9070-c82feb44a0d4" />
ight="289" alt="image" src="https://github.com/user-attachments/assets/ee3a29e3-bef3-4bf7-af90-73f63cc9bb82" />

So looking at these alerts, 2024-08-15 00:12:00 is 7 hours ahead our alerts in Kibaba 2024-08-14 17:11:59 

So if our Sec Onion alerts shows a timestamp of ~17:12 on 2024-08-14 we can look at Kibana and see a GET request that received a response of 159.232 bytes, which most likely triggered the Sec Onion alert

<img width="3034" height="531" alt="image" src="https://github.com/user-attachments/assets/b53931c9-3062-4d4f-bb3e-745bac649784" />

<img width="2371" height="373" alt="image" src="https://github.com/user-attachments/assets/9f036f56-7032-4fc2-8db9-c7d4f2937a64" />

ChatGPT told us that the /managements?16553 with all it's parameters is indicative of a webpage but I wanted to know what to look for to be sure so we can add the filter "file.resp.mime.types" as a column and we see it says it's a zip file, which is common in malware delivery

We also see that the /data/ executable payload we narrowed down earlier says it's x-dosexec

Sequence of Events:

quote.checkfedexexp.com  → delivers ZIP (initial payload container)
72.5.43.29 /data/...     → delivers EXE (actual malware)

Now we can verify beaconing behavior and isolate only the suspected C2 traffic

We can remove the lure stage and show only post-infection traffic

<img width="1539" height="127" alt="image" src="https://github.com/user-attachments/assets/ff74b1b0-8c4d-4751-8b6f-50efd4ce2757" />

Next we look for the beaconing patterns

<img width="687" height="222" alt="image" src="https://github.com/user-attachments/assets/b73fea69-bc91-4f56-b7c9-5f5727f2c9f8" />

Which we have already

<img width="2350" height="567" alt="image" src="https://github.com/user-attachments/assets/9c65e46e-5f2e-4898-8d96-ad85da57b19a" />

<img width="699" height="657" alt="image" src="https://github.com/user-attachments/assets/ab590f6a-a63d-4b72-b525-3a331cf8aa56" />

<img width="2695" height="469" alt="image" src="https://github.com/user-attachments/assets/b13d364f-635d-4a83-bf39-751501238672" />

The chart is showing number of HTTP events over time

*Doesn't care what kind they are, just counting requests"

<img width="600" height="202" alt="image" src="https://github.com/user-attachments/assets/4c8ce940-e17c-453d-9606-0ea5ef2d52b5" />

### To verify infection was limited to only one host

Temporarily remove the KQL query for 10.8.15.133

Search for only malicious ip

<img width="2135" height="855" alt="image" src="https://github.com/user-attachments/assets/c47496ac-41fc-483a-bc66-a512d3e84273" />

Then group by source.ip

<img width="889" height="743" alt="image" src="https://github.com/user-attachments/assets/ab158c4a-10c2-4263-97a0-3f74dfe1eb76" />

We only see the 10.8.15.133

Double Checking Suricata Alerts

event.dataset:suricata.alert AND 72.5.43.29

<img width="2489" height="981" alt="image" src="https://github.com/user-attachments/assets/ee12b404-d651-4622-b030-0e0219da4d01" />

Add source ip column and we should only see the internal ip 10.8.15.133

























































