+++
title = "Profiling in .NET Core"
date = 2021-04-09
[taxonomies]
tags = ["dotnet-diagnostics"]
+++

## installation of prerequisites

This guide is for popos 20.10 (groovy like)

**perfcollect**

```bash
$ curl -OL https://aka.ms/perfcollect
$ chmod +x perfcollect
$ # sudo ./perfcollect install
```

Copy the perfcollect bash script to `~/bin`.

Under `Pop!_OS` the script will resolve it to Debian (and not Ubuntu), because
the following `DISTRIB_ID` is unknown

```
$ cat /etc/lsb-release
DISTRIB_ID=Pop
DISTRIB_RELEASE=20.10
DISTRIB_CODENAME=groovy
DISTRIB_DESCRIPTION="Pop!_OS 20.10"
```

Therfore we do _not_ use `sudo ./perfcollect install`, because of this and other issues. Instead we manually install the deps below.

**linux-headers**

Must be downloaded to install perf for correct kernel version `uname -r`

```bash
$ sudo apt-get install linux-headers-$(uname - r)
```

**perf**

```bash
$ sudo apt-get install linux-tools-common
$ sudo apt-get install linux-tools-generic
$ sudo apt-get install linux-tools-$(uname -r)
$ sudo apt-get install linux-cloud-tools-generic
$ sudo apt-get install linux-cloud-tools-$(uname -r)
```

If you kernel version and linux-tool-generic version are same, you may be done,
and `perf --version` should report a version (and no warning/error).

```bash
$ perf --version
perf version 5.11.7
```

You could also get

```bash
perf --version
WARNING: perf not found for kernel 5.8.0-7642

  You may need to install the following packages for this specific kernel:
    linux-tools-5.8.0-7642-generic
    linux-cloud-tools-5.8.0-7642-generic

  You may also want to install one of the following packages to keep up to date:
    linux-tools-generic
    linux-cloud-tools-generic
```

**NOTES**

- `linux-tools|linux-tools-common|linux-tools-generic` always points at the tools for the most up to date
kernel version. When running an older kernel, if you want perf without
rebooting to the newer kernel you have to explicitly install the tools paired
with that kernel (hence the `uname -r` shell expansion).
- To see the installed version of linux-tools (perf)

```bash
$ ls /usr/lib/linux-tools
5.11.0-7612-generic/
```

**lttng**

```bash
# groovy requires 12.2.x releaseline branch
$ sudo add-apt-repository ppa:lttng/ppa
$ sudo apt-get update
```

The ppa will make it possible to use v12.2.5 of lttng-modules

```bash
$ apt policy lttng-modules-dkms
lttng-modules-dkms:
  Installed: 2.12.x+stable+git1398+202103251918~ubuntu20.10.1
  Candidate: 2.12.x+stable+git1398+202103251918~ubuntu20.10.1
  Version table:
 *** 2.12.x+stable+git1398+202103251918~ubuntu20.10.1 500
        500 http://ppa.launchpad.net/lttng/ppa/ubuntu groovy/main amd64 Packages
        500 http://ppa.launchpad.net/lttng/ppa/ubuntu groovy/main i386 Packages
        100 /var/lib/dpkg/status
     2.12.2-1ubuntu1 500
        500 http://us.archive.ubuntu.com/ubuntu groovy/universe amd64 Packages
        500 http://us.archive.ubuntu.com/ubuntu groovy/universe i386 Packages
```

```bash
$ sudo apt-get install lttng-tools
$ sudo apt-get install lttng-modules-dkms
$ sudo apt-get install liblttng-ust0
$ sudo apt-get install liblttng-ust-dev
```

**NOTE**: `liblttng-ust0` provides the tracer shared library (the runtime dependency of an instrumented app/library) and `liblttng-ust-dev` provide the necessary to compile applications/library with lttng-ust tracepoints.


`lttng` should report a version.

```bash
$ lttng --version
lttng (LTTng Trace Control) 2.12.3 - (Ta) Meilleure
```

## perfcollect

`percollect` will use `perf` to sample CPU stacks at 1000 Hertz:

```bash
$ perf record -k 1 -g --pid=228368 -F 1000 -e cpu-clock
# or if no PID
$ perf record -k 1 -g -a -F 1000 -e cpu-clock
```

