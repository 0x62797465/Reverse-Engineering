# Initial Analysis
When we open up the binary in binary ninja we are greeted with a static binary, that means that everything is in the binary (scanf, puts, printf, etc).
When we look at _start (the entrypoint) we are greeted with: our classic main initialization
```asm
00401ea0  f30f1efa           endbr64
00401ea4  31ed               xor     ebp, ebp  {0x0}
00401ea6  4989d1             mov     r9, rdx
00401ea9  5e                 pop     rsi {__return_addr}
00401eaa  4889e2             mov     rdx, rsp {arg_8}
00401ead  4883e4f0           and     rsp, 0xfffffffffffffff0
00401eb1  50                 push    rax {var_8}
00401eb2  54                 push    rsp {var_8} {var_10}
00401eb3  4531c0             xor     r8d, r8d  {0x0}
00401eb6  31c9               xor     ecx, ecx  {0x0}
00401eb8  48c7c72d214000     mov     rdi, main
00401ebf  67e8fb270000       call    __libc_start_main
```
But since this is static, that means the __libc_start_main could've been tampered with!
# Finding the tampering in __libc_start_main
When we scroll all the way down to
```asm
00404906  xor     esi, esi  {0x0}
00404908  xor     edi, edi  {0x0}
0040490a  mov     rdi, qword [rsp+0x8 {var_40}]
0040490f  call    ohnop
00404914  mov     rdx, r12
00404917  mov     esi, ebp
00404919  call    __libc_start_call_main
```
we see a call to ohnop. What is this? Well, it doesn't look like something standard. It is filled with a bunch of nops and subtracts 0xb from rdi:
```asm
ohnop:
00401fcc  push    rdi {var_8}
00401fcd  xor     rdi, rdi  {0x0}
00401fd0  call    _dl_debug_initialize
00401fd5  pop     rdi {var_8}
00401fd6  sub     rdi, 0xb
00401fda  nop
00401fdb  nop
00401fdc  nop
00401fdd  nop
00401fde  nop
00401fdf  retn     {__return_addr}
```
What is in rdi? Well, it's the address of main of course! So what happens when we subtract 0xb from main? Let's do the math: 0x40212d-0xb=0x402122
So let's jump to that address and see what's there:
```asm
00402122  e9ff000000         jmp     hello
{ Does not return }

00402127                       90 90 90 90 90 90                  ......

0040212d  int64_t main()

0040212d  55                 push    rbp {__saved_rbp}
0040212e  4889e5             mov     rbp, rsp {__saved_rbp}
00402131  4883ec30           sub     rsp, 0x30
00402135  64488b0425280000…  mov     rax, qword [fs:0x28]
0040213e  488945f8           mov     qword [rbp-0x8 {var_10}], rax
00402142  31c0               xor     eax, eax  {0x0}
...
```
A jump to hello?
# The real main
This is the actual main function, which is named hello:
```asm
00402226  void hello() __noreturn

00402226  55                 push    rbp {var_8}
00402227  4889e5             mov     rbp, rsp {var_8}
0040222a  488d05683e0900     lea     rax, [rel data_496099]
00402231  4889c7             mov     rdi, rax  {data_496099, "Hello world!"}
00402234  e8d72d0100         call    _IO_puts
00402239  bf00000000         mov     edi, 0x0
0040223e  e89d310000         call    exit
```
# What did we learn?
Don't trust static binaries to do what their name says that they're doing!
