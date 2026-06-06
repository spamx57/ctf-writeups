[[Easy]]

## Task 1
**48bb6e862e54f2a795ffc4e541caed4d** - md5 hash
- https://md5decrypt.net/en/ > `easy`

**CBFDAC6008F9CAB4083784CBD1874F76618D2A97** - SHA1
- https://md5decrypt.net/en/Sha1/ > `password123`

**1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032** - SHA256
- https://md5decrypt.net/en/Sha256/ > `letmein`

**$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom** - blowfish
- Blowfish tends to be a much more resource intensive hash to crack, so we're going to have to do this locally with hashcat. First, if we look at the hint they suggest limiting to 4 letter words and utilizing rockyou. 
- First we'll filter rockyou and extract only the 4 character words into a new document:
		`grep '^....$' /usr/share/wordlists/rockyou.txt > rockmini.txt`
- Now that we've limited our potential answers, we can then use hashcat utilizing this 4 char wordlist. 
		`hashcat -m 3200 hash.txt rockmini.txt`
			*The -m specifies 3200 which is the code for blowfish, hash.txt is our hash in a text file, and obviously rockmini.txt is the file we created.*
- This will then give us our result:
>$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom:bleh

**279412f945939ba78ce0758d3fd83daa** - MD4
https://md5decrypt.net/en/Md4/ > `Eternity22`

## Task 2
**F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85** - SHA256
https://md5decrypt.net/en/Sha256/ > `paule`

**1DFECA0C002AE40B8619ECF94819CC1B** - NTLM
https://md5decrypt.net/en/Ntlm/ > `n63umy8lkf4i`
**`$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`** - Salted SHA-512
- For this one we will follow the same process as before since we can see it's 6 chars long:
		`grep '^......$' /usr/share/wordlists/rockyou.txt > rockyou6.txt`
		
		hashcat -m 1800 hash2.txt rockyou6.txt`
- After running for about 6 minutes, it will eventually give us our result:
>`$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.:waka99`

**e5d8870e5bdd26602cab8dbe07a942c8669e56d6** - HMAC-SHA1
- Luckily this one isn't nearly as lengthy since we have the key for it, so we'll skip filtering rockyou this time and just go right for the hashcat command
- `hashcat -m 160 -a 0 e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme /usr/share/wordlists/rockyou.txt`
		*This specifies hmac-sha1, -a 0 is a dictionary attack, the hash and it's key are next, then obviously the normal rockyou wordlist.*
> e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme:481616481616


And boom, there's our completed room