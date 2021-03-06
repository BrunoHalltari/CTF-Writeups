## Free Wifi 1 :

The  challenge featured a pcapng file to download, after checking it for  a while i found a GET request to /staff.html 

![Ricgiesta](https://user-images.githubusercontent.com/59454895/79156182-54ee7180-7dca-11ea-84f4-f4599f7dfe48.PNG)

so i just tried to make a get request to freewifi.ctf.umbccd.io/staff.html and i got the flag  in the main web-page : 

![Prima](https://user-images.githubusercontent.com/59454895/79245164-a7359e00-7e6f-11ea-9bf0-56f45a480e71.PNG)+

Flag: DawgCTF{w3lc0m3_t0_d@wgs3c_!nt3rn@t!0n@l}





## Free Wifi 3 :

In this challenge ,to start, i checked a POST request to /forgotpassword.html .
After looking inside the html code of the forgot password page i found something interesting

![Meglio](https://user-images.githubusercontent.com/59454895/79161259-11e4cc00-7dd3-11ea-9787-e99cfbf8438f.PNG)


This is an email value of an ipotetic user of the web site!
Now I just put the email true.grit@umbccd.io inside the text box and I pressed "I forgot my password" Intercepting the request with burp.
It was interesting because in the header i found the email that i used before , but  with the owner's username :D and that's really good.
Why is it good ? 
Because without a proper validation we can manipulate the request header modifying just the email value and leaving the username unchanged , so the website believes we are a legitimate user while an attacker( in this case me  ) could receive the authentic link to change the password.



In the end I changed the email in the request taken with burp with one of my property, leaving the username unchanged. I sent the modified request and received the flag :

![Greve](https://user-images.githubusercontent.com/59454895/79158863-fa0b4900-7dce-11ea-8393-51736cf4aec0.PNG)



## Free Wifi 4 :

On the /staff.html page as well as logging in with a username and password you could use the wifi key,so i decided to look some request on the /staff.html page from the pcapng file.
Looking better i discovered a WifiKey nonce parameter encoded in base64 from the cookies section ( you were also notified that the hashing algorithm for wifikey was sha1)

![Cookie](https://user-images.githubusercontent.com/59454895/79171575-4a43d480-7dea-11ea-8c72-0d5118723279.PNG)

After doing the sha1 of the nonce i noticed that the first eight characters where the same as the login token , here is an example from a previous login (i got it always from the pcapng file provided by the authors of the challenge)

Nonce: WifiKey nonce=MjAyMC0wNC0wOCAxNzowMQ== . with sha1 it becomes 5004f47ae3e2e7c1c9a5ea4d1666f95e6b06b062
Login token (of the same session) : ImE4OGVkMWY1ZDg4YWU4MmQxMzFmODg4ZmVhMWY2MDQ0ZjUxMDA4MjAi.Xo4DYg.fnXZhEmbQPzEgFWLgEfjSTeqX2Y&passcode=5004f47a

![Cookie2](https://user-images.githubusercontent.com/59454895/79172677-45345480-7ded-11ea-8307-a4a65f6dbe2f.PNG)
![TokenCsrf](https://user-images.githubusercontent.com/59454895/79172740-6f861200-7ded-11ea-960a-eda5ee154567.PNG)

If you look better the passcode in the token you will notice that it is the same as the first eight characters of the  nonce with the SHA1 (5004f47a).

Now we can be sure that the first 8 characters of the sha1 are used to generate the passcode and finally use  this method to log in with the wifi key :D ! .

To complete this challenge  i made a request from my browser to /staff.html intercepting the request with burp in order to take the wifikey nonce of the session, i did the sha1 of the nonce and I put the first eight characters in the "wifiKey" text box trying to login.

It went well and the web page printed the flag :  DawgCTF{k3y_b@s3d_l0g1n!}


## Free Wifi 2 :

The description of this challenge was " I saw someone's screen and it looked like they stayed logged in, somehow...  " and a pcap file was provided for donwload.

After looking for a while inside the pcapng file i found an  interesting request to /jwtlogin
![token](https://user-images.githubusercontent.com/59454895/79248691-0e555180-7e74-11ea-83c0-4340f2d79920.PNG)

I tried to make a request but all I got was this :

{
    "description": "Request does not contain an access token",
    "error": "Authorization Required",
    "status_code": 401
}
 So i realized that we have  to create a jwt token to get the authorization.

For hours I kept trying to create tokens that used the username field, since in the other challenges I had found an   email "true.grit@umbccd.io" , but nothing... So I tried to log in from /staff.html by inserting in the username field "true.grit@umbccd.io" without putting any password, intercepting the request with burp.

It was the right move because in the header of the error page i found this -----> " JWT 'identity'=31337; Path=/ "  and this is very useful for us because we know that the jwt token is using the identity and username field ( even if they are deprecated ).

To finish this challenge i just went again to /jwtlogin intercepting the request with burp , then i put the token in the header like this:

Authorization: JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6MzEzMzcsInVzZXJuYW1lIjoidHJ1ZS5ncml0QHVtYmNjZC5pbyIsImlhdCI6IjE1ODY2OTg2NDYiLCJleHAiOiIxNTg2Njk5NTE3IiwibmJmIjoiMTU4NjY5ODUxNyJ9._eJaJQszRDarG_lY_xu7Yt7nTksNzFiEBE1-N6B5eXY

After i sent the modified request with the jwt token i got the flag :

![ahahahah](https://user-images.githubusercontent.com/59454895/79263641-1d93c980-7e8b-11ea-891c-5c90d1c1c85a.PNG)

Flag: DawgCTF{y0u_d0wn_w!t#_JWT?}






