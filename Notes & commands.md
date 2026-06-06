[[Help]]
`id` - What groups user belongs to
`ls /` - Shows top level directories
`ls /home` - sometimes lists other users
`uname -a` or `cat /etc/os-release` - System info
`sudo -l` - shows sudo permissions
`find / -perm -4000 2>/dev/null` - SUID Binaries | Looks for binaries that allow privesc
`find / -writable -type f 2>/dev/null` - Writable files to modify for potential privesc
`find / -type f 2>/dev/null | grep -i flag` - finds files with 'flag' in name
