+++
title = "Using CLI Diagnostic Tools for .NET Core"
date = 2021-04-02
draft = true
[taxonomies]
tags = ["dotnet-diagnostics"]
+++

Some of the tools can be downloaded via CURL from the URL
`https://aka.ms/{tool-name}/{rid}`

```bash
$ curl https://aka.ms/dotnet-sos/linux-x64
$ dotnet-sos install
```

A better way for dev machine is to use `dotnet tool install` command.


### Tools

- dotnet-sos
- dotnet-symbols
- dotnet-dump
- dotnet-gcdump
- dotnet-trace
- dotnet-counters

## dotnet-symbol

The dotnet-symbol global tool downloads files (symbols, DAC, modules, etc.)
needed for debugging core dumps and minidumps.

The latest versions of LLDB and WinDbg should download the symbols automatically.
But sometimes it does not work.

```
$ dotnet-symbols --output /home/maxfire/symbols /usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.4/libcoreclr.so
$ dotnet-symbol --output /home/maxfire/symbols /usr/share/dotnet/shared/Microsoft.NETCore.App/5.0.4/libcoreclr.so
Downloading from http://msdl.microsoft.com/download/symbols/
Writing files to /home/maxfire/symbols
Writing: /home/maxfire/symbols/libcoreclr.so
Writing: /home/maxfire/symbols/libcoreclr.so.dbg
Writing: /home/maxfire/symbols/libmscordaccore.so
Writing: /home/maxfire/symbols/libmscordbi.so
Writing: /home/maxfire/symbols/mscordaccore.dll
Writing: /home/maxfire/symbols/mscordbi.dll
ERROR: Not Found: libsos.so - 'http://msdl.microsoft.com/download/symbols/libsos.so/elf-buildid-coreclr-d7f152a38161fbd2b5a81ff01acb8066f683d1b5/libsos.so'
ERROR: Not Found: SOS.NETCore.dll - 'http://msdl.microsoft.com/download/symbols/sos.netcore.dll/elf-buildid-coreclr-d7f152a38161fbd2b5a81ff01acb8066f683d1b5/sos.netcore.dll'
$ ll ~/symbols
Permissions Size User    Date Modified Name
.rw-rw-r--  7.1M maxfire  2 Apr 13:51  libcoreclr.so
.rw-rw-r--   90M maxfire  2 Apr 13:51  libcoreclr.so.dbg
.rw-rw-r--  2.6M maxfire  2 Apr 13:52  libmscordaccore.so
.rw-rw-r--  1.8M maxfire  2 Apr 13:52  libmscordbi.so
.rw-rw-r--  1.0M maxfire  2 Apr 13:52  mscordaccore.dll
.rw-rw-r--  1.0M maxfire  2 Apr 13:52  mscordbi.dll
```

NOTE: Native symbols on Linux has dbg extension.

In LLDB we have to setup the downloaded symbols

```bash
(lldb) settings set target.debug-file-search-paths /home/maxfire/symbols
(lldb) target symbols add /home/maxfire/symbols/libcoreclr.so.dbg
```

NOTE: We can add this to our `.lldb` file so that it is a configured user setting.

Apart from downloading symbols, the tool can also process memory dumps.

Running dotnet-symbol against a dump file will, by default, download all the modules, symbols, and DAC/DBI files needed to debug the dump including the managed assemblies. Because SOS can now download symbols when needed, most Linux core dumps can be analyzed using lldb with only the host (dotnet) and debugging modules. To get these files necessary for diagnosing a core dump with lldb run:

```bash
dotnet-symbol --host-only --debugging core_20201230_120705
```

## dotnet-dump

Creates memory dumps of managed processes (**Full**|Heap|Mini) - on Docker requires **SYS_PTRACE** capability

```
$ dotnet-dump collect -n testapp
```

It contains a subcommand to analyze collected dumps

```bash
$ dotnet-dump analyze core_20201230_120705
```

We can also open dump in vs2019, WinDbg, or LLDB.

## dotnet-gcdump

Collects **GC dumps** of live .NET processes

GC dumps are udesful to find root objects or compare the number of objects between dumps. Basically answer
why objects are not collected?

```bash
$ dotnet-gcdump ps
      478 testproj ......
$ dotnet-gcdump collect --process-id 478
Writing .....
```

There is also the **report** subcommand

```bash
$ dotnet-gcdump report 20201230_142837_767.gcdump
```

We can also analyze GC dumps in vs2019 or **PerfView**.

## dotnet-trace

Collects .NET traces (including sampling)

- Provides profiles for common scenarios (for example cpu-sampling, gc-collect)
- We can define our own providers list (--providers {providers})
- We can also use .rsp file to avoid typing long commands (DiagnosticSourcefilters(!))

```bash
$ dotnet-trace collect --profile gc-verbose -p 478
```

In .NET 5, we can also trace the launch of the process

```bash
$ dotnet-trace collect -- ./testproj
```

Assembly loading, runtime configuration, deps file issues etc.

It is best to use **PerfView** or vs2019 to analyze the generated trace files. That
is to move to Windowsa to analyze the collected data.

We can convert the trace file to Speedscope or Chromium formats, if needed

```bash
$ dotnet-trace convert trace.nettrace -f {Chromium|Speedscope}
```

And use browser based tooling to analyze the collected data.

## dotnet-counters

Periodically collects selected counter values (for exampole, cpu-usage, gc-heap-size or exception-count)

```bash
$ dotnet-counters collect -p 767
```

Output counter values are written to text file (CSV or JSON).

The tool can not only collect counter values written to a file, but can also display
the values during a live session

```bash
$ dotnet-counters monitor --counters System.Runtime[cpu-usage,gc-heap-size] -p 767
```