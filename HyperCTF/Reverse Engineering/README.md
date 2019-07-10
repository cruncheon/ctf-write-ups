# Reverse Engineering
Reverse Engineering was one of the Challenges in the HyperCTF game A Flock Of Joeys participated in.

The hint was `Find the flag in the compiled C file` and it included an attached file.

## Initial investigation
This was my first attempt at anything like this. So I started with the first two tools that popped into my head. `ldd` and `binwalk`.

```
$ ldd CompiledFile 
	not a dynamic executable
```

```
$ binwalk CompiledFile 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
1284          0x504           Unix path: /usr/lib/dyld
1424          0x590           Unix path: /usr/lib/libSystem.B.dylib

```

Hmm. Nothing too useful there. Certainly nothing that looks like a flag.

Another Joey gave `strings` a go.

```
$ strings CompiledFile 
__PAGEZERO
__TEXT
__text
__TEXT
__stubs
__TEXT
__stub_helper
__TEXT
__cstring
__TEXT
__unwind_info
__TEXT
__DATA
__nl_symbol_ptr
__DATA
__got
__DATA
__la_symbol_ptr
__DATA
__data
__DATA
__common
__DATA
__LINKEDIT
/usr/lib/dyld
/usr/lib/libSystem.B.dylib
HaHaHa You Failed
@___stack_chk_guard
@dyld_stub_binder
@___stack_chk_fail
@_printf
_mh_execute_header
?main
CputInto
RencryptionKey
__mh_execute_header
_encryptionKey
_main
_putInto
___stack_chk_fail
___stack_chk_guard
_printf
dyld_stub_binder
```

You'll notice in the middle of all that there's `HaHaHa You Failed`. Apparently that's not the way to go.

At the same time I was looking at this in a hex editor and saw the same thing.

```
00000F90  FF FF 68 18  00 00 00 E9   DC FF FF FF  48 61 48 61   ..h.........HaHa
00000FA0  48 61 20 59  6F 75 20 46   61 69 6C 65  64 0A 00 00   Ha You Failed...
```

I tried searching for `flag` in the hex editor, but no dice.

