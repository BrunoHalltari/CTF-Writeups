## JSCLEAN :

The description of this challenge was :   
"JavaScript Cleaning Service: Transform ugly JavaScript files to pretty clean JavaScript files!" 

There was also a python file provided for the download with this code :

```python
import os
import sys
import subprocess


def main(argv):
    print("Welcome To JavaScript Cleaner")
    js_name = input("Enter Js File Name To Clean: ")
    code = input("Submit valid JavaScript Code: ")

    js_name = os.path.basename(js_name) # No Directory Traversal for you

    if not ".js" in js_name:
        print("No a Js File")
        return

    with open(js_name,'w') as fin:
        fin.write(code)

    p = subprocess.run(['/usr/local/bin/node','index.js','-f',js_name],stdout=subprocess.PIPE);
    print(p.stdout.decode('utf-8'))

main(sys.argv)

```

The first thing I tried was to call the file with a random name and insert some javascript code to see what I would get.
All i got was a beautifier :

![test](https://user-images.githubusercontent.com/59454895/80551634-09d48100-89bc-11ea-8e5e-8690c86ff36f.PNG)

Looking in the python code i discovered that with the subprocess module he spawn a processes with a specific name , so i tried to call the file "index.js"  and put some  javascript, discovering that I had managed to inject it.

![Esorcista](https://user-images.githubusercontent.com/59454895/80551972-22916680-89bd-11ea-9fc4-7ae28fd5e679.PNG)

Now we have just to extract the flag , i did it injecting this code :
" const fs = require('fs');const a = fs.readFileSync('flag.txt', 'utf8');console.log(a) "

The flag is served :
![Capture](https://user-images.githubusercontent.com/59454895/80552374-5ae57480-89be-11ea-9f4d-7d087fb70550.PNG)

Flag: flag{Js_N3v3R_FuN_2_Re4d}