We now have installed the following prerequisites:

- `perf`: the Linux Performance Events subsystem and companion user-mode collection/viewer application. perf is part of the Linux kernel source, but is not usually installed by default.

- `LTTng`: Used to capture event data emitted at runtime by CoreCLR. This data is then used to analyze the behavior of various runtime components such as the GC, JIT, and thread pool.

We need to enable an environment variable in order to provide symbol information.

```bash
export COMPlus_PerfMapEnabled=1
export COMPlus_EnableEventLog=1
```

`COMPlus_PerfMapEnabled=1` tells the runtime to emit information that enables `perf_event` to resolve JIT-compiled code symbols.

## Sampling (aka profiling) with perf (perfcollect)

**terminal 1**

```bash
$ dotnet bin/release/net5.0/TraceProfilingDemo.dll
```

**terminal 2**

```bash
$ ps -aux | grep 'TraceProfilingDemo'
maxfire   **186749**  0.3  0.0 3137176 31852 pts/2   Sl+  15:37   0:00 **dotnet bin/release/net5.0/TraceProfilingDemo.dll**
$ sudo perf record -g --pid 186749
```

or if not attaching (and only using one terminal)

```bash
$ sudo perf record -g -- dotnet bin/release/net5.0/TraceProfilingDemo.dll
```

we ask `perf` to collect call stacks (-g).

Then

```bash
$ sudo perf report
```

**NOTE**: `perf_events` has JIT support to solve this, which requires the .NET Runtime
to maintain a `/tmp/perf-{PID}.map` file for symbol translation. You enables/disable
writing `/tmp/perf-$pid.map` on Linux systems using `COMPlus_PerfMapEnabled`.

**NOTE**: The `perf record` command saves in the `perf.data` unique identifiers
for all ELF images relevant to the measurement. In per-thread mode, this includes
all the ELF images of the monitored processes. In cpu-wide mode, it includes all
running processes running on the system. Those unique identifiers are generated
by the linker if the `-Wl,--build-id` option is used. Thus, they are called build-id.
The build-id are a helpful tool when correlating instruction addresses to ELF images.
To extract all build-id entries used in a perf.data file, issue:

```bash
$ readelf -n /usr/lib/linux-tools-5.11.0-7612/perf | grep -e '[B|b]uild [I|i][d|D]:'
    Build ID: fffd2b3dec83e5d91c81d73f38e0b127b0ef2a09
$ readelf -n /usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5/libclrjit.so | grep -e '[B|b]uild [I|i][d|D]:'
    Build ID: d1a471bd128c619ccac914129ec3c6a19e977c04
```

An ELF build id is a unique identifier assigned by the linker to an executable.

**sysctl**

Read the current value:

```bash
$ sysctl kernel.kptr_restrict
kernel.kptr_restrict = 1
```

Modify the value:

```bash
$ sudo sysctl -w kernel.kptr_restrict=0
sysctl kernel.kptr_restrict=1
```

To make your modifications reboot persistent, you should edit `/etc/sysctl.conf`
or create a file in `/etc/sysctl.d/50-mytest.conf` (edit the file as root),
containing:

```ini
kernel.kptr_restrict=1
```

In which case you should execute this command to reload your configuration:

```bash
$ sysctl -p /etc/sysctl.conf
```

**Symbols: user code libraries**

We can tell the .NET Runtime (JIT) to emit additional debugging info, which
perf will then use to convert function addresses to their names. The only thing
we need to do for to happen is to set `COMPlus_PerfMapEnabled` environmental variable
to 1

**terminal 1**

```bash
# COMPlus_PerfMapEnabled tells the runtime to emit information
# that enables perf_event to resolve JIT-compiled code symbols.
$ COMPlus_PerfMapEnabled=1 dotnet bin/release/net5.0/TraceProfilingDemo.dll
```

**Symbols: framework libraries**

The Framework libraries (System.Private.CoreLib.dll etc) should also have debuginfo (symbols).
The symbols are different than app-level symbols because the framework is pre-compiled
while apps are just-in-time-compiled. For code like the framework that was precompiled
to native code, you need a special tool called crossgen that knows how to generate the
mapping from the native code to the name of the methods.

