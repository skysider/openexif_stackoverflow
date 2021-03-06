# openexif_stackoverflow

### Product
openexif

### Version
2.1.4

### Vulnerability
Stack buffer overflow 

### Description

The result of gdb is as follows:

```
#gdb  /work/openexif-2_1_4-src/examples/ExifTagDump/ExifTagDump

(gdb) run ../out/crashes/id:000007,sig:11,src:000000,op:havoc,rep:64
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /work/openexif-2_1_4-src/examples/ExifTagDump/ExifTagDump ../out/crashes/id:000007,sig:11,src:000000,op:havoc,rep:64

Program received signal SIGSEGV, Segmentation fault.
0x000000000041d735 in ExifImageFile::readDQT (this=<optimized out>, length=<optimized out>) at ExifImageFileRead.cpp:286
286                         mJpegTables->Q[tableNum]->quantizer[openexif_jpeg_natural_order[i]] = qt[i];
(gdb) bt
#0  0x000000000041d735 in ExifImageFile::readDQT (this=<optimized out>, length=<optimized out>) at ExifImageFileRead.cpp:286
#1  0x000000000041b4e2 in ExifImageFile::readImage (this=0x7fffffffe378) at ExifImageFileRead.cpp:125
#2  0x0000000000410190 in ExifImageFile::initAfterOpen (this=0x7fffffffe378, cmode=0x4eff4c "r") at ExifImageFile.cpp:435
#3  0x0000000000429cd7 in ExifOpenFile::open (this=0x7fffffffe378, filename=0x7fffffffe8ab "../out/crashes/id:000007,sig:11,src:000000,op:havoc,rep:64", cmode=0x4eff4c "r") at ExifOpenFile.cpp:78
#4  0x0000000000402514 in main (argc=<optimized out>, argv=<optimized out>) at ExifTagDump.cpp:64
```

The result of running the program directly.
```
/work/openexif-asan/examples/ExifTagDump/ExifTagDump id:000007,sig:11,src:000000,op:havoc,rep:64
==52414==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffd09fbca04 at pc 0x000000545852 bp 0x7ffd09fbc9d0 sp 0x7ffd09fbc9c8
READ of size 1 at 0x7ffd09fbca04 thread T0                         
    #0 0x545851  (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x545851)
    #1 0x526014  (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x526014)
    #2 0x567caa  (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x567caa)
    #3 0x511369  (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x511369)
    #4 0x7f44ec56a82f  (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #5 0x41b158  (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x41b158)

Address 0x7ffd09fbca04 is located in stack of thread T0 at offset 36 in frame
    #0 0x54415f  (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x54415f)

  This frame has 1 object(s):
    [32, 36) 'input' <== Memory access at offset 36 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow (/work/openexif-asan/examples/ExifTagDump/ExifTagDump+0x545851)
Shadow bytes around the buggy address:
  0x1000213ef8f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000213ef900: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000213ef910: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000213ef920: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000213ef930: 00 00 00 00 00 00 00 00 00 00 00 00 f1 f1 f1 f1
=>0x1000213ef940:[04]f3 f3 f3 00 00 00 00 00 00 00 00 00 00 00 00
0x1000213ef950: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00                                                                                                                                                                                                     [1/1937]
  0x1000213ef960: f1 f1 f1 f1 04 f2 00 00 04 f3 f3 f3 f3 f3 f3 f3
  0x1000213ef970: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x1000213ef980: 00 00 00 00 f1 f1 f1 f1 04 f3 f3 f3 00 00 00 00
  0x1000213ef990: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb

```

The poc is attached [here](crash.jpg).

