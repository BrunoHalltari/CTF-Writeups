## panda-facts :

The description of this challenge was " I just found a hate group targeting my favorite animal. Can you try and find their secrets? We gotta take them down! " and the home page of the website was presented with a login form like this :

![Capture](https://user-images.githubusercontent.com/59454895/85876753-79a59300-b7d6-11ea-9bba-550b1ea16fd0.PNG)

## Also an index.js was provided for the donwload :

```javascript
global.__rootdir = __dirname;

const express = require('express');
const bodyParser = require('body-parser');
const cookieParser = require('cookie-parser');
const path = require('path');
const crypto = require('crypto');

require('dotenv').config();

const INTEGRITY = '12370cc0f387730fb3f273e4d46a94e5';

const app = express();

app.use(bodyParser.json({ extended: false }));
app.use(cookieParser());

app.post('/api/login', async (req, res) => {
    if (!req.body.username || typeof req.body.username !== 'string') {
        res.status(400);
        res.end();
        return;
    }
    res.json({'token': await generateToken(req.body.username)});
    res.end;
});

app.get('/api/validate', async (req, res) => {
    if (!req.cookies.token || typeof req.cookies.token !== 'string') {
        res.json({success: false, error: 'Invalid token'});
        res.end();
        return;
    }

    const result = await decodeToken(req.cookies.token);
    if (!result) {
        res.json({success: false, error: 'Invalid token'});
        res.end();
        return;
    }

    res.json({success: true, token: result});
});

app.get('/api/flag', async (req, res) => {
    if (!req.cookies.token || typeof req.cookies.token !== 'string') {
        res.json({success: false, error: 'Invalid token'});
        res.end();
        return;
    }

    const result = await decodeToken(req.cookies.token);
    if (!result) {
        res.json({success: false, error: 'Invalid token'});
        res.end();
        return;
    }

    if (!result.member) {
        res.json({success: false, error: 'You are not a member'});
        res.end();
        return;
    }

    res.json({success: true, flag: process.env.FLAG});
});

app.use(express.static(path.join(__dirname, '/public')));

app.listen(process.env.PORT || 3000);

async function generateToken(username) {
    const algorithm = 'aes-192-cbc'; 
    const key = Buffer.from(process.env.KEY, 'hex'); 
    
    const iv = Buffer.alloc(16, 0);

    const cipher = crypto.createCipheriv(algorithm, key, iv);

    const token = `{"integrity":"${INTEGRITY}","member":0,"username":"${username}"}`

    let encrypted = '';
    encrypted += cipher.update(token, 'utf8', 'base64');
    encrypted += cipher.final('base64');
    return encrypted;
}

async function decodeToken(encrypted) {
    const algorithm = 'aes-192-cbc'; 
    const key = Buffer.from(process.env.KEY, 'hex'); 
    
    const iv = Buffer.alloc(16, 0);
    const decipher = crypto.createDecipheriv(algorithm, key, iv);

    let decrypted = '';

    try {
        decrypted += decipher.update(encrypted, 'base64', 'utf8');
        decrypted += decipher.final('utf8');
    } catch (error) {
        return false;
    }

    let res;
    try {
        res = JSON.parse(decrypted);
    } catch (error) {
        console.log(error);
        return false;
    }

    if (res.integrity !== INTEGRITY) {
        return false;
    }

    return res;
}



```
After the login we get a page like this :

![Home](https://user-images.githubusercontent.com/59454895/85878713-72cc4f80-b7d9-11ea-9a45-15a1d9a41b40.PNG)

I Tryed to click on the button but i got an error message because i wasn't a member , after looking a while i realized there was a route to api/validate , so i just went to https://panda-facts.2020.redpwnc.tf/api/validate in order to see how the token was generated :

![Capture888888](https://user-images.githubusercontent.com/59454895/85879097-0dc52980-b7da-11ea-8a84-f5bf5b54fbd5.PNG)

So i realized we had to change the member value , looking the code i saw that the token was crypted with aes-192-cbc and encoded in base64, so my first tought was to make a token with member : 1 , encrypting it and  encoding the result in base64 to inject the token in the http header but  i couldn't do it because we didn't had the key.

Watching the code again and again  realized that the token was made in this way :
```javascript  
const token = `{"integrity":"${INTEGRITY}","member":0,"username":"${username}"}` 
```
Here we can do a JSON injection because the username is not validated , so i just injected ' prova","member":1,"username":"prova ' in the usename field to get a token like this:

{"integrity":"12370cc0f387730fb3f273e4d46a94e5","member":0,"username":"prova","member":1,"username":"prova"}

After login overloading the JSON syntax i pressed the button and got the flag :

![dd](https://user-images.githubusercontent.com/59454895/85880771-e0c64600-b7dc-11ea-85be-31ed5ec0c1a9.PNG)

Flag: flag{1_c4nt_f1nd_4_g00d_p4nd4_pun}


