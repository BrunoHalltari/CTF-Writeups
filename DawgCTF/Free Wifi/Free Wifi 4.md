Free Wifi 4

On the /staff.html page as well as logging in with a username and password you could use the wifi key,so i decided to look some request on the /staff.html page from the pcapng file.
Looking better i discovered a WifiKey nonce parameter encoded in base64 from the cookies section ( you were also notified that the hashing algorithm for wifikey was sha1)

![Cookie](https://user-images.githubusercontent.com/59454895/79171575-4a43d480-7dea-11ea-8c72-0d5118723279.PNG)

After doing the sha1 of the nonce i noticed that the first eight characters where the same as the login token , here is an example from a previous login (i got it always from the pcapng file provided by the authors of the challenge)

Nonce: WifiKey nonce=MjAyMC0wNC0wOCAxNzowMQ== . with sha1 it becomes 5004f47ae3e2e7c1c9a5ea4d1666f95e6b06b062
Login token (of the same session) : ImE4OGVkMWY1ZDg4YWU4MmQxMzFmODg4ZmVhMWY2MDQ0ZjUxMDA4MjAi.Xo4DYg.fnXZhEmbQPzEgFWLgEfjSTeqX2Y&passcode=5004f47a

![Cookie2](https://user-images.githubusercontent.com/59454895/79172677-45345480-7ded-11ea-8307-a4a65f6dbe2f.PNG)
![TokenCsrf](https://user-images.githubusercontent.com/59454895/79172740-6f861200-7ded-11ea-960a-eda5ee154567.PNG)

If you look better the passcode in the token you will notice that the passcode is the same as the first eight characters of the  nonce with the SHA1 (5004f47a).

Now we can be sure that the first 8 characters of the sha1 are used to generate the passcode and finally use  this method to log in with the wifi key :D ! .

To complete this challenge  i made a request from my browser to /staff.html intercepting the request with burp in order to take the wifikey nonce of the session, i did the sha1 of the nonce and I put the first eight characters in the "wifiKey" text box trying to login.

it went well and the web page printed the flag :  DawgCTF{k3y_b@s3d_l0g1n!}
