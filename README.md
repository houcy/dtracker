DataTracker
===========

About
------
DataTracker is a tool for collecting high-fidelity data provenance from unmodified Linux programs. It is based on [Intel Pin][pin] _Dynamic Binary Instrumentation_ framework and [libdft][libdft] _Dynamic Taint Analysis_ library. The taint marks supported by the original libdft are of limited size and cannot provide adequate fidelity for use in provenance tracking. For this, DataTracker uses a [modified version][libdft-mod] of the library developed at [VU University Amsterdam][vu-cs].

DataTracker was developed at VU University Amsterdam by Manolis Stamatogiannakis.

Requirements
-------------
DataTracker runs on 32bit Linux systems. This limitation is imposed by the current version of libdft. However, the methods of both software are not platform-specific. So, in principle, they can be ported on any platform supported by Intel Pin. The requirements for running DataTracker are:

*  A C++11 compiler and unix build utilities (e.g. GNU Make). 
*  A recent (>=2.13) version of Intel Pin. The framework must be present in directory ``pin`` inside the DataTracker top directory.
*  A suitable version of the [modified libdft][libdft-mod] - typically the latest available. This must be placed in directory ``support/libdft``.
*  Python 2.7 for converting raw provenance to [PROV][prov] format in [Turtle][turtle] syntax.

Installation
-------------
After cloning DataTracker, follow these steps to compile it.

**Build environment:**
On Debian/Ubuntu systems, you should install ``build-essential`` meta-package which will provide a C++ compiler and GNU Make. On other systems, you should either install some equivalent meta-package or install the tools one by one using trial and error.

**Intel Pin:** You can [manually download][pin-dl] a suitable Pin version and extract it in ``pin`` directory. For convenience, a makefile is provided which takes care of this. I.e. it downloads and extracts a suitable Pin version. Invoke it using:

```
make -C support -f makefile.pin
```

**libdft:** The modified libdft is packed as a submodule of DataTracker. You need to disable Git's certificate checking to successfully retrieve it. Because libdft does not use [Pin's makefile infrastructure][pin-makefile] you need to set ``PIN_HOME`` environment variable before compiling it. 

```
export PIN_HOME=$(pwd)/pin
GIT_SSL_NO_VERIFY=true git submodule update --init
make support-libdft
```

**dtracker pin tool**: Finaly compile the pin tool of DataTracker using:

```
make
```

If all above steps were successfull, ``obj-ia32/dtracker.so`` will be created. This is Pin tool containing all the instrumentation required to capture provenance.


Runnning
---------

### Capturing raw provenance
To capture provenance from a program, launch it from the unix shell using something like this:

```
./pin/pin.sh -follow_execv -t ./obj-ia32/dtracker.so <knobs> -- <program> <args>
```

The command runs the program under Pin
In addition to the standard Pin knobs, DataTracker additionally supports these tool-specific knobs:

* ```-stdin [1|0]```: Turns tracking of data read from the standard input on or off. Default if off.
* ```-stdout [1|0]```: Turns logging of provenance of data written to standard output on or off. Default if on.
* ```-stderr [1|0]```: Turns logging of provenance of data written to standard error on or off. Default if off.

Note that launching large programs using the method above takes a lot of time. For such programs, it is suggested to first launch the program and then attach DataTracker to the running process like this:

```
./pin/pin.sh -follow_execv -pid <pid> -t ./obj-ia32/dtracker.so <knobs>
```

The raw provenance generated by DataTracker is contained in file ``rawprov.out``. Any additional debugging information are written in file ``pintool.log``.

### Converting to PROV
The ``raw2ttl.py`` script converts the raw provenance generated by DataTracker to [PROV][prov] format in [Turtle][turtle] syntax. The converter works as a filter. So, a conversion would look like this:

```
python raw2ttl.py < rawprov.out > prov.ttl
```

### Visualizing provenance
For visualization of the generated provenance, we suggest using [``provconvert``][provconvert] from Luc Moreau's [ProvToolbox][provtoolbox]. It is suggested to use the binary release. 

Of course any other PROV-compatible tool can be used, either directly, or via conversion of the Turtle file to a supported syntax.
If you were able to produce any good-looking provenance graph, we'd love to incorporate them in these pages.

Sample programs
----------------
In this repository we plan to also include the sample programs we used for evaluating the effectiveness of DataTracker. ETA: mid-April 2014. 


[pin]: http://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool
[pin-dl]: http://software.intel.com/en-us/articles/pintool-downloads
[pin-makefile]: http://software.intel.com/sites/landingpage/pintool/docs/62732/Pin/html/index.html#MAKEFILES
[libdft]: http://www.cs.columbia.edu/~vpk/research/libdft/
[libdft-mod]: https://git.cs.vu.nl/r.vermeulen/libdft
[vu-cs]: http://www.cs.vu.nl/en/
[turtle]: http://www.w3.org/TeamSubmission/turtle/
[prov]: http://www.w3.org/TR/2013/NOTE-prov-overview-20130430/
[provconvert]: https://github.com/lucmoreau/ProvToolbox/wiki/provconvert
[provtoolbox]: https://github.com/lucmoreau/ProvToolbox/wiki/ProvToolbox-Home
