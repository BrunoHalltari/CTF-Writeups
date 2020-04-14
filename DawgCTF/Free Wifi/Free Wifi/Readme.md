Free Wifi 1 :

The  challenge featured a pcapng file to download, after checking it for  a while i found a GET request to /staff.html 

![Ricgiesta](https://user-images.githubusercontent.com/59454895/79156182-54ee7180-7dca-11ea-84f4-f4599f7dfe48.PNG)

so i just tried to make a get request to freewifi.ctf.umbccd.io/staff.html and i got the flag  in the main web-page :) .


Free Wifi 3 :

In this challenge ,to start, i checked a POST request to /forgotpassword.html .
After looking inside the html code of the forgot password page i found something interesting

![Meglio](https://user-images.githubusercontent.com/59454895/79161259-11e4cc00-7dd3-11ea-9787-e99cfbf8438f.PNG)


This is an email value of an ipotetic user of the web site!
Now I just put the email true.grit@umbccd.io inside the text box and I pressed "I forgot my password" Intercepting the request with burp.
It was interesting because in the header i found the email that i used before , but  with the owner's username :D and that's really good.
Why is it good ? 
Because without a proper validation we can manipulate the request header modifying just the email value and leaving the username unchanged , so the website believes we are a legitimate user while an attacker( in this case me XD ) could receive the authentic link to change the password.



In the end I changed the email in the request taken with burp with one of my property, leaving the username unchanged. I sent the modified request and received the flag :

![Greve](https://user-images.githubusercontent.com/59454895/79158863-fa0b4900-7dce-11ea-8393-51736cf4aec0.PNG)



Free Wifi 4 :

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

It went well and the web page printed the flag :  DawgCTF{k3y_b@s3d_l0g1n!}
