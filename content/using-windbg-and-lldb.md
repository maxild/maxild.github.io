+++
title = "Using Windbg (cdb) and lldb"
date = 2021-03-30
draft = true
[taxonomies]
tags = ["dotnet-diagnostics"]
+++

## Use Cases

- User mode live debugging (native and managed code)
- Kernel mode debugging
- Dump analysis (post-mortem debugging)
- Open Binary (open chrash dump)

<!-- more -->

## Commands

Most common commands

- `.hh`
- `k`
- `dv`
- `dc`
- `x`
- `ln`
- `s`
- `dt`
- `r`
- `p`
- `t`
- `e`
- `u`
- `bp`
- `~` (threads in user mode)
- `|` (processes in user mode)

### Native commands

Comes with debugger engine. Covers the core commands (breakpoint, read/write registers/memory).

### Meta commands

Config for the debugger. Start with dot (`.`).

### Extension commands

Start with bang (`!`). We will concentrate on the SOS extension in order to debug
managed code in the native debuggers on both
Windows and Linux.

### Other commands

Keywords in command scripts (`.for`, `.while`, `.continue` etc).

## Symbols and Modules

If you include the string cache*; in your symbol path, symbols loaded from any element that appears to the right of this string are stored in the default symbol cache directory on the local computer.

If you include the string `cache*localsymbolcache;` in your symbol path,
symbols loaded from any element that appears to the right of this string are
stored in the `localsymbolcache` directory. Therefore the following path
will load microsoft public symbols, and cache them in the `C:\Users\maxfire\.symbolcache`
folder.

```
cache*C:\Users\maxfire\.symbolcache;SRV*https://msdl.microsoft.com/download/symbols
```