Having exhausted my good ideas, and not ready to ask the Internet, I spun up a new VM (can't be too paranoid), disconnected it from the network, and attempted to run the file.

```
$ chmod +x CompiledFile 
$ ./CompiledFile 
bash: ./CompiledFile: cannot execute binary file: Exec format error
```

Huh. In hindsight it didn't look like an ELF either.... Time to get some outside advice. And by advice I mean, DuckDuckGo.

https://askubuntu.com/questions/648555/bash-program-cannot-execute-binary-file-exec-format-error

Ah, so we can use `file` to identify it. And there's a hint in this answer but we'll get there.

```
$ file CompiledFile 
CompiledFile: Mach-O 64-bit x86_64 executable, flags:<NOUNDEFS|DYLDLINK|TWOLEVEL|PIE>
```

Mach-O? What's that?

https://en.wikipedia.org/wiki/Mach-O

Ah. So this is likely an OSX executable. Well, I have nothing to run that on, so lets dig deeper.

http://blog.vinceliu.com/2009/06/examining-binary-files-in-linux.html

Many of these tools didn't yield anything, as they were expecting x86/amd64 files (anything with a `-m i8086` parameter, I just dropped that param)

`readelf` sounded promising ("To disassembled a compiled binary"), but oh yeah, not an ELF. Pretty much all of those tools were expecting something I could run on my system, and this was not that. So what else can we try? Back to DDG!

## Hopper

https://www.hopperapp.com/

"The macOS and Linux Disassember"

Well that sounds promising! And while not free, it comes with a free trial (with some serious limitations, but they'll do for our purposes). There's even a `.deb`, excellent. (And it runs just fine on Kali)

When launching Hopper, it will ask for a license file. Just click the "Try the Demo" button.

`File -> Read Executable to Disassemble`

When the options appeared, I went with the defaults.

At this point I got a window with a bunch of machine code and assembly. If your program is really, really simple, assembly isn't the worst thing to try and understand. Especially if you only need to get part of it. So lets poke through this thing and see what pops out.

Huh. There's a section here where they add a series of values... See how there's a `mov` over and over, something to do with `var_30` every time, but a different value each time. But it's assembly and that's kind of a pain. I lied about it being easy. I wonder if there's a better representation somewhere. Keep digging.

```
0000000100000d60         push       rbp
0000000100000d61         mov        rbp, rsp
0000000100000d64         sub        rsp, 0x40
0000000100000d68         mov        rax, qword [___stack_chk_guard_100001010]   ; ___stack_chk_guard_100001010
0000000100000d6f         mov        rax, qword [rax]
0000000100000d72         mov        qword [rbp+var_8], rax
0000000100000d76         mov        dword [rbp+var_34], 0x0
0000000100000d7d         mov        ecx, dword [rbp+var_34]
0000000100000d80         mov        edx, ecx
0000000100000d82         add        edx, 0x1
0000000100000d85         mov        dword [rbp+var_34], edx
0000000100000d88         movsxd     rax, ecx
0000000100000d8b         mov        byte [rbp+rax+var_30], 0x66
0000000100000d90         mov        ecx, dword [rbp+var_34]
0000000100000d93         mov        edx, ecx
0000000100000d95         add        edx, 0x1
0000000100000d98         mov        dword [rbp+var_34], edx
0000000100000d9b         movsxd     rax, ecx
0000000100000d9e         mov        byte [rbp+rax+var_30], 0x6c
0000000100000da3         mov        ecx, dword [rbp+var_34]
0000000100000da6         mov        edx, ecx
0000000100000da8         add        edx, 0x1
0000000100000dab         mov        dword [rbp+var_34], edx
0000000100000dae         movsxd     rax, ecx
0000000100000db1         mov        byte [rbp+rax+var_30], 0x61
0000000100000db6         mov        ecx, dword [rbp+var_34]
0000000100000db9         mov        edx, ecx
0000000100000dbb         add        edx, 0x1
0000000100000dbe         mov        dword [rbp+var_34], edx
0000000100000dc1         movsxd     rax, ecx
0000000100000dc4         mov        byte [rbp+rax+var_30], 0x67
0000000100000dc9         mov        ecx, dword [rbp+var_34]
0000000100000dcc         mov        edx, ecx
0000000100000dce         add        edx, 0x1
0000000100000dd1         mov        dword [rbp+var_34], edx
0000000100000dd4         movsxd     rax, ecx
0000000100000dd7         mov        byte [rbp+rax+var_30], 0x7b
0000000100000ddc         mov        ecx, dword [rbp+var_34]
0000000100000ddf         mov        edx, ecx
0000000100000de1         add        edx, 0x1
0000000100000de4         mov        dword [rbp+var_34], edx
0000000100000de7         movsxd     rax, ecx
0000000100000dea         mov        byte [rbp+rax+var_30], 0x72
0000000100000def         mov        ecx, dword [rbp+var_34]
0000000100000df2         mov        edx, ecx
0000000100000df4         add        edx, 0x1
0000000100000df7         mov        dword [rbp+var_34], edx
0000000100000dfa         movsxd     rax, ecx
0000000100000dfd         mov        byte [rbp+rax+var_30], 0x30
0000000100000e02         mov        ecx, dword [rbp+var_34]
0000000100000e05         mov        edx, ecx
0000000100000e07         add        edx, 0x1
0000000100000e0a         mov        dword [rbp+var_34], edx
0000000100000e0d         movsxd     rax, ecx
0000000100000e10         mov        byte [rbp+rax+var_30], 0x62
0000000100000e15         mov        ecx, dword [rbp+var_34]
0000000100000e18         mov        edx, ecx
0000000100000e1a         add        edx, 0x1
0000000100000e1d         mov        dword [rbp+var_34], edx
0000000100000e20         movsxd     rax, ecx
0000000100000e23         mov        byte [rbp+rax+var_30], 0x34
0000000100000e28         mov        ecx, dword [rbp+var_34]
0000000100000e2b         mov        edx, ecx
0000000100000e2d         add        edx, 0x1
0000000100000e30         mov        dword [rbp+var_34], edx
0000000100000e33         movsxd     rax, ecx
0000000100000e36         mov        byte [rbp+rax+var_30], 0x32
0000000100000e3b         mov        ecx, dword [rbp+var_34]
0000000100000e3e         mov        edx, ecx
0000000100000e40         add        edx, 0x1
0000000100000e43         mov        dword [rbp+var_34], edx
0000000100000e46         movsxd     rax, ecx
0000000100000e49         mov        byte [rbp+rax+var_30], 0x29
0000000100000e4e         mov        ecx, dword [rbp+var_34]
0000000100000e51         mov        edx, ecx
0000000100000e53         add        edx, 0x1
0000000100000e56         mov        dword [rbp+var_34], edx
0000000100000e59         movsxd     rax, ecx
0000000100000e5c         mov        byte [rbp+rax+var_30], 0x33
0000000100000e61         mov        ecx, dword [rbp+var_34]
0000000100000e64         mov        edx, ecx
0000000100000e66         add        edx, 0x1
0000000100000e69         mov        dword [rbp+var_34], edx
0000000100000e6c         movsxd     rax, ecx
0000000100000e6f         mov        byte [rbp+rax+var_30], 0x23
0000000100000e74         mov        ecx, dword [rbp+var_34]
0000000100000e77         mov        edx, ecx
0000000100000e79         add        edx, 0x1
0000000100000e7c         mov        dword [rbp+var_34], edx
0000000100000e7f         movsxd     rax, ecx
0000000100000e82         mov        byte [rbp+rax+var_30], 0x66
0000000100000e87         mov        ecx, dword [rbp+var_34]
0000000100000e8a         mov        edx, ecx
0000000100000e8c         add        edx, 0x1
0000000100000e8f         mov        dword [rbp+var_34], edx
0000000100000e92         movsxd     rax, ecx
0000000100000e95         mov        byte [rbp+rax+var_30], 0x31
0000000100000e9a         mov        ecx, dword [rbp+var_34]
0000000100000e9d         mov        edx, ecx
0000000100000e9f         add        edx, 0x1
0000000100000ea2         mov        dword [rbp+var_34], edx
0000000100000ea5         movsxd     rax, ecx
0000000100000ea8         mov        byte [rbp+rax+var_30], 0x33
0000000100000ead         mov        ecx, dword [rbp+var_34]
0000000100000eb0         mov        edx, ecx
0000000100000eb2         add        edx, 0x1
0000000100000eb5         mov        dword [rbp+var_34], edx
0000000100000eb8         movsxd     rax, ecx
0000000100000ebb         mov        byte [rbp+rax+var_30], 0x34
0000000100000ec0         mov        ecx, dword [rbp+var_34]
0000000100000ec3         mov        edx, ecx
0000000100000ec5         add        edx, 0x1
0000000100000ec8         mov        dword [rbp+var_34], edx
0000000100000ecb         movsxd     rax, ecx
0000000100000ece         mov        byte [rbp+rax+var_30], 0x31
0000000100000ed3         mov        ecx, dword [rbp+var_34]
0000000100000ed6         mov        edx, ecx
0000000100000ed8         add        edx, 0x1
0000000100000edb         mov        dword [rbp+var_34], edx
0000000100000ede         movsxd     rax, ecx
0000000100000ee1         mov        byte [rbp+rax+var_30], 0x32
0000000100000ee6         mov        ecx, dword [rbp+var_34]
0000000100000ee9         mov        edx, ecx
0000000100000eeb         add        edx, 0x1
0000000100000eee         mov        dword [rbp+var_34], edx
0000000100000ef1         movsxd     rax, ecx
0000000100000ef4         mov        byte [rbp+rax+var_30], 0x78
0000000100000ef9         mov        ecx, dword [rbp+var_34]
0000000100000efc         mov        edx, ecx
0000000100000efe         add        edx, 0x1
0000000100000f01         mov        dword [rbp+var_34], edx
0000000100000f04         movsxd     rax, ecx
0000000100000f07         mov        byte [rbp+rax+var_30], 0x79
0000000100000f0c         mov        ecx, dword [rbp+var_34]
0000000100000f0f         mov        edx, ecx
0000000100000f11         add        edx, 0x1
0000000100000f14         mov        dword [rbp+var_34], edx
0000000100000f17         movsxd     rax, ecx
0000000100000f1a         mov        byte [rbp+rax+var_30], 0x6b
0000000100000f1f         mov        ecx, dword [rbp+var_34]
0000000100000f22         mov        edx, ecx
0000000100000f24         add        edx, 0x1
0000000100000f27         mov        dword [rbp+var_34], edx
0000000100000f2a         movsxd     rax, ecx
0000000100000f2d         mov        byte [rbp+rax+var_30], 0x65
0000000100000f32         mov        ecx, dword [rbp+var_34]
0000000100000f35         mov        edx, ecx
0000000100000f37         add        edx, 0x1
0000000100000f3a         mov        dword [rbp+var_34], edx
0000000100000f3d         movsxd     rax, ecx
0000000100000f40         mov        byte [rbp+rax+var_30], 0x7d
0000000100000f45         mov        rax, qword [___stack_chk_guard_100001010]   ; ___stack_chk_guard_100001010
0000000100000f4c         mov        rax, qword [rax]
0000000100000f4f         mov        rsi, qword [rbp+var_8]
0000000100000f53         cmp        rax, rsi
0000000100000f56         jne        loc_100000f64
```

Ooo, `Window -> Show Psuedo Code of Procedure`. That sounds nifty!

```
*(int8_t *)(rbp + (sign_extend_64(0x0) - 0x30)) = 0x66;
    *(int8_t *)(rbp + (sign_extend_64(0x1) - 0x30)) = 0x6c;
    *(int8_t *)(rbp + (sign_extend_64(0x2) - 0x30)) = 0x61;
    *(int8_t *)(rbp + (sign_extend_64(0x3) - 0x30)) = 0x67;
    *(int8_t *)(rbp + (sign_extend_64(0x4) - 0x30)) = 0x7b;
    *(int8_t *)(rbp + (sign_extend_64(0x5) - 0x30)) = 0x72;
    *(int8_t *)(rbp + (sign_extend_64(0x6) - 0x30)) = 0x30;
    *(int8_t *)(rbp + (sign_extend_64(0x7) - 0x30)) = 0x62;
    *(int8_t *)(rbp + (sign_extend_64(0x8) - 0x30)) = 0x34;
    *(int8_t *)(rbp + (sign_extend_64(0x9) - 0x30)) = 0x32;
    *(int8_t *)(rbp + (sign_extend_64(0xa) - 0x30)) = 0x29;
    *(int8_t *)(rbp + (sign_extend_64(0xb) - 0x30)) = 0x33;
    *(int8_t *)(rbp + (sign_extend_64(0xc) - 0x30)) = 0x23;
    *(int8_t *)(rbp + (sign_extend_64(0xd) - 0x30)) = 0x66;
    *(int8_t *)(rbp + (sign_extend_64(0xe) - 0x30)) = 0x31;
    *(int8_t *)(rbp + (sign_extend_64(0xf) - 0x30)) = 0x33;
    *(int8_t *)(rbp + (sign_extend_64(0x10) - 0x30)) = 0x34;
    *(int8_t *)(rbp + (sign_extend_64(0x11) - 0x30)) = 0x31;
    *(int8_t *)(rbp + (sign_extend_64(0x12) - 0x30)) = 0x32;
    *(int8_t *)(rbp + (sign_extend_64(0x13) - 0x30)) = 0x78;
    *(int8_t *)(rbp + (sign_extend_64(0x14) - 0x30)) = 0x79;
    *(int8_t *)(rbp + (sign_extend_64(0x15) - 0x30)) = 0x6b;
    *(int8_t *)(rbp + (sign_extend_64(0x16) - 0x30)) = 0x65;
    *(int8_t *)(rbp + (sign_extend_64(0x17) - 0x30)) = 0x7d;
    if (**___stack_chk_guard == var_8) {
            rax = 0x0;
    }
    else {
            rax = __stack_chk_fail();
    }
    return rax;
}
```

Yesssss. I can read something C-like a heck of a lot easier than assembly. It looks like they're taking those values you see at the far right, and sticking them into different memory addresses, sequentially. Much like you do to build a string in C. Maybe, just maybe, it is a string? So lets paste that into a text doc, hack it to pieces so we just have those final hex values, and run it through one of my python tools I wrote to see what those hex values look like in ASCII.

A little search and replace later and these are the hex values. I wrote them out to `trythis.hex`.

```
66 6c 61 67 7b 72 30 62 34 32 29 33 23 66 31 33 34 31 32 78 79 6b 65 7d
```

And now let's see what they are...

```
$ ./hexasc.py trythis.hex 
flag{r0b42)3#f13412xyke}
```

JACKPOT!

## Conclusion

If we were able to run this executable, I suspect it would have printed the flag on the screen. Perhaps other teams with OSX machines did just that. Or perhaps there were other oddities about it that prevented that from happening. I don't know. All I know is this is how _I_ got the flag. And considering the name of the challenge ("Reverse Engineering"), I think this was the right way to do it. Either way I had fun and I learned something. Those are the real goals.

If you're interested in `hexasc.py` or some of my other simple conversion tools, check out my GitHub repo: https://github.com/QBFreak/ctf-tools

I hope this write up helps someone, even if it's only me, next time I have to do something like this.