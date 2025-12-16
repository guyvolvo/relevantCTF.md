**So here are the rules and hints we get from the challenge:**

- Any tools or techniques are permitted in this engagement, however we ask that you attempt manual exploitation first
- Locate and note all vulnerabilities found
- Submit the flags discovered to the dashboard
- Only the IP address assigned to your machine is in scope
- Find and report ALL vulnerabilities (yes, there is more than one path to root)
- Note - Nothing in this room requires Metasploit
- Writeups will not be accepted for this room.

Alright lets start

First we have to connect to the vpn, 

```bash
2025-12-16 06:21:58 Initialization Sequence Completed
2025-12-16 06:21:58 Data Channel: cipher 'AES-256-GCM', peer-id: 288
2025-12-16 06:21:58 Timers: ping 10, ping-restart 120
2025-12-16 06:21:58 Protocol options: explicit-exit-notify 1, protocol-flags cc-exit tls-ekm dyn-tls-crypt
```

I'll try scanning the machine for any services that could be running 

```sh
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -T4 10.64.160.63
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-16 07:16 EST
Nmap scan report for 10.64.160.63
Host is up (0.15s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds (workgroup: WORKGROUP)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2025-12-16T12:16:50+00:00
|_ssl-date: 2025-12-16T12:17:30+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2025-12-15T12:08:06
|_Not valid after:  2026-06-16T12:08:06
Service Info: Host: RELEVANT; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-12-16T12:16:54
|_  start_date: 2025-12-16T12:08:06
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h36m00s, deviation: 3h34m41s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-12-16T04:16:53-08:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.68 seconds
```
So the open services on the machine are HTTP, RPC, SMB , and RDP
First ill have to try smb enumeration,

```sh
┌──(kali㉿kali)-[~]
└─$ smbclient -L //10.64.160.63 -N       

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.64.160.63 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

┌──(kali㉿kali)-[~]
└─$ smbclient //10.64.160.63/ADMIN$ -N
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                          
┌──(kali㉿kali)-[~]
└─$ smbclient //10.64.160.63/C$ -N    
tree connect failed: NT_STATUS_ACCESS_DENIED
                                                                                          
┌──(kali㉿kali)-[~]
└─$ smbclient //10.64.160.63/IPC$ -N  
Try "help" to get a list of possible commands.
smb: \> shares
shares: command not found
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
mkfifo         more           mput           newer          notify         
open           posix          posix_encrypt  posix_open     posix_mkdir    
posix_rmdir    posix_unlink   posix_whoami   print          prompt         
put            pwd            q              queue          quit           
readlink       rd             recurse        reget          rename         
reput          rm             rmdir          showacls       setea          
setmode        scopy          stat           symlink        tar            
tarmode        timeout        translate      unlock         volume         
vuid           wdel           logon          listconnect    showconnect    
tcon           tdis           tid            utimes         logoff         
..             !              
smb: \> 
```

It appears we established a connection with the IPC$ which is not very useful lets try another way

```sh
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                     https://github.com/ShawnDEvans/smbmap

[\] Checking for open ports...                                                            [|] Checking for open ports...                                                            [/] Checking for open ports...                                                            [-] Checking for open ports...                                                            [*] Detected 1 hosts serving SMB        
[\] Authenticating...                                                                     [|] Authenticating...                                                                     [/] Authenticating...                                                                     [-] Authenticating...                                                                     [\] Authenticating...                                                                     [|] Authenticating...                                                                     [/] Authenticating...                                                                     [-] Authenticating...                                                                     [\] Authenticating...                                                                     [|] Authenticating...                                                                     [/] Authenticating...                                                                     [-] Authenticating...                                                                     [\] Authenticating...                                                                     [|] Authenticating...                                                                     [/] Authenticating...                                                                     [*] Established 0 SMB connections(s) and 0 authenticated session(s)
[-] Closing connections..                                                                 [\] Closing connections..                                                                 [|] Closing connections..                                                                 [/] Closing connections..                                                                 [-] Closing connections..                                                                                                                                                           [*] Closed 0 connections
                                                                                          
┌──(kali㉿kali)-[~]
└─$ enum4linux -a 10.64.160.63
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Dec 16 07:27:37 2025

 =========================================( Target Information )=========================================                                                                           
                                                                                          
Target ........... 10.64.160.63                                                           
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ============================( Enumerating Workgroup/Domain on 10.64.160.63 )============================                                                                           
                                                                                          
                                                                                          
[E] Can't find workgroup/domain                                                           
                                                                                          
                                                                                          

 ================================( Nbtstat Information for 10.64.160.63 )================================                                                                           
                                                                                          
Looking up status of 10.64.160.63                                                         
No reply from 10.64.160.63

 ===================================( Session Check on 10.64.160.63 )===================================                                                                            
                                                                                          
                                                                                          
[E] Server doesn't allow session using username '', password ''.  Aborting remainder of tests.                                                                                      
                                                                                          
                                                                                          
┌──(kali㉿kali)-[~]
└─$ rpcclient -U "" -N 10.64.160.63   
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED
```
lets try seeing what we get as user 'guest' with crackmapexec

