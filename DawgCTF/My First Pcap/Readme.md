## My First Pcap :

 The description of this challenge was "Find the flag in the network traffic” and a .pcap file was provided.
 First of all i checked inside the file the http traffic by implementing the "http" filter on wireshark ,and doing so I found a GET request to /flag.txt.
 Reading the content of the request I found a text encoded in base 64 " RGF3Z0NURntuMWMzX3kwdV9mMHVuZF9tM30=" and when i decoded it i got this : DawgCTF{n1c3_y0u_f0und_m3}.
 
 Flag: DawgCTF{n1c3_y0u_f0und_m3}
