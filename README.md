# BluePill

**Update:** Our article "On the Dissection of Evasive Malware" describing BluePill was accepted for publication in IEEE Transactions on Information Forensics and Security (TIFS): you can find it [here](https://ieeexplore.ieee.org/document/9018111). *We will extend the instructions below in the next few days* :-)

BluePill is an open-source dynamic analysis framework for neutralizing evasive behavior in malware.
It aims at reconciling transparency requirements typical of automatic malware analysis with manipulation capabilities required for dissection.

The repository contains a polished snapshot of code under active development, and for the time being we share it with the community as such: BluePill is currently a research prototype, and any feedback would be **greatly** appreciated. 

BluePill build on dynamic binary instrumentation (DBI) to observe outputs in adversarial queries that a sample can make on the environment, and fix them when their results would give away the presence of an analysis system or human agents behind it.

It builds on Intel Pin (we just added support for v3.11) and requires Visual Studio 2015 for its compilation. Make sure you extract the working tree of Pin to `C:\Pin311` or change the related property value in the Visual Studio project we provide.

BluePill has been tested on 32-bit malware running on Windows 7 SP1, mainly on a 32-bit install and to a good extent under WoW64. Support to 64-bit malware does not require changes to the design: we handle it in our internal fork, and we will hopefully be releasing this code soon.

To cope with DBI artifacts, BluePill builds on a library of mitigations for Intel Pin devised as part of the [code](https://github.com/season-lab/sok-dbi-security/) from our ACM ASIACCS 2019 paper *SoK: Using Dynamic Binary Instrumentation for Security (And How You May Get Caught Red-Handed)*. We have extended the work with additional mitigations for DBI overheads and other red pills targeting debugging and exceptions.

Once you compile BluePill, you will find a `bluepill32.dll` library in `C:\Pin311`. To run a program under BluePill use:

```
C:\Pin311\pin.exe -t bluepill32.dll -- <file.exe>
```

Change the defaults in `config.h` to control the mitigations used by default or to enable command-line knobs for them (in the latter case set the `FIXED_KNOBS` macro to `0`).

To access dissection capabilities of BluePill you need to connect from a debugger to the GDB remote interface of Pin. To this end you need to provide additional options when invoking Pin of the form:

```
C:\Pin311\pin.exe -appdebug —appdebug_server_port 10000 -t bluepill32.dll -debugger [other options] -- <file.exe>
```

And connect to the desired port number. The application will stay paused until you connect a debugger, but if you instead attach one to the process you will end up debugging Pin and the JIT-ted code. For IDA Pro users select the *Remote GDB debugger* option and connect to `localhost`. To map missing memory segments you can use the `AddSegments.py` IDAPython script available in the `scripts/` folder: we defined a custom `vmmap` GDB command that gets invoked by the script and transfers memory layout information from the pintool to the debugger.

Exception handling requires a workaround for the current GDB server implementation. When you need to pass an exception to the application just send a `wait` command right after you receive the exception message, then disconnect and reconnect IDA to BluePill, which meanwhile will put the execution on hold in response to the command.

*We apologize for the incompleteness/minimalism in the documentation. We wanted to release the code as soon as possible while we keep working on the docs and on making the interfaces smoother :-)*

### Authors
* Daniele Cono D'Elia ([@dcdelia](https://github.com/dcdelia)) - design
* Federico Palmaro ([@nik94](https://github.com/nik94)) - main developer
