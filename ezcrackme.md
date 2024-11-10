# Background
![image](https://github.com/user-attachments/assets/028bc282-e819-4b17-a885-3376aa0f1e7c)\
Someone was looking at a crackme I made and said this. So I looked throughout dozens of discord servers and found a crackme that *they* made. 

# Unpacking
With DIE we can see that it uses:\
`Packer: UPX(3.95)[NRV2E_LE32,brute,Modified(53554f4d)]`\
This shows that the binary is packed with a **modified** version of upx, this means that we can't use upx to statically unpack it. When we try to unpack it we get the message:\
`upx: crackme2: NotPackedException: not packed by UPX`\
Let's try dynamic unpacking! First, I'll open the binary in a arch linux docker image and see what it calls using strace:
```
...
mprotect(0x4bd000, 12288, PROT_READ)    = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}) = 0
write(1, "Need exactly one argument.\n", 27Need exactly one argument.
) = 27
exit_group(-1)
+++ exited with 255 +++
```
Let's hook the write system call since we know the program uses that. We also know that UPX unpacks the program in full before executing it, so we can just dump the memory of the paused process.
```
h@3779c8b06506 /m/share [255]> gdb ./crackme2
GNU gdb (GDB) 15.2
...
Reading symbols from ./crackme2...
(No debugging symbols found in ./crackme2)
(gdb) catch syscall 1
Catchpoint 1 (syscall 'write' [1])
(gdb) r
Starting program: /mnt/share/crackme2

Catchpoint 1 (call to syscall write), 0x0000000000450af7 in ?? ()
(gdb)
```
Now in order to dump the memory, we have to read the map of the process:
```
h@3779c8b06506 /root> pgrep crackme
102
164
h@3779c8b06506 /root> cat /proc/164/maps
00400000-00401000 r--p 00000000 00:00 0
00401000-00495000 r-xp 00000000 00:00 0
00495000-004bc000 r--p 00000000 00:00 0
004bc000-004bd000 ---p 00000000 00:00 0
004bd000-004c0000 r--p 00000000 00:00 0
004c0000-004c4000 rw-p 00000000 00:00 0
...
```
Now, in gdb, we can dump the memory using the gdb command `dump memory newbinary.bin 0x00400000 0x004c4000`.

# Actually reverse engineering the binary
We are greated with a standard libc_start_main type of _start, making the main identification easy. The main itself seems to call strlen but the function itself is destroyed, but we can assume that it wants a string length of 6. The actual check is sub_401d05. The first arguement is the user input, the second arguement is the string "lAmBdA", and the last arguement is 0x0203020305 (finding this is a bit confusing using decompilation, but by either emulation or looking at the assembly you can confirm it). As for sub_401d05 itself:
```c
{
int32_t var_c = 0;
char var_e = 1;

do
{
    if (constent_two[var_c] + constent_one[var_c] != input[var_c])
        return 0;
    
    var_c += 1;
    
    if (!constent_one[var_c])
        break;
} while (input[var_c]);

return 1;

```
So it seems our input is just lAmBdA + 0x0302030205. We can find these values using the single python line: 
```py
print(chr(ord('l') + 2) + chr(ord('A') + 3) + chr(ord('m') + 2) + chr(ord('B') + 3) + chr(ord('d') + 5) + chr(ord('A')))
```

# Solve
```
h@185890ad2ae4 /m/share [1]> ./crackme2 nDoEiA
Yes, nDoEiA is correct!
```
