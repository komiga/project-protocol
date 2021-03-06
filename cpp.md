
# C++

Nearly all C++ projects require a C++11-compliant compiler and
standard library. Specific requirements are given in the project's
README.

API documentation is generated by Doxygen. For C++11 compliance,
you'll need the newest version (projects are on 1.8.9.1). Use the
`doc/build.sh` script to generate HTML.

## Support

As far as I know, none of my projects support Windows/MSVC. This is (as of
writing) primarily due to Microsoft's lack of C++11 support.

duct++ is the one of the few projects that supports GCC and libstdc++.¹ Clang
is required for almost everything else due to either libstdc++'s lack of C++11
support or ceformat's human rights violations. The latter concern may be more
easily addressed when C++1y lands with relaxed constexpr constraints, but
sufficient cries of agony may sway me into making ceformat support GCC by
making the constexpr artwork configurably constexpr.

libstdc++ may be supported for some projects that require Clang, but it's best
to use libc++ -- especially on Linux.

**1**: This might not be accurate to the latest GCC and libstdc++. If a project
builds with a GCC/libstdc++ version greater than 4.7.3, let me know.

## Toolchain setup

All C++ projects use Premake 4.4-beta5+ with precore for the build system.
Premake generates a platform-specific makefile or project file (VS, Xcode) from
a Premake script, which uses Lua. precore is a set of extensions to simplify
Premake scripts. Projects do not support Premake 5.x (which greatly changes the
interface and is still in alpha).

1. Install Premake 4.4-beta5+ from http://industriousone.com/premake/download.
2. `$ git clone https://github.com/komiga/precore.git` somewhere stationary.
3. Add the variable `PRECORE_ROOT` to your `.bashrc` pointing to precore's root.
   This is required by the Premake scripts to import precore.

Next you should install Clang 3.6+ and libc++. The current stable Clang should
do, but the latest SVN revision (read: unstable) of libc++ is preferred. If
you're on Linux, see "Build on Linux using …" here: http://libcxx.llvm.org.

Once everything is working properly, the build steps are:

1. `$ premake4 [options] <generate-action>`.
2. `$ make [config=...]` or compile with your IDE.

Libraries will likely have build configurations for debug/release and
architecture. With GNU Make, an x86_64 debug build would be
`$ make config=debug64`. The native-arch debug config should be the default.

If you want to compile with Clang using GNU Make, it's a bit hacky because
Premake doesn't currently support it. You'll have to first:

```
$ export CC=clang CXX=clang++
```

And then run Premake with (assuming libc++):

```
$ premake4 --clang --stdlib=c++ gmake
```

The previous command uses `opt-clang` from precore. C++11 projects will also
use `c++11-core` from precore (implicitly) to specify `-std=c++11` for Linux
and Mac OS.

## Importing

There should be a `build.lua` file in the root of every project. It wires up the
precore configs for the project's dependencies and whatnot. It is effectively
the entire build configuration of the project, as well as init-configs for the
Premake projects in case you want to build them locally.

Dependencies that use the `build.lua` script can be imported with
`precore.import()` and linked to with the precore config `<project_name>.dep`.

These assume the dependency is already compiled (into `<PROJECT_NAME>/build/`).
The import path is not modified unless the `<project_name>.projects` config is
executed.

`build.lua` always relies on a global `DEP_PATH` substitution defined to a
directory containing directories of all the expected libraries. These should
just be symlinks to project roots, or builds of libraries otherwise.
