+++
title = "Installing and Using SOS on Windows"
date = 2021-01-24
[taxonomies]
tags = ["dotnet-diagnostics"]
+++

I want to debug .NET Core 3.x (and .NET 5) code.

I was a bit confused after reading the [documentation](https://github.com/dotnet/diagnostics/blob/master/documentation/installing-sos-windows-instructions.md). I have [commented](https://github.com/dotnet/diagnostics/issues/496#issuecomment-766253226) on a github issue to figure this out.

<!-- more -->

When debugging at the commandline

```powershell
$ cdb dotnet .\bin\release\net5.0\Sample.dll
```

I could tell that the SOS extension wasn't loaded, and I struggled to load it with `.loadby sos coreclr` (which is the old/wrong way, because `SOS.dll` is no longer distributed next to `coreclr.dll` and the `LoadLibrary` call will fail).

Then I found `C:\Program Files\dotnet\shared\Microsoft.NETCore.App\3.1.11\SOS_README.md`,

```
SOS and other diagnostic tools now ship of band and work with any version of the .NET Core runtime.

SOS has moved to the diagnostics repo here: https://github.com/dotnet/diagnostics.git.

Instructions to install SOS: https://github.com/dotnet/diagnostics#installing-sos.
```

that says that SOS is no longer distributed with the SDK.

Long story short: Install SOS on Windows the same as on Linux:

```powershell
$ dotnet tool install -g dotnet-sos
.
$ dotnet-sos install
Installing SOS to C:\Users\maxfire\.dotnet\sos from C:\Users\maxfire\.dotnet\tools\.store\dotnet-sos\5.0.160202\dotnet-sos\5.0.160202\tools\netcoreapp2.1\any\win-x64
Creating installation directory...
Copying files...
Execute '.load C:\Users\maxfire\.dotnet\sos\sos.dll' to load SOS in your Windows debugger.
SOS install succeeded
```

After that everything worked in Windbg/Cdb wrt SOS, and I could debug .NET Core 3.x (.NET 5.x.) on Windows.
