We know our suspected ip is 10.8.15.133
So we focus on that as a source.ip and destination.ip

<img width="3152" height="245" alt="image" src="https://github.com/user-attachments/assets/070e2441-f9fe-4120-8276-94d7b45b4f30" />

<img width="864" height="976" alt="image" src="https://github.com/user-attachments/assets/a98d6247-0f5b-4d4b-a1cc-e6010b70f6f0" />

We see that HTTP has the most traffic
Next we'll filter to view only HTTP traffic from the infected host

Because we we filtered on "network.protocol:http" we were not able to see fields like http.uri or http.method becasue those are coming from Zeek datasets and our current search was searching suricata flow records.

So we removed the network.protocol tag
Now we can see data
<img width="3156" height="942" alt="image" src="https://github.com/user-attachments/assets/2f981165-89f6-4a86-93e2-231eee2171bf" />

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





















































































