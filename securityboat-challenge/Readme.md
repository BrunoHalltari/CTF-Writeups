## Securityboat challenge
![2022-12-05_16-55](https://user-images.githubusercontent.com/59454895/205682163-7dbc6b1b-2279-4f43-a2bc-7a702228b692.png)

The web-site doesn't have a ton of features. it is possible to insert a note and a title, then the user can view the content just inserted.
Looking at the code, it is immediately possible to identify the injection point:

![2022-12-05_17-02](https://user-images.githubusercontent.com/59454895/205683886-012d1e35-0d2c-40a5-8a0e-f6b5a9acd38a.png)

### Injection Point
in the code above ```localStorage.flag``` is passed to ```InnerHTML```, but ```localStorage``` is going to inherit from the Object.prototype. It means that if we have an client-side prototype pollution we can to control the localStorage value, here there is an example:
![2022-12-05_17-30](https://user-images.githubusercontent.com/59454895/205690366-1a23ef2c-4acd-4336-811e-019c53515d58.png)

I managed to confirm the presence of prototype pollution in this way:

![2022-12-05_19-14](https://user-images.githubusercontent.com/59454895/205711384-3b05731e-58ab-4b2d-a08e-11733a032b2b.png)

Now that the vulnerability is confirmed, the condition ```if(document.domain !== 'ctf.securityboat.in')``` must be bypassed in order to reach the injection point, to do it we can use a vulnerability called dom clobbering.

### Dom Clobbering

With Dom Clobbering is also possible to clobber `document` properties with some tags, like the `<img>` or `<form>` tag, let's make an example:


![](https://i.imgur.com/3Nn3xJp.png)

You can see how `document.URL` is returning a string with the url of the page, but after I put `<img name=URL>` the img element will be returned instaed of the string, let's make another example:

![](https://i.imgur.com/SgBmupO.png)

As you can see in this screenshot, if we have an img tag on the page with the name attribute set to “try”, “document.try” now refers to this HTML element.
As you might now have guessed, we can use this technique to override the “document.domain” variable with an HTML element and bypass the condition.

Now that this concept is clear, we can use something like `https://ctf.securityboat.in/?title=a1&body=1&subject=a%3Cform%20name=DOMAIN%3E%3C/form%3E&__proto__[flag]=%3Cimg%20src=x%20onerror=alert()%3E`. 

The first part (`<form name=DOMAIN>`) is related to the dom clobbering technique that we need to bypass the condition and reach the injection point described above, the second part of the payload is usefull to us because we can pollute `flag` with our malicious content, but the alert won't pop up because there is a CSP to bypass.

### CSP Bypass and Conclusion

These were the rules of the CSP inside a meta tag:
```html
 <meta
      http-equiv="Content-Security-Policy"
      content="default-src 'self'; style-src 'unsafe-inline' https://cdn.jsdelivr.net; font-src  https://cdn.jsdelivr.net; script-src 'self' https://cdn.jsdelivr.net; object-src 'none'"
 />
```
Here the bypass is very simple, the directive ```'script-src'``` is accepting ```jsdeliver```, therefore, inside ```jsdeliver``` there is a well known gadget that we can abuse to bypass the csp:

```html
 <script src="https://unpkg.com/csp-bypass@1.0.2-0/dist/sval-classic.js"></script>
  <br csp="alert(1)">
```
Full Payload:

```https://ctf.securityboat.in/?title=a1&body=1&subject=a<form name=DOMAIN></form>&__proto__[flag]=<iframe srcdoc="<br csp=alert(localStorage.getItem('flag'))><script src=https://cdn.jsdelivr.net/npm/csp-bypass@1.0.2/dist/sval-classic.js></script>"></iframe>```


