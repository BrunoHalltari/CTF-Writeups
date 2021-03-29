
## Intigriti — XSS Challenge 

![Capture](https://user-images.githubusercontent.com/59454895/112804703-fff84980-9074-11eb-972e-85a85c837655.PNG)

## Let's start

The challenge website had an input field to enter notes where we could write what we want.
First of all i tried to fuzz in order to see how the challenge would react , the first thing i noticed it was a csrf token and our notes in the post request,
i also noticed that the backend checks the submitted note. If we send a special character to escape the HTML element, such as <  , the server would sanitize with the  htmlspecialchars() PHP function, which should prevent XSS attempts on this field.

Fortunately during my test i also tried to put an email in the note field wich helped me to find hidden feature.

![Capture3](https://user-images.githubusercontent.com/59454895/112806481-18696380-9077-11eb-95b0-8221d5b8cc54.PNG)

I tought it was interesting bacuse maiby we could escape since the email address appears inside the href attribute of an <a> tag.
After i decided to read how "mailto" works from the rfc , and i found out there is a strange way to declare the email:
``` "\\\"it's\ ugly\\\""@example.org; ```
 It was a very good find , because it could allow to inject javascript between ``` "\\\"it's\ ugly\\\" ```  and ``` "@example.org; ```
  
the final payload was :
``` "\\\"traffik\prova\\\"\ onmouseover=alert('flag{THIS_IS_THE_FLAG}');"@outlook.it  ```

The following payload breaks out of the attribute and adds an onmouseover event.
![Capture4](https://user-images.githubusercontent.com/59454895/112809118-fae9c900-9079-11eb-951b-3f9bd7a7ae00.PNG)


## CSRF Bypass
The challenge has  CSRF token(in this case hashed with MD5) built in. The page generates a token and stores it in a hidden form input. When the form is submitted, the server checks this token matches in order to ensure the form hasn’t been submitted by a malicious site. 

But that's not all , there was also a comment with a timestamp and it helped me a lot , because after a research i understood it's a typical mistake to take the current UNIX timestamp, hash it and use it as a CSRF token because is easy to predict

![Capture5](https://user-images.githubusercontent.com/59454895/112812171-15717180-907d-11eb-91a2-a6b0f9df6459.PNG)