```sh
┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.64.160.63 -u 'guest' -p ''

SMB         10.64.160.63    445    RELEVANT         [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB         10.64.160.63    445    RELEVANT         [+] Relevant\guest: 
                                                                                                            
┌──(kali㉿kali)-[~]
└─$ smbclient -L //┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.64.160.63 -u '' -p ''

[*] First time use detected
[*] Creating home directory structure
[*] Creating default workspace
[*] Initializing SMB protocol database
[*] Initializing FTP protocol database
[*] Initializing RDP protocol database
[*] Initializing SSH protocol database
[*] Initializing WINRM protocol database
[*] Initializing MSSQL protocol database
[*] Initializing LDAP protocol database
[*] Copying default configuration file
[*] Generating SSL certificate
SMB         10.64.160.63    445    RELEVANT         [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB         10.64.160.63    445    RELEVANT         [-] Relevant\: STATUS_ACCESS_DENIED 
                                                                                          
┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.64.160.63 -u 'guest' -p ''

SMB         10.64.160.63    445    RELEVANT         [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB         10.64.160.63    445    RELEVANT         [+] Relevant\guest: 

                                                                                                            
┌──(kali㉿kali)-[~]
└─$ smbclient -L //10.64.160.63 -U guest%        

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.64.160.63 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
                                                                                                            
┌──(kali㉿kali)-[~]
└─$ smbmap -H 10.64.160.63 -u guest      

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                     https://github.com/ShawnDEvans/smbmap

[\] Checking for open ports...                                                                              [|] Checking for open ports...                                                                              [/] Checking for open ports...                                                                              [-] Checking for open ports...                                                                              [*] Detected 1 hosts serving SMB
[\] Authenticating...                                                                                       [|] Authenticating...                                                                                       [/] Authenticating...                                                                                       [-] Authenticating...                                                                                       [\] Authenticating...                                                                                       [|] Authenticating...                                                                                       [/] Authenticating...                                                                                       [-] Authenticating...                                                                                       [\] Authenticating...                                                                                       [|] Authenticating...                                                                                       [/] Authenticating...                                                                                       [-] Authenticating...                                                                                       [\] Authenticating...                                                                                       [|] Authenticating...                                                                                       [/] Authenticating...                                                                                       [*] Established 1 SMB connections(s) and 1 authenticated session(s)
[-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                   [\] Enumerating shares...                                                                                   [|] Enumerating shares...                                                                                   [/] Enumerating shares...                                                                                   [-] Enumerating shares...                                                                                                                                                                                       
[+] IP: 10.64.160.63:445        Name: 10.64.160.63              Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        nt4wrksv                                                READ, WRITE
[\] Closing connections..                                                                                   [|] Closing connections..                                                                                   [/] Closing connections..                                                                                   [-] Closing connections..                                                                                   [\] Closing connections..                                                                                   [|] Closing connections..                                                                                   [/] Closing connections..                                                                                   [-] Closing connections..                                                                                   [*] Closed 1 connections
```

Jackpot we see that the user guest doesnt require a password and that we can read and write to share nt4wrksv

```sh
┌──(kali㉿kali)-[~]
└─$ smbclient  //10.64.160.63/nt4wrksv -U guest% 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Dec 16 07:30:34 2025
  ..                                  D        0  Tue Dec 16 07:30:34 2025
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020

                7735807 blocks of size 4096. 5101049 blocks available
smb: \> get passwords.txt
getting file \passwords.txt of size 98 as passwords.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
```

Nice we got password.txt lets look inside now
