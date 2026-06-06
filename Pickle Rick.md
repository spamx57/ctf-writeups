[[Easy]]  - [[Web Challenge]]


We start off being given a web server, my instance is hosted at 10.113.131.114. The page itself says we need to log on to Rick's computer to find the 3 secret ingredients, which are our flags for this challenge. 

Since we know the IP right away, we might as well run a quick nmap scan just to see what the server is all hosting. Either due to the TryHackMe vpn or other technical limitations, I was getting an "Operation not permitted" for my usual `sudo nmap -A 10.113.131.114`, so I instead opted for a TCP connect scan (`sudo nmap -sT 10.113.131.114`) which worked beautifully. This scan showed us 2 open ports:

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Going back to the page and checking its source, we will see a comment at the bottom stating:
    Note to self, remember username!
    Username: R1ckRul3s

The page source header also gives some subdirectory information, showing the `/assets` source subdirectory that may contain other clues. When traversing to 10.113.131.114/assets, we are met with a number of files to explore.  Despite this, after downloading the images/gifs and checking the strings or doing a binwalk on them, they all turn out to just be a red herring.

After attempting to enumerate some common sub-directories, /robots.txt and /login.php both ended up being exactly what we needed for the rest of the challenge. /login.php gave us a user/password page where we can fill in the username we got from the original html and the password that we get from robots.txt.

Once inside, there is a page element that copies our entries as bash commands as the sudo user, `www-data`. Using these privileges, we're able to find the flags in the below directories using 'strings' since 'cat' was disabled:
- /home/rick
- /root/
- default directory







### Failures:
Literally everything.
- Tried to enumerate every subdirectory using GoBuster w/ no luck
- Tried to binwalk, strings, and whatever else all of the files in /assets/ but they were all a red herring
- Thought the username was for ssh and we were supposed to find an ssh private key
- Couldn't find /login.php so I had to consult the writeup
- Spent ___way___  too long on trying to find a private key
- Didn't look at /robots.txt to find password


