## Cookie  Forge :

The description of this challenge was :
" Help a new local cookie bakery startup shake down their new online ordering and loyalty rewards portal at https://cookie-forge.cha.hackpack.club!

I wonder if they will sell you a Flask of milk to go with your cookies... "

![bella 5](https://user-images.githubusercontent.com/59454895/85864768-3db60200-b7c5-11ea-952d-4d1f70872df4.PNG)

First of all i tried to go on " Flagship Loyalty " but all i got was this : 

![Noflag](https://user-images.githubusercontent.com/59454895/85865093-aac99780-b7c5-11ea-82b9-5bcfb1a9da42.PNG)

Using burp i managed to intercept a cookie field , also there was a hint about flask in the description and and  a huge cookie on the main page , so i realized they were using flask to generate the token . I used flask-unsign to read the cookie , launching this comand :
" flask-unsign -d -c eyJmbGFnc2hpcCI6ZmFsc2UsInVzZXJuYW1lIjoidGVzdCJ9.Xpn4fg.FHIOgRaiS8-WKoGU1vX5_b9h5q4 " and it gave me {'flagship': False, 'username': 'test'}.

I knew we had to change the flagship value  but every cookie as a session password , i just bruteforced the password using flask-unsign ( the result was " password1" ) and made a cookie where i had this values {'flagship': True , 'username': 'test'} .



To finish this challenge  i encoded my own session with flagship set to True

![CookieToken](https://user-images.githubusercontent.com/59454895/85865297-f8de9b00-b7c5-11ea-9088-dbeb39d9aadc.PNG)

After sending the request with my session i went to Flagship Loyalty and the flag was served :

![bella4](https://user-images.githubusercontent.com/59454895/85866326-82429d00-b7c7-11ea-8aef-d028385476e7.PNG)

Flag: flag{0hmy@ch1ng_p@ncre@5th33$3@r3_d3l1c10$0}
