# NativeRegMod

This is basically a native application that can modify the registry during the early boot stage.

## So what's a native application?
An excellent description can be found by Mark at Sysinternals; http://technet.microsoft.com/en-us/sysinternals/bb897447.aspx
In short it is an application you can configure to run before the Win32 subsystem is loaded, similar to autochk,exe. What this means is that we can halt the Windows boot while in native mode (NT) and do whatever we programmed our native app to do. To give an idea of roughly when this occurs during boot, it is right after the system thread has finished phase 1 (executive and kernel initialization considered complete), and the session manager (smss.exe) has been started. In fact, it is smss.exe that starts configured native applications. It does so by reading the registry key: HKLM\SYSTEM\CurrentControlSet\Control\SessionManager\BootExecute. However at this stage, no other registry hives than then SYSTEM have been loaded (and obviously that is what this application can modify), and only 2 processes are running (system and smss). Csrss comes into play later when the subsystem is loaded. For this reason, a native application can not use the Windows API (kernel32.dll etc), but must use the NT API (ntdll.dll). So it has some similarity to kernel mode coding, though the native apps are actually running in user mode, almost right after user mode has been created. But since it is compiled with subsystem=native, it will not be possible to run it like other exe's (when win32 subsystem is loaded). To speed up the testing of such an application it is therefore an advantage to compile a win32 equivalent to execute directly within Windows.

## What operations are supported?
- Modify existing Value's data or type
- Create new value
- Create new key
- Delete value
- Delete key

## How to configure OS
The application file must be located within the \Windows\System32 folder. And the relevant registry key are:
[IMG]http://i1100.photobucket.com/albums/g407/joakimschicht/BootExecute_zps70e2f5d1.png[/IMG]


The included reg file will import the correct setting, as shown in the above image.

## Configuration of application
It will search in the root of all volumes for a file name NativeRegMod.config. The config file must have 1 configuration/modification per line (new line), and all settings must be comma separated. Currently 3 reg types are supported: REG_SZ, REG_DWORD and REG_BINARY. Due to the comma as separator, any key/value name must not have comma in it. The structure of this file is:
```
NativeRegKeyPath,ValueName,RegType,Data,
NativeRegKeyPath2,ValueName2,RegType2,Data2,
```

### Some important rules to follow regarding the config:

Assumptions:
1. New line feed for each registry key.
2. Strings separated by comma. Therefore every setting must end with a comma, even the last one on each line.
3. No setting must have comma in its value.
3. The configuration file is expected to found at the root of a volume, and must be named NativeRegMod.config.
4. Registry type must be either REG_SZ, REG_DWORD or REG_BINARY.
5. Value of REG_DWORD must be specified in decimal.
6. Value of REG_BINARY must be a sequence of hexvalues without "0x" or "\x" prefix and without spaces. Hexvalues (A-F) must be in capitals (for instance A not a).
7. When deleting a key or value put "DELETE" as reg type.

### Sample configuration:
```
\Registry\Machine\SYSTEM\Setup\NewKey1,,,,
\Registry\Machine\SYSTEM\Setup\NewKey1\NewKey2,test_sz,REG_SZ,something,
\Registry\Machine\SYSTEM\Setup,test_dword,REG_DWORD,10,
\Registry\Machine\SYSTEM\Setup,test_binary,REG_BINARY,00112233445566778899AABBCCDDEEFF,
\Registry\Machine\SYSTEM\Setup\OldKey,,DELETE,,
\Registry\Machine\SYSTEM\Setup\OldKey2,OldValueName,DELETE,,
```

### Explanation per line:
1. Creating the key "NewKey1" at \Registry\Machine\SYSTEM\Setup
2. Create the key "NewKey2" under the key created in first line. Then create a value "test_sz" of type REG_SZ with the data "something".
3. Update the data of an existing REG_DWORD value with name "test_dword" with the new data of decimal 10.
4. Update the data of an existing REG_BINARY value with name "test_binary" with the new data of "00112233445566778899AABBCCDDEEFF".
5. Delete the key \Registry\Machine\SYSTEM\Setup\OldKey
6. Delete the value named "OldValueName" under the key \Registry\Machine\SYSTEM\Setup\OldKey2

## Warning
The error checking is far from perfect, and the input evaluation is limited. It is expected to be correct. It should not be regarded as a safe C implementation. However from all my tests, the worst thing that have happened, is that the application crash and Windows continue booting fine. Of course if you are modifying system critical registry parts, then chances are good that you may mess up the system. And actually, that is the kind of use the application was made for. So, ideally you would be testing with it in a virtual machine where you have snapshots to revert. 

## What can it be used for?
That's up to you to figure out. However if you are still reading and find it interesting, you likely will come up with something.

## Target OS
Should really run on any modern Windows version and architecture. Has been tested on:
XP SP2 32-bit
Windows 7 SP1 32-bit
Windows 7 SP1 64-bit

Even though there exist compiled versions for both 32 and 64-bit, the 32-bit also works on 64-bit as long as WoW64 is present (default except for standard WinPE).

## Build Instructions

1. Download and Install Windows Driver Kit 7 from https://www.microsoft.com/en-in/download/details.aspx?id=11800
2. Open the required Build Environment. For Example - Start- Programs - Windows Driver Kits - WDK 7600.16385.1 - Build Environments - Windows 7 - x86 Checked Build Environments. A command prompt should open
3. Navigate to the directory where you have checked out this code.
4. Run `build -cgw`
5. The exe file should be generated in bin/i386/


http://reboot.pro/topic/18872-nativeregmod/