If you place the crossgen tool in the same directory as the .NET Runtime DLLs
(e.g. libcoreclr.so), then perfcollect can find it and add the framework symbols
to the trace file for you.

We can use a well-known trick. The crossgen tool is part of any self-contained
published app. We can copy the crossgen tool from any self contained package.

```bash
$ dotnet publish --self-contained -r linux-x64 -c release TraceProfilingDemo.csproj
```

As a side effect of creating the self-contained application the dotnet 'muxer'
will download a nuget package called `runtime.linux-x64.microsoft.netcore.app` and
place it in the directory `~/.nuget/packages/runtime.linux-x64.microsoft.netcore.app/VERSION`,
where VERSION is the version number of your .NET Core runtime (e.g. 5.0.5).
Under that folder is a tools directory and inside there is the crossgen tool you need.

```bash
$ find ~/.nuget/packages -iname 'crossgen'
/home/maxfire/.nuget/packages/microsoft.netcore.app.runtime.linux-x64/3.1.1/tools/crossgen
/home/maxfire/.nuget/packages/microsoft.netcore.app.runtime.linux-x64/5.0.5/tools/crossgen
/home/maxfire/.nuget/packages/microsoft.netcore.app.runtime.linux-x64/5.0.3/tools/crossgen
/home/maxfire/.nuget/packages/microsoft.netcore.app.runtime.linux-x64/5.0.0/tools/crossgen
/home/maxfire/.nuget/packages/microsoft.netcore.app.runtime.linux-musl-x64/5.0.0/tools/crossgen
/home/maxfire/.nuget/packages/microsoft.netcore.app.runtime.osx-x64/5.0.0/tools/crossgen
# copy
$ cp ~/.nuget/packages/microsoft.netcore.app.runtime.linux-x64/5.0.5/tools/crossgen \
     /usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5
# check
$ find /usr/share/dotnet/ -iname 'crossgen' -type f
/usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5/crossgen
```

Once you have done this, perfcollect will use crossgen to include framework symbols.
The warning that perfcollect used to issue should go away. This only has to be one
once per machine (until you update your runtime).

**NOTE:** Only `perf record` and `lttng create session` via `perfcollect collect` will
use crossgen. If using `perf record` directly, use `COMPlus_ZapDisable=1`.

**Alternative: Turn off use of precompiled code**

If you don't have the abiltiy to update the .NET Runtime (to add crossgen),
there is another approach to getting framework symbols. You can tell the runtime
to simply not use the precompiled framework code. The code will be Just in
time compiled and the special crossgen tool is not needed.

This works, but will increase startup time for your code by something like
a second or two. If you can tolerate that (you probably can), then this is an
alternative.

```bash
$ COMPlus_PerfMapEnabled=1 COMPlus_ZapDisable=1 dotnet bin/release/net5.0/TraceProfilingDemo.dll
```

**NOTE:** This can also be used when debugging (sourcelinked) framework code in IDEs
to force local variables in stack frames to have symbols.

**Symbols: native runtime code**

