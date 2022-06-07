# Don't panic (:
```
This is yet another of those 'crackme' puzzles, with a little bit of extra spice. The crackme is in form of bootable image.

It is absolutely urgent that you use qemu-system-x86_64 to run the bootable challenge file. Other virtual machines may not work, maybe. I really dont know if they do or not. No guarantees anyway.

The idea of this puzzle is that you'll have to find a password from the bootable image. The password is in form of GENZ{foo_bar}. The password also acts as the flag for this challenge.

Good luck, have fun, and most imporantly, don't panic (:
```
Hint 1: `Don't panic (:`  
Attachments: `bl.bin`  

We're given a file and told that it can be ran inside `qemu-system_x86-64`.

First we check what the file is:
```bash
└─$ file bl.bin 
bl.bin: DOS/MBR boot sector
```

And if there are any files embedded inside:
```bash
└─$ binwalk -e bl.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------

                                                                                   
```
Seems we'll have to do it the hard way, as there are no actual files in the image.

Running the file will reveal to us that it's being booted by `SeaBIOS`, which happens to reveal to us that we're most likely dealing with i8086 and with a 16-bit system.

Spin up the QEMU with debugger:
```bash
qemu-system-x86_64 -s -S -m 512 -hda bl.bin -serial stdio
```

Attach Radare2 and configure the settings:
```bash
r2 -D gdb -d gdb://localhost:1234
e asm.arch=x86
e asm.bits=16
```

Then some googling (or looking at the instructions) reveals to us that the entrypoint is at 0x7C00, which we then analyze:
```bash
db 0x7c00
e dbg.bpinmaps=0
dc

aaaa

# in V > v, select the function with enter, then we can generate graph view with V > V
```

Next, I mostly put breakpoints after each function to get a generic idea of what they do.
```
db 0x7c2e
db 0x7c56
db 0x7c6d
dc
```

0x7C22 - internal print to screen? (from register si)
0x7C28 - does some initialization stuff, no idea, seems irrelevant
0x7C2e - resets the screen and prints new stuff?
0x7C56 - asks for the decryption key

0x7C6d (read the decryption key):
```asm
xor ax, ax; ax = 0
int 0x16; wait for a keystroke, stored into al as ascii character
stosb byte es:[di], al; store byte into es:[di], which starts at 0x7c00 (?)
mov ah, 0xe; ah = 0xe
int 0x10; print that character?
loop 0x7c56; decrement cx (which was initially 32)
```

0x7C78 (print thank you, initialize decryption routine):
```asm
mov si, 0x7d76; "THANK YOU :)"
call 0x7cc2; print "THANK YOU :)"
mov ax, 0xb800; ax = 0xb800
mov es, ax; es = ax = 0xb800
xor bx, bx; bx = 0
mov di, 3; di = 3
```

0x7C88 (read disk):
```asm
mov ah, 2; ah = 2
mov al, byte [0x7c2d]; al = byte at 0x7c2d (= 0xf)
mov ch, 0; ch = 0
mov cl, 2; cl = 2
mov dh, 0; dh = 0
int 0x13; read 15 disk sectors, track 0, sector 0, head 0, into 0xb800 (es:bx)
jb 0x7ca6; basically try to read 3 (value of di) times from disk per documentation for int13,2
```

0x7C97 (retry if failed):
```asm
cmp al, byte [0x7c2d]; compare al with 15 (= did we read 15 sectors)
jl 0x7caf; jump if we didn't read 15 sectors (=> retry reading again)
```

0x7C9d:
```asm
ljmp 0xb800
```

0xB8000:
```
jmp 0x800f; jump to the copied sector
```

Based on this, we know that the 32 character decryption key we enter (which also is the flag) is copied to the stack due to sp being set to 0x7C00. Though I'm not sure what we can do with this, as r2 seems to be missing bunch of instructions where something actually happens.

Putting breakpoints right after printing the thank you message (0x7C7E) will start replacing the message on the screen, though as r2 doesn't allow stepping out due to not understanding the instructions (or something - I don't reverse engineer at all :v) we're unable to figure out what causes that behavior.

After playing multiple hours with just trying to get r2 or IDA Freeware (apparently I had to download an older version to be able to use 8086 arch), I had also found a tool named `ndisasm` which produced pretty good looking results but never looked that close into it as it seemed to match r2 output.

