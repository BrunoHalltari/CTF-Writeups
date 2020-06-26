## Login :

The description of this challenge was : " I made a cool login page. I bet you can't get in! " and the home page of the website was presented with a login form like this :

![Capture22](https://user-images.githubusercontent.com/59454895/85869348-d8b1da80-b7cb-11ea-9df9-d9bd98fb9e41.PNG)

## Also an index.js was provided for the donwload :

```javascript
global.__rootdir = __dirname;

const express = require('express');
const bodyParser = require('body-parser');
const cookieParser = require('cookie-parser');
const path = require('path');
const db = require('better-sqlite3')('db.sqlite3');

require('dotenv').config();

const app = express();

app.use(bodyParser.json({ extended: false }));
app.use(cookieParser());

app.post('/api/flag', (req, res) => {
    const username = req.body.username;
    const password = req.body.password;
    if (typeof username !== 'string') {
        res.status(400);
        res.end();
        return;
    }
    if (typeof password !== 'string') {
        res.status(400);
        res.end();
        return;
    }

    let result;
    try {
        result = db.prepare(`SELECT * FROM users 
            WHERE username = '${username}'
            AND password = '${password}';`).get();
    } catch (error) {
        res.json({ success: false, error: "There was a problem." });
        res.end();
        return;
    }
    
    if (result) {
        res.json({ success: true, flag: process.env.FLAG });
        res.end();
        return;
    }

    res.json({ success: false, error: "Incorrect username or password." });
});

app.use(express.static(path.join(__dirname, '/public')));

app.listen(process.env.PORT || 3000);

db.prepare(`CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT,
    password TEXT);`).run();

db.prepare(`INSERT INTO 
    users (username, password)
    VALUES ('${process.env.USERNAME}', '${process.env.PASSWORD}');`).run();
```
When i had a first look at the code i understood that there was a call to api/flag  with the credentials when we try to login , without any  kind of sanitization . I tried to insert "admin' or 1=1 --  "  in the username field in order to have a query like this : ``` SELECT * FROM users 
            WHERE username = 'admin' or 1=1 -- '
            AND password = '${password}'; ``` . It worked because in this way i have been validated as the admin without any password thanks to sql comments.
            
            
            
            


![Capture](https://user-images.githubusercontent.com/59454895/85872271-cc2f8100-b7cf-11ea-9ec7-dd6dc83aa37d.PNG)

Flag: flag{0bl1g4t0ry_5ql1}
