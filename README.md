tupcfg
======

**Configure your project for the [tup build system](http://gittup.org/tup/ "Tup home page").**

Motivations
-----------

  * Makefiles are slow and a pain to maintain.
  * AutoTools and CMake are huge pain.

Tup is by design one of the fastest build system, but as it is langage agnostic, 
it doesn't have a clue about library location, compiler version or any high level
system configuration. I initially solved this problem with dirty scripts for each 
project using Tup, and after getting tired of maintaining those scripts, I made up
this small project that saves me a lot of time.

Features
--------

Tupcfg is written in pure Python3, and should work on any platform that support python3.
It has been successfully tested on Linux, MacOSX and Windows.

It lets you configure your project builds in Python3 as well and generate for you Tup files.

**Yes, no more CMake or AutoTools crappy langages, just Python.**

It provides handy tools for C/C++ compiler and libraries, but aims to support more langages, compiler and
libraries. It's up to you to submit/request more features or support :)

Additionally, it has a pluggable generator system that allows you to build your project even when tup is
not available, or for release builds from scratch (Makefile generator is implemented).

Getting started
---------------

### Installation

Just drop the [configure](https://github.com/hotgloupi/tupcfg/blob/master/src/main.py) script
in the root directory of your project.

On unices, you could do:

    $ cd /path/to/your/project
    $ wget 'https://github.com/hotgloupi/tupcfg/raw/master/src/main.py' -O configure
    $ chmod +x configure

This script is written in python3, so you'll obviously need python3 on your computer. If the python
executable does not use python3, you may want to change the first line of the configure script. But
I would recommend that you set python3 as the default python version :).

The configure script will ensure that:
  * You have tup executable somewhere
  * the tupcfg python package is available

You can install yourself these two dependencies, or let the configure script install them for you.

    $ ./configure --self-install --tup-install

Note that same flags could be used later to upgrade tupcfg and tup.

You are now asked to manually edit the file `.config/project.py`, which defines your project rules.


### The project file 

The `.config/project.py` file must define the following function:

    def configure(project, build):
        print("Configuring build", build.directory)

This function is called for each build directory and is in charge of adding targets to the build.
For the purpose of this Getting started section, let's do a simple executable.

    from tupcfg.lang.c import gcc
    
    def configure(project, build):
        compiler = gcc.Compiler(project, build)
        compiler.link_executable('test', ['main.c'])

Assuming you have source file named `main.c` at the root of your project, for example:

    #include <stdio.h>
    
    int main()
    {
        printf("Hello, world!\n");
        return 0;
    }
    
You can now configure your project in a build directory `build`

    $ ./configure build
    Configuring build
    Just run `make -C build`

The first output line comes from our `configure()` function, while the second only appears
when a build directory is created. Now, your project should be something like that:

    $ tree .
    ├── build
    │   ├── Makefile
    │   └── Tupfile
    ├── configure
    └── main.c
    
    1 directory, 4 files
    
The makefile `build/Makefile` is generated for convinience by the configure script. 
It just call the `tup` executable, which is located in `.config/tup/tup` when installed
automatically. The `Tupfile` is generated by the `configure()` function (the compiler add targets).

As suggested, just run `make -C build` and enjoy the magic :)

The configure script
--------------------

### Synopsis

    $ ./configure --help
    usage: configure [-h] [-D DEFINE] [-E EXPORT] [-v] [-d] [--dump-vars]
                     [--dump-build] [--install] [--self-install] [--tup-install]
                     [build_dir]

    Configure your project for tup

    positional arguments:
      build_dir             Where to build your project

    optional arguments:
      -h, --help            show this help message and exit
      -D DEFINE, --define DEFINE
                          Define build specific variables
      -E EXPORT, --export EXPORT
                          Define project specific variables
      -v, --verbose       verbose mode
      -d, --debug         debug mode
      --dump-vars         dump variables
      --dump-build        dump commands that would be executed
      --install           install when needed
      --self-install      install (or update) tupcfg
      --tup-install       install (or update) tup

The script and its underlying library `tupcfg` were designed to give to the user
a help messge each time something is wrong. This means that launching the script several times 
should suffice to get everything working.

### The build directories

You can specify one or more directories where to build your project. The main idea
is to give you the ability to create *variants*. If you do not specify any build directory, 
all build directories are configured.

### Defining variables

You can define some variables for the whole project or for specific builds with -E and -D.
Note that the -D flags applies on all build specified on the command line, or all project builds
when none are specified.

Project variables are saved in the file `.config/.project_vars`, whereas build
variables are store in their respective directory in a file named `.build_vars`.

They contain a python dictionary that could be read or modified with the `pickle` python module.

    >>> import pickle 
    >>> pickle.loads(open('.config/.project_vars', 'rb').read())

You can easily dump all project and builds variables using the `--dump-vars` flag.

### Dumping the build

While this is mainly a debug functionality, dumping all targets can be of a great help
in some cases. Use `--dump-build` when you feel it :)

### Auto install everything

As seen previously the `--tup-install` and `--self-install` flags force the installation or the update
of tup and tupcfg. To install only when tup or tupcfg are not found, use the `--install` flag instead.


