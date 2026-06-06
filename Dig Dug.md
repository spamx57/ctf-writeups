[[Easy]] | [[DNS]]

This challenge gives us a DNS Server IP, URL, and a fair bit of hints on what we need to do to get the flag from the DNS server. Here is the info for my specific instance:
- URL: givemetheflag.com
- DNS IP: 10.66.144.232

If we try going to the IP on our browser, it doesn't return a webpage. If we run an nmap scan on the host, it will show us that  ports 22 and 53 are both open, but 53 is filtering results. 
`nmap -sV 10.66.144.232`

Given the clues in the lab as well as this information, we can run the following command for DNS records on the specified site:
`dig @10.66.144.232 givemetheflag.com`

Doing this will give us the following output with our flag

> ; <<>> DiG 9.18.39-0ubuntu0.24.04.5-Ubuntu <<>> @10.67.184.16 givemetheflag.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13644
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

>;; QUESTION SECTION:
;givemetheflag.com.		IN	A

>;; ANSWER SECTION:
givemetheflag.com.	0	IN	TXT	"flag{0767ccd06e79853318f25aeb08ff83e2}"

>;; Query time: 2 msec
;; SERVER: 10.67.184.16#53(10.67.184.16) (UDP)
;; WHEN: Sat Jun 06 01:10:36 UTC 2026
;; MSG SIZE  rcvd: 86

***NOTE: This box may not work correctly using the VPN. You may need to use the attackbox in order to get the proper response if the above command does not work.*** 