Almost when I was about to give up, I was going everything checking if I missed something obvious, and noticed that ndisasm output has a lot of cmp instructions with individual characters towards the end which I didn't catch in r2 output, and conveniently enough they look like the flag:
```asm
0000037F  58                pop ax              ; retrieve character from stack
00000380  86C4              xchg al,ah          ; swap al and ah
00000382  80FC6C            cmp ah,0x6c         ; l
00000385  0F854301          jnz near 0x4cc      ; fail
00000389  3C7D              cmp al,0x7d         ; }
0000038B  7404              jz 0x391            ; if matches, continue to 0x391
0000038D  83C408            add sp,byte +0x8

00000390  005886            add [bx+si-0x7a],bl ; bork'd parsing?
00000393  C480FC6F          les ax,[bx+si+0x6ffc] ; cmp al,0x6f, o

00000397  0F853101          jnz near 0x4cc      ; fail
0000039B  3C6F              cmp al,0x6f         ; o
0000039D  7403              jz 0x3a2            ; if matches, continue to 0x3A2
0000039F  E92A01            jmp 0x4cc           ; fail
000003A2  58                pop ax              ; retrieve character from stack
000003A3  86C4              xchg al,ah          ; swap al and ah
000003A5  80FC5F            cmp ah,0x5f         ; _
000003A8  0F852001          jnz near 0x4cc      ; fail
000003AC  3C63              cmp al,0x63         ; c
000003AE  7402              jz 0x3b2            ; if matches, continue to 0x3b2
000003B0  EBCD              jmp short 0x37f
000003B2  58                pop ax              ; retrieve character from stack
000003B3  86C4              xchg al,ah          ; swap al and ah
000003B5  FEC4              inc ah              ; increase ah by one
000003B7  30E0              xor al,ah           ; al ^ ah
000003B9  80FC70            cmp ah,0x70         ; o
000003BC  7403              jz 0x3c1            ; if matches, continue to 0x3C1
000003BE  E90B01            jmp 0x4cc           ; fail
000003C1  3C1F              cmp al,0x1f         ; o
000003C3  7407              jz 0x3cc            ; if matches, continue to 0x3CC
000003C5  BA8306            mov dx,0x683
000003C8  E91E01            jmp 0x4e9
000003CB  F058              lock pop ax         ; retrieve character from stack
000003CD  86C4              xchg al,ah          ; swap al and ah
000003CF  80FC5F            cmp ah,0x5f         ; _
000003D2  0F85F600          jnz near 0x4cc      ; fail
000003D6  3C74              cmp al,0x74         ; t
000003D8  7403              jz 0x3dd            ; if matches, continue to 0x3DD
000003DA  E9EF00            jmp 0x4cc           ; fail
000003DD  58                pop ax              ; retrieve character from stack
000003DE  86C4              xchg al,ah          ; swap al and ah
000003E0  80FC6E            cmp ah,0x6e         ; n
000003E3  0F85E500          jnz near 0x4cc      ; fail
000003E7  3C74              cmp al,0x74         ; t
000003E9  7403              jz 0x3ee            ; if matches, continue to 0x3EE
000003EB  E9DE00            jmp 0x4cc           ; fail
000003EE  58                pop ax              ; retrieve character from stack
000003EF  86C4              xchg al,ah          ; swap al and ah
000003F1  80FC69            cmp ah,0x69         ; i
000003F4  0F85D400          jnz near 0x4cc      ; fail
000003F8  3C73              cmp al,0x73         ; s
000003FA  7403              jz 0x3ff            ; if matches, continue to 0x3FF
000003FC  E9CD00            jmp 0x4cc           ; fail
000003FF  58                pop ax              ; retrieve character from stack
00000400  86C4              xchg al,ah          ; swap al and ah
00000402  3D5F6D            cmp ax,0x6d5f       ; m_
00000405  7403              jz 0x40a            ; if matches, continue to 0x40A
00000407  E9C200            jmp 0x4cc           ; fail
0000040A  58                pop ax              ; retrieve character from stack
0000040B  86C4              xchg al,ah          ; swap al and ah
0000040D  80FC72            cmp ah,0x72         ; r
00000410  0F85B800          jnz near 0x4cc      ; fail
00000414  3C61              cmp al,0x61         ; a
00000416  7403              jz 0x41b            ; if matches, continue to 0x41B
00000418  E9B100            jmp 0x4cc
0000041B  58                pop ax              ; retrieve character from stack
0000041C  86C4              xchg al,ah          ; swap al and ah
0000041E  80FC73            cmp ah,0x73         ; s
00000421  0F85A700          jnz near 0x4cc      ; fail
00000425  3C5F              cmp al,0x5f         ; _
00000427  7403              jz 0x42c            ; if matches, continue to 0x42C
00000429  E9A000            jmp 0x4cc
0000042C  58                pop ax              ; retrieve character from stack
0000042D  86C4              xchg al,ah          ; swap al and ah
0000042F  80FC5F            cmp ah,0x5f         ; _
00000432  0F859600          jnz near 0x4cc      ; fail
00000436  3C61              cmp al,0x61         ; a
00000438  7406              jz 0x440            ; if matches, continue to 0x440
0000043A  BA8306            mov dx,0x683
0000043D  E9A900            jmp 0x4e9
00000440  58                pop ax              ; retrieve character from stack
00000441  86C4              xchg al,ah          ; swap al and ah
00000443  80FC68            cmp ah,0x68         ; h
00000446  0F858200          jnz near 0x4cc      ; fail
0000044A  3C65              cmp al,0x65         ; e
0000044C  7402              jz 0x450            ; if matches, continue to 0x450
0000044E  EB7C              jmp short 0x4cc     ; fail
00000450  58                pop ax              ; retrieve character from stack
00000451  86C4              xchg al,ah          ; swap al and ah
00000453  80FC61            cmp ah,0x61         ; a
00000456  7574              jnz 0x4cc           ; fail
00000458  3C63              cmp al,0x63         ; c
0000045A  7402              jz 0x45e            ; if matches, continue to 0x45E
0000045C  EB6E              jmp short 0x4cc
0000045E  58                pop ax              ; retrieve character from stack
0000045F  86C4              xchg al,ah          ; swap al and ah
00000461  80FC7B            cmp ah,0x7b         ; {
00000464  7566              jnz 0x4cc           ; fail
00000466  3C63              cmp al,0x63         ; c
00000468  7402              jz 0x46c            ; if matches, continue to 0x46C
0000046A  EB60              jmp short 0x4cc     ; fail
0000046C  58                pop ax              ; retrieve character from stack
0000046D  86C4              xchg al,ah          ; swap al and ah
0000046F  FEC8              dec al              ; decrease al
00000471  3C59              cmp al,0x59         ; Z
00000473  7402              jz 0x477            ; if matches, continue to 0x477
00000475  EB55              jmp short 0x4cc     ; fail
00000477  80F411            xor ah,0x11         ; ah ^ 0x11
0000047A  80FC5F            cmp ah,0x5f         ; N
0000047D  7402              jz 0x481            ; if matches, continue to 0x481
0000047F  EB4B              jmp short 0x4cc     ; fail
00000481  58                pop ax              ; retrieve character from stack
00000482  86C4              xchg al,ah          ; swap al, ah
00000484  80FC47            cmp ah,0x47         ; G
00000487  7543              jnz 0x4cc           ; fail
00000489  3C45              cmp al,0x45         ; E
0000048B  744F              jz 0x4dc            ; matches
0000048D  EB3D              jmp short 0x4cc     ; fail
0000048F  B800B0            mov ax,0xb000       ; jmp dst, ax = 0xb000
00000492  8EC0              mov es,ax           ; es = ax = 0xb000
00000494  BF0008            mov di,0x800        ; di = 0x800
00000497  BA1027            mov dx,0x2710       ; dx = 0x2710
0000049A  B80000            mov ax,0x0          ; ax = 0x0
0000049D  F366AB            rep stosd           ; repeat
000004A0  C3                ret                 ; return
000004A1  B80000            mov ax,0x0          ; jmp dst, ax = 0
000004A4  8EC0              mov es,ax           ; es = ax = 0
000004A6  BF3605            mov di,0x536        ; di = 0x536
000004A9  B91001            mov cx,0x110        ; cx = 0x110
000004AC  F3AA              rep stosb           ; repeat...?
000004AE  C3                ret                 ; return
000004AF  FC                cld                 ; clear DF
000004B0  31C0              xor ax,ax           ; ax = 0
000004B2  8ED8              mov ds,ax           ; ds = ax = 0
000004B4  66BA00800B00      mov edx,0xb8000     ; edx = 0xb8000
000004BA  B40F              mov ah,0xf          ; ah = 0xf
000004BC  AC                lodsb               ; load string
000004BD  84C0              test al,al          ; check if al is 1
000004BF  7409              jz 0x4ca            ; if not, jump (=> exit)
000004C1  678902            mov [edx],ax        ; edx = ax = 0
000004C4  6683C202          add edx,byte +0x2   ; edx += 0x2
000004C8  EBF2              jmp short 0x4bc     ; jump
000004CA  F4                hlt                 ; jmp dst, exit
000004CB  C3                ret                 ; return
000004CC  E8D2FF            call 0x4a1          ; jmp dst
000004CF  E8BDFF            call 0x48f
000004D2  BED006            mov si,0x6d0        ; si = 0x6d0
000004D5  E8D7FF            call 0x4af
000004D8  FA                cli                 ; exit
000004D9  F4                hlt                 ; exit
000004DA  EBFC              jmp short 0x4d8
000004DC  E8C2FF            call 0x4a1          ; jmp dst
000004DF  BEC406            mov si,0x6c4        ; si = 0x6c4
000004E2  E8CAFF            call 0x4af
000004E5  FA                cli                 ; exit
000004E6  F4                hlt                 ; exit
```

Going through all of the relevant looking instructions by hand and commenting them out reveals us the following characters in order:
```
0x37F # l}
0x391 # oo
0x3A2 # _c
0x3B2 # oo
0x3CC # _t
0x3DD # nt
0x3EE # is
0x3FF # m_
0x40A # ra
0x41B # s_
0x42C # _a
0x440 # he
0x450 # ac
0x45E # {c
0x46C # NZ
0x481 # GE
```

And as the stack is popped from the end first, we just concat each characters from bottom to top, giving us the flag `GENZ{cache_as_ram_isnt_too_cool}`.