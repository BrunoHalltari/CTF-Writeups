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
