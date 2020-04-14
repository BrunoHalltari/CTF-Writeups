## Where did the spaces go? :

In this challenge we just had this code :
-.--.---...--....-.--...-.-----..-...--.---..--...-.--...-...-..-......----..-...-........--.-.-..--..-...-..-......----...--..-...-..-.....-.-.---.-.----.


At first glance it may seem like a morse code , but it is not.
Morse is too variable.  Each character is a different length ,it would have to be brute-forced by hand and that's too much, also there isn't the "DAWGCTF" flag format if we decode from morse.
After that I checked some telegraph codes it turns out to be Baudot .

Baudot is a telegraph code with fixed  length representation for each char , to be more specific a fixed length of 5 bits.
A way to make sure if we are talking about Baudot is to add up all the characters and see if the sum is divisible by five , in this case the given string is 155 chars long , so it's divisible by five .


In the end I used a baudot decoder , but first we have to convert the string into binary , so i thought  " - " meant 0  and the " . " meant 1 obtaining this :
01001000111001111010011101000001101110010001100111010011101110110111111000011011101111111100101011001101110110111111000011100110111011011111010100010100001

Now that i had the binary i converted it with the baudot decoder and i got : DAWGCTFBAUD⇧0⇩T⇧1⇩SN⇧0⇩TM⇧0⇩RSE .

But it was still not the correct flag , i just removed the " ⇧ ⇩ " and the flag  was correct .

Flag: DAWGCTFBAUD0T1SN0TM0RSE
