[[Easy]] | [[DNS]]

This challenge gives us a DNS Server IP, URL, and a fair bit of hints on what we need to do to get the flag from the DNS server. Here is the info for my specific instance:
URL - givemetheflag.com
DNS IP: 10.66.144.232

If we try going to the IP on our browser, it doesn't return a webpage. And just simply doing `dig @10.66.144.232 givemetheflag.com` seems to get a refused connection, saying the server cannot be reached. In which case, I next decided to run an nmap scan just to see if port 53 was even open.
`nmap -sV 10.66.144.232`

This nmap scan shows us that ports 22 and 53 are both open, but 53 is filtering results. 

> Whatever. the answer was just the first command I ran but the box doesn't work right.