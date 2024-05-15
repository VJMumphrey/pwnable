# Solution

This binary has been packed with upx.
We know this because if you follow the function call tree from entry point from main to some unkown fucntion called LOAD for instance.
The LOAD function loads something from the LOAD0 segment of the binary.

We can see this here from rizin
```asm
- offset - | 0 1  2 3  4 5  6 7  8 9  A B  C D  E F| 0123456789ABCDEF
0x00400001 |454c 4602 0101 0300 0000 0000 0000 0002| ELF.............
0x00400011 |003e 0001 0000 00f0 a444 0000 0000 0040| .>.......D.....@
0x00400021 |0000 0000 0000 0000 0000 0000 0000 0000| ................
0x00400031 |0000 0040 0038 0002 0040 0000 0000 0001| ...@.8...@......
0x00400041 |0000 0005 0000 0000 0000 0000 0000 0000| ................
0x00400051 |0040 0000 0000 0000 0040 0000 0000 0004| .@.......@......
0x00400061 |ad04 0000 0000 0004 ad04 0000 0000 0000| ................
0x00400071 |0020 0000 0000 0001 0000 0006 0000 00d8| . ..............
0x00400081 |620c 0000 0000 00d8 626c 0000 0000 00d8| b.......bl......
0x00400091 |626c 0000 0000 0000 0000 0000 0000 0000| bl..............
0x004000a1 |0000 0000 0000 0000 0020 0000 0000 00fc| ......... ......
0x004000b1 |ace0 a155 5058 211c 080d 1600 0000 0021| ...UPX!........!
0x004000c1 |7c0d 0021 7c0d 0090 0100 0092 0000 0008| |..!|...........
0x004000d1 |0000 00f7 fb93 ff7f 454c 4602 0101 0300| ........ELF.....
0x004000e1 |0200 3e00 010e 5810 401f df2f ecdb 402f| ..>...X.@../..@/
0x004000f1 |7838 0c45 2638 0006 0a21
```

running this decompresses the binary
```bash
upx -d ./flag
```

runnning the binary sprints, "I will malloc() and strcpy the flag there. take it."

This means that we will probably have to read the dynamic flag in memory using a debugger.

reading the disassembly it shows a string being loaded into memory

```asm
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00496622  0200 0000 0000 5550 582e 2e2e 3f20 736f  ......UPX...? so
0x00496632  756e 6473 206c 696b 6520 6120 6465 6c69  unds like a deli
0x00496642  7665 7279 2073 6572 7669 6365 203a 2900  very service :)
```

"UPX? sounds like a delivery service :)"

This string is the flag