|  Command     |   Example  |
| ------------ | ---------- |
| .reload | `.reload /f notepad.exe`  |
| !sym noisy | |
| !sym quiet | |
| .load |   |
| !itoldyouso <module> | `!itoldyouso notepad` |
| !lmi <module> | `!lmi notepad` |
| !chksym <module> |  `!chksym notepad` |
| k | |
| .sympath |  |
| .symfix |  |
| ld <module> | |
| lm  | `lm` |
| lmvm <module> | `lmvm notepad` |
| x <module>!<function> | `x notepad!wWinMain` or `x notepad!*Main*` |
| ln <address> | ```ln 00007ff7`74b6b0a0``` |

Target = Debuggee
Target can be running process (live debugging) or some type of dumb (post-mortem debugging)

Learn to use procdump (sysinternals tool)

Learn `module!symbol` notation (e.g. kernel32!CreateFileW)

Checkout `.prefer_dml 1`, `version`, `.logopen` (preserve output)

NOTE: When you attach the debugger to a process, the debugger injects a thread
that will inject a software breakpoint at the instruction pointer (`__asm int 3` with
opcode 0xcc). This is how the target/debuggee/process is suspended by the debugger
waiting for your input.

## Process and Threads

|     (process)
||    (mode)

## Basic thread commands

~
~*k
~*e!dumpstack
~5s   (switch to thread 5)
!threads  (managed threads)
!dumpstack (managed stack trace)
!dumpstack -ee (only managed frames)
!eestack (all managed threads)
~19e!dumpstack
!clrstack (also can show locals `-l`, args `-p`, or both `-a`)

## Callstack

k
kb
kvn
kM (has .frame and dv commands as DML)

## Variables

dv <pattern> (each thread has a stack, each stack frame has some local variables, dv can inspac variables)
? <expr>
r <register>
?? <c++=expr>

**lldb**
frame variable <variable>
print <expr>

## Registers and Memory

NOTE: Quick refresher on x64 calling conventions: 4 register fast call convention. First four args passed in

- rcx
- rdx
- r8
- r9

NOTE: After that, there is stack-backing


r command

d* commands

dps
dc

db (bytes)
dp (pointers)
du (unicode)
dw (word)

Can be used without symbols (low-level)

d{format} {address} (.hh d, db, dp)
dps {address}
dt {type} {address}       (dump type -- class, struct, primitive)
!address {address}
e{format} {address}

dt mylist
dx mylist (windbg preview feature, linq queries)

dt* (lots of options)

dt <type>       (information from symbols here)
dt <local variable>
dt <type> <pointer>
dt -r<depth> <type> <pointer>
dt <type> <field> <pointer>

**ldbb**

memory read
memory region
memory write

## Unassemble (disassembly)

u*
u {adddress}
uf (unassemble function)
uf /c
ub (unassemble backwards -- how did the code reach the break/execution point)

u {address} L10
ub {return address}

## Controlling debuggee (program) execution

.restart

g   (go)
gu
t
p
{g|t|p}a {address}
wt

qd  (detach)

**lldb**

thread step-out
thread step-in (s)   step into code
thread step-inst (si)      step into assembly
thread step-over (n)
thread step-inst-over (ni)
thread until -a {address}


## Handling debugging events (exceptions and events)

sx* set exception (command to control the behavior of the debugger when it encounters exceptions or events)

sxe   (enable exception break)
sxd   (disable exception break)

sxe   (1st chance exception break)
sxd   (2nd chance exception break)
sxn   (notity=output/echo)
sxi   (ignore)
gn

sxe clr (break on all managed exceptions, 1st chance managed exceptions)
sxe av  (break on AV's)

Check out `!StopOnException` for more granular managed exception handling

sxe ld clr
sxe ld coreclr

sxe ld clrjit

**lldb**

process handle {signal}
breakpoint set -E

## Code breakpoints (aka function address breakpoints, pointer/address breakpoint)

bl    (list all breakpoints)
bp {adddress|function}     (create)
b{e|d} {bp-number}      (enable/disable)
bc {bp-number|*}       (remove)

```
bd 1     (disable)
be 1     (enable)
bc 1     (clear)
bc *     (all supportsd star/wildcard)
```

.bpcmds

bm (combine x and bp)
bm HelloWorld!*main*

**lldb**

break list
break set -a {address}
break set -n {function}
break {enable|disable} {bp-number}
break delete {bp-number}

## Data breakpoints (break on access, aka memory breakpoints, watchpoints)

Break just after access to memory location

r=read
w=write

ba r{bytesize} {address}
ba w{bytesize} {address}

**lldb**

watch[point] list
watch[point] {address}

## Conditional breakpoints

A small predicate/program/script is given together with the breakpoint command.

We can even use the program to print the stack on break and just "go"

It is always better to use `gc` than `g` to go from a conditional breakpoint.

```
bp HelloWorld!MytestFunc ".if ( poi(testVar)>0n1500 ) {} .else {k;gc}"
bp kernet32!CreateFileW "du @rcx; gc"  (dump arg, go -- 1st arg is in register RCX)
```

NOTE: The do nothing branch `{}` is the breakpoint.

## Managed debugging in Native debuggers

https://docs.microsoft.com/en-us/dotnet/framework/tools/sos-dll-sos-debugging-extension

We need translation layer (SOS) because of

- JIT compilation
- assembly metadata
- garbage collection

### Windows

In order to use windbg/cdb we have to

```powershell
$ dotnet tool [install|update] -g dotnet-sos
$ dotnet-sos install
```

This installs the extension into your home folder

```
.load C:\Users\maxfire\.dotnet\sos\sos.dll
!sos.help
```

**NOTE**: If you are using WindDbg Preview, then the extension work out of the box.

### Linux

```bash
$ dotnet tool install -g dotnet-sos
$ dotnet-sos install
Creating installation directory...
Copying files...
Updating existing /home/mikem/.lldbinit file - LLDB will load SOS automatically at startup
SOS install succeeded
```

The extension is automatically loaded by LLDB

```bash
$ lldb
(lldb) soshelp
```

## SOS commands

!sos.help (windbg)
soshelp (lldb)

`!sos.hel clrstack`

Same commands in

- `LLDB`
- `WinDbg`: `!dumbdomain`
- `dotnet-dump analyze`


## Managed code breakpoints

!bpmd {module} {symbol}
!bpmd -md {md}
!bpmd {sourcefile}:{line}
!bpmd -list
!bpmd -clear {bp-number}
!bpmd -clearall

NOTE: The symbol (function) in managed code (IL) may or may not have been JITTED into real code. When JITTED
the sos extension adds a physical breakpoint (as can be seen by `bl`)

Learn `{module}` `{member}` metadata notation (e.g. MyConsoleApp.exe MyNamespace.MyType.Function)

bpmd = bp metadata

## Inspecting manged heap

eeheap
dumpheap
!dumpheap -stat
dumpheap -mt {mt}   (very useful for finding specific types)
dumpheap -type {type}
dumbobj {address}
gcroot {address}
finalizequeue
!da {address}   (dump array)
!refs (memory leak)
!locks (hangs)

!dumpheap -type Exception
!dumpheap -stat

## Threads and callstacks

!threads (windbg)
clrthreads (lldb)
clrstack
dumpstack
dso
pe

## IL inspection

dumpil {md}
ip2md {adddress}
u {md|address}

## Managed exceptions

!pe   (print exception)

.foreach (ex {!dumpheap -type Exception -short}) { .echo "************"; !pe ${ex} }