We use the [dotnet-symbol CLI tool](https://github.com/dotnet/symstore/blob/main/src/dotnet-symbol/README.md).
First download all the symbol files for the shared runtime to a temporary folder

```bash
$ mkdir /tmp/symbols
$ dotnet symbol --symbols --output /tmp/symbols /usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5/lib*.so
Downloading from http://msdl.microsoft.com/download/symbols/
Writing files to /tmp/symbols
Writing: /tmp/symbols/libSystem.IO.Compression.Native.so.dbg
Writing: /tmp/symbols/libSystem.Native.so.dbg
Writing: /tmp/symbols/libSystem.Net.Security.Native.so.dbg
Writing: /tmp/symbols/libSystem.Security.Cryptography.Native.OpenSsl.so.dbg
Writing: /tmp/symbols/libclrjit.so.dbg
Writing: /tmp/symbols/libcoreclr.so.dbg
Writing: /tmp/symbols/libcoreclrtraceptprovider.so.dbg
Writing: /tmp/symbols/libdbgshim.so.dbg
Writing: /tmp/symbols/libhostpolicy.so.dbg
Writing: /tmp/symbols/libmscordaccore.so.dbg
Writing: /tmp/symbols/libmscordbi.so.dbg
```

Then copy the symbols into the shared framework (aka runtime) so that native debuggers
like `lldb` or native tracers like `per_event` and `lltng` can find them

```bash
$ find /usr/share/dotnet/ -iname 'libcoreclr.so' -type f
/usr/share/dotnet/shared/Microsoft.NETCore.App/3.1.13/libcoreclr.so
/usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5/libcoreclr.so
$ sudo cp /tmp/symbols/* /usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5
# Checking copy
$ find /usr/share/dotnet/ -iname 'libcoreclr.*' -type f
/usr/share/dotnet/shared/Microsoft.NETCore.App/3.1.13/libcoreclr.so
/usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5/libcoreclr.so.dbg
/usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.5/libcoreclr.so
```

TODO: Write a script to automate this for every new runtime version, and put it in `~/bin/dotnet-dbg-symbols`.
Should probably also download and copy crossgen.

## Sampling and Tracing with perfcollect

`perfcollect` uses `perf_event` and `lttng` simultanously.

**terminal 1**

```bash
# COMPlus_PerfMapEnabled tells the runtime to emit information
# that enables perf_event to resolve JIT-compiled code symbols.
$ COMPlus_PerfMapEnabled=1 dotnet bin/release/net5.0/TraceProfilingDemo.dll
```

**terminal 2**

```bash
$ sudo ~/bin/perfcollect collect mysession -nolttng -pid 207180
```

To view it

```bash
$ perfcollect view mysession.trace.zip
# or
$ perfcollect view mysession.trace.zip --viewer lltng
```

Check the log

```bash
$ unzip mysession.trace.zip
$ bat mysession.trace/perfcollect.log
```

Also check that `perf.data.text` is non-empty. This is the file that PerfView will
open and parse, after transfering the *.trace.zip file to Windows.

This is content of the unzipped *.trace.zip file created by perfcollect

```bash
$ ll
Permissions Size User    Date Modified Name
drwxr-xr-x     - maxfire 10 Apr 01:05  debuginfo
.rw-------  1.2M maxfire 10 Apr 01:05  jit-259356.dump
.rw-r--r--  1.9k maxfire 10 Apr 01:05  Microsoft.Win32.Primitives.ni.{a8cc8fd0-e844-4e1b-9a7b-1ad5ecab8ba2}.map
.rw-r--r--  447k maxfire 10 Apr 01:05  perf-259356.map
.rw-------  4.2M maxfire 10 Apr 01:05  perf-jit.data
.rw-------  3.3M maxfire 10 Apr 01:05  perf.data
.rw-r--r--   45M maxfire 10 Apr 01:05  perf.data.txt
.rw-r--r--   15k maxfire 10 Apr 01:05  perfcollect.log
.rw-r--r--  1.4k maxfire 10 Apr 01:05  perfinfo-259356.map
.rw-r--r--  127k maxfire 10 Apr 01:05  System.Collections.ni.{598cee39-c64b-4542-b9ed-359eca21048c}.map
.rw-r--r--   41k maxfire 10 Apr 01:05  System.Console.ni.{81a94ac1-409c-47a6-bcbc-765f68c9ec2b}.map
.rw-r--r--  4.4M maxfire 10 Apr 01:05  System.Private.CoreLib.ni.{510c7690-83fe-4de5-8a2e-fdf79f2ec929}.map
.rw-r--r--   13k maxfire 10 Apr 01:05  System.Runtime.InteropServices.ni.{caea2a6a-873b-493f-85ff-ad31bb0ee993}.map
.rw-r--r--     0 maxfire 10 Apr 01:05  System.Runtime.ni.{1b676a8e-57d8-40e6-bce4-80d063a96828}.map
.rw-r--r--     0 maxfire 10 Apr 01:05  System.Text.Encoding.Extensions.ni.{07860c2a-14e0-43bd-90a1-8fefaa54f9c2}.map
.rw-r--r--   16k maxfire 10 Apr 01:05  System.Threading.ni.{a5d41e01-ef15-40da-8106-60ee513d891a}.map
```

## Tracing with lttng

COMPlus_EnableEventLog
