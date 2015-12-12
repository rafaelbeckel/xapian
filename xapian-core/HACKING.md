Instructions for hacking on Xapian
==================================

.. contents:: Table of contents

This file is aimed to help developers get started with working on
Xapian.  The documentation contains a section covering various internal
aspects of the library - this can also be found on the Xapian website
<http://xapian.org/>.

Extra options to give to configure
==================================

Note: Non-developer configure options are described in INSTALL

You will probably want to use some of these if you're going to be developing
Xapian.

--enable-assertions
	This enables compiling of assertion code which will throw
	Xapian::AssertionError if the code detects violating of
	preconditions, postconditions, or fails other consistency checks.

--enable-assertions=partial
	This option enables a subset of the assertions enabled by
	"--enable-assertions", but not the most expensive.  The intention is
	that it should be suitable for use in a real-world system for tracking
	down problems without imposing too much of an overhead (but note that
	we haven't yet performed timings to measure the overhead...)

--enable-log
	This enables compiling code into the library which generates verbose
	debugging messages.  See "Debugging Messages", below.

--enable-log=profile
	In 1.2.0 and earlier, this used to use the debug logging macros to
	report to stderr how long each method takes to execute.  This feature
	was removed in 1.2.1 - you are likely to get better results using
	dedicated profiling tools - for more information see:
	http://trac.xapian.org/wiki/ProfilingXapian

--enable-maintainer-mode
	This tells configure to enable make dependencies for regenerating build
	system files (such as configure, Makefile.in, and Makefile) and other
	generated files (such as the stemmers and query parser) when required.
	These are disabled by default as some make programs try to rebuild them
	when it's not appropriate (e.g. BSD make doesn't handle VPATH except
	for implicit rules).  For this reason, we recommend GNU make if you
	enable maintainer mode.  You'll also need a non-cross-compiling C
	compiler for compiling the Lemon parser generator and the Snowball
	stemming algorithm compiler.  The configure script will attempt to
	locate one, but you can override this autodetection by passing
	CC_FOR_BUILD on the command line like so::

	./configure CC_FOR_BUILD=/opt/bin/gcc

--enable-documentation
	This tells configure to enable make dependencies for regenerating
	documentation files.  By default it uses the same setting as
	--enable-maintainer-mode.

Debugging Messages
==================

If you configure with --enable-log, lots of places in the code generate
debugging messages to tell us what they're up to - this information can be
very useful for debugging both the Xapian library and code which uses it.  But
the quantity of information generated is potentially vast so there's a
mechanism to allow you to select where to store the log and which types of
message you're interested by setting environment variables.  You can:

 * set XAPIAN_DEBUG_LOG to be the path to a file that you would like debugging
   output to be appended to, or to the special value ``-`` to indicate that you
   would like debugging output to be sent to stderr.  Unless XAPIAN_DEBUG_LOG
   is set, no debug logging will be performed.  Occurrences of %p in
   XAPIAN_DEBUG_LOG will be replaced with the current process-id.

 * set XAPIAN_DEBUG_FLAGS to a string of capital letters indicating the types
   of debugging message you would like to display (the default is to log calls
   to API functions and methods).  These letters are shown in the first column
   of the log output, and are also listed in ``common/debuglog.h``.  If the
   first character is ``-``, then the letters indicate those categories of
   message *not* be shown instead.  As a consequence of this, setting
   ``XAPIAN_DEBUG_FLAGS=-`` will give you all debugging messages.

These environment variables only have any effect if you ran configure with the
--enable-log option.

The format is::

    <message type> <pid> [<this>] <message>

For example::

    A 16747 [0x57ad1e0] void Xapian::Query::Internal::validate_query()

Each nested call adds another space before the ``[`` so you can easily see
which function call and return messages correspond.

Debugging memory allocations
============================

The testsuite can make use of valgrind 3.3.0 or newer to check for memory
leaks, reads from uninitialised memory, and some other bugs during tests.

Valgrind doesn't support every platform, but Xapian contains very little
platform specific code (and most of what there is is Microsoft Windows
specific) so even just testing with valgrind on one platform gives good
coverage.

If you have a new enough version of valgrind installed, it's automatically
detected by configure and used when running the testsuite.  The testsuite runs
more slowly under valgrind, so if you wish to disable this auto-detection you
can run configure with:

./configure VALGRIND=

Or you can disable use of valgrind during a particular run of "make check"
like so:

make check VALGRIND=

Or disable it while running a test directly (under sh or bash):

VALGRIND= ./runtest ./apitest

Running test programs
=====================

To run all tests, use ``make check``.  You can also run just the subset of
tests which exercise the inmemory, remote progserver, remote TCP,
multi-database, glass, or chert backends using ``make check-inmemory``,
``make check-remoteprog``, ``make check-remotetcp``, ``make check-multi``,
``make check-glass``, or ``make check-chert``
respectively.

Also, ``make check-remote`` will run the tests on both variants of the remote
backend, and ``make check-none`` will run those tests which don't use any
backend.  These are handy shortcuts when doing development work on a particular
backend.

The runtest script (in the tests subdirectory) takes care of the details of
running the test programs (including setting up the environment so they work
when srcdir != builddir and handling libtool dynamically linked binaries).  To
run a test program by hand (rather than via make) just use:

./runtest ./apitest

You can specify options and arguments.  Individual test programs optionally
take one or more test names as arguments, and you can also pass ``-v`` to get
more verbose output from failing tests, e.g.:

./runtest ./apitest -v deldoc1

If the number of the test is omitted, all tests with that basename are run,
so to run deldoc1, deldoc2, etc:

./runtest ./apitest deldoc

You can also use runtest to run a test program under gdb (or most other tools):

./runtest gdb ./apitest -v deldoc1
./runtest valgrind ./apitest -v deldoc1

Some test programs take special arguments - for example, you can restrict
apitest to the chert backend using ``-bchert``.

There are a few environmental variables which the testsuite harness checks for
which you might find useful:

  XAPIAN_TESTSUITE_SIG_DFL:
    By default, the testsuite harness catches signals and handles them
    gracefully - the current test is failed, and the testsuite moves onto the
    next test.  If you want to suppress this (some debugging tools may work
    better if the signal is not caught) set the environment variable
    XAPIAN_TESTSUITE_SIG_DFL to any value to prevent the testsuite harness
    from installing its own signal handling.

  XAPIAN_TESTSUITE_OUTPUT:
    By default, the testsuite harness uses ANSI escape sequences to give
    colour output if stdout is a tty.  You can disable this feature by setting
    XAPIAN_TESTSUITE_OUTPUT=plain (alternatively, piping the output (e.g.
    through ``cat`` or ``more``) will have the same effect).  Auto-detection
    can be explicitly specified with XAPIAN_TESTSUITE_OUTPUT=auto (or empty).
    Any other value forces the use of colour.  Colour output is always disabled
    on Microsoft Windows, so XAPIAN_TESTSUITE_OUTPUT has no effect there.

  XAPIAN_TESTSUITE_LD_PRELOAD:
    The runtest script will add this to LD_PRELOAD if it is set, allowing you
    to easily load LD_PRELOAD libraries when running the testsuite.  The
    original intended use was to allow use of libeatmydata
    (https://www.flamingspork.com/projects/libeatmydata/) which makes fsync
    and related calls no-ops, but configure now checks for the eatmydata
    wrapper script and this is used automatically.  However, there may be
    other LD_PRELOAD libraries which are useful, so we've left the machinery
    in place.

Speeding up the testsuite with eatmydata
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The testsuite does a lot of small database operations, and the calls to fsync,
fdatasync, etc which Xapian makes by default can slow down testsuite runs
substantially.  There's a handy LD_PRELOAD library called eatmydata
(http://www.flamingspork.com/projects/libeatmydata/), which can help here, by
turning fsync and related calls into no-ops.

You need a version of eatmydata with the eatmydata wrapper script (version 37
or newer), and then configure should auto-detect it and it'll get used when
running the testsuite (via runtest).  If you wish to disable this
auto-detection for some reason, you can run configure with:

./configure EATMYDATA=

Or you can disable use of eatmydata during a particular run of "make check"
like so:

make check EATMYDATA=

Or disable it while running a test directly (under sh or bash):

EATMYDATA= ./runtest ./apitest

Using various debugging, profiling, and leak-finding tools
==========================================================

GCC's libstdc++ supports a debug mode, which checks for various misuses of
the STL - to enable this, define _GLIBCXX_DEBUG when building Xapian:

  ./configure CPPFLAGS=-D_GLIBCXX_DEBUG

For documentation of this option, see:
http://gcc.gnu.org/onlinedocs/libstdc++/debug.html

Note: all C++ code must be compiled with this defined or you'll get problems.
Xapian's API headers include a check that the same setting is used when
building code using Xapian as was used to build Xapian.

To use valgrind (http://www.valgrind.org/), no special build options are
required, but make sure you compile with debugging information (on by default
for GCC) and the valgrind documentation recommends disabling optimisation (with
optimisation, line numbers in error messages can be confusing due to code
inlining, etc):

  ./configure CXXFLAGS='-O0 -g'

To use gdb (http://www.gnu.org/software/gdb/), no special build options are
required, but make sure you compile with debugging information (on by default
for GCC).  You'll probably find debugging easier if you compile without
optimisation (with optimisation, line numbers in error messages can be
confusing due to code inlining, etc, and the values of some variables can't be
printed because they've been eliminated from the code completely):

  ./configure CXXFLAGS='-O0 -g'

To enable profiling for gprof:

  ./configure CXXFLAGS=-pg LDFLAGS=-pg

To use Purify (a proprietary tool):

  ./configure CXXLD='purify c++' --disable-shared

To use Insure (another proprietary tool):

  ./configure CXX=insure

To use lcov (at least version 1.10) to generate a test coverage report (see
`lcov.xapian.org <http://lcov.xapian.org/>`_ for reports) there are two make
targets:

  * coverage-reconfigure: reruns configure in the source tree.  See
    Makefile.am for details of the configure options used and why they
    are needed.
  
  * coverage-check: runs "make check" and generates an HTML report in a
    directory called "lcov".  You can specify extra arguments to pass to the
    ``genhtml`` tool using GENHTML_ARGS, like so::

    make coverage-check GENHTML_ARGS=--html-gzip

If you have runes for using other tools, please add them above, or send them
to us so we can.

Snapshots
=========

If you want to try unreleased Xapian code, you can fetch it from our git
repository.  For convenience, we also provide bootstrapped tarballs (much like
the sourcecode download for any release version) which get built every 20
minutes if there have been any changes checked in.  These tarballs need to
pass "make distcheck" to be automatically uploaded, so using them will help
to assure that you don't pick a "bad" version.  The snapshots are available
from the "Bleeding Edge" page of the Xapian website.

Building from git
=================

When building from a git checkout, we *strongly* recommend that you use
the ``bootstrap`` script in the top level directory to set up the tree ready
for building.  This script will check which directories you have checked out,
so you can bootstrap a partial tree.  You can also ``touch .nobootstrap`` in
a subdirectory to tell bootstrap to ignore it.

You will need the following tools installed to build from git:

* GNU m4 (for autoconf)
* perl 5 (for automake; also for various maintainer scripts)
* python >= 2.3 (for generating the Python bindings)
* GNU make (or another make which support VPATH for explicit rules)
* GNU flex (for building doxygen)
* GNU bison (for building doxygen)
* Tcl (to generate unicode/unicode-data.cc)

For a recent version of Debian or Ubuntu, this command should ensure you have
all the necessary tools and libraries::

    apt-get install build-essential m4 perl python zlib1g-dev uuid-dev wget flex bison tcl

If you want to build Omega, you'll also need::

    apt-get install libpcre3-dev libmagic-dev

On Fedora, the uuid library can be installed by doing::

    yum install libuuid-devel

On Mac OS X, if you're using macports you'll want the following:

  * file (magic.h in configure)

If you're using homebrew you'll want the following::

    brew install libmagic pcre

If you're doing much development work, you'll probably also want the following
tools installed:

* valgrind for better testsuite error finding
* ccache for faster rebuilds
* eatmydata for faster testsuite runs

The repository does not contain any automatically generated files
(such as configure, Makefile.in, Snowball-generated stemmers, Lemon-generated
parsers, SWIG-generated code, etc) because experience shows it's best to keep
these out of version control.  To avoid requiring you to install the correct
versions of the tools required, we either include the source to these tools in
the repo directly (in the case of Snowball and Lemon), or the bootstrap script
will download them as tarballs (autoconf, automake, libtool, and doxygen) or
from git (SWIG), build them, and install them within the source tree.

To download source tarballs, bootstrap will use wget, curl or lwp-request if
installed.  If not, it will give an error telling you the URL to download from
by hand and where to copy the file to.

Bootstrap will then run autoreconf on each of the checked-out subdirectories,
and generate a top-level configure script.  This configure script allows you to
configure xapian-core and any other modules you've checked out with single
simple command, such that the other modules link against the uninstalled
xapian-core (which is very handy for development work and a bit fiddly to set
up by hand).  It automatically passes --enable-maintainer-mode to the
subprojects so that the autotools will be rerun if configure.ac, Makefile.am,
etc are modified.

The bootstrap script doesn't care what the current directory is.  The top-level
configure script generated by it supports building in a separate directory to
the sources: simply create the directory you want to build in, and then run the
configure script from inside that directory.  For example, to build in a
directory called "build" (starting in the top level source directory)::

  ./bootstrap
  mkdir build
  cd build
  ../configure

When running bootstrap, if you need to add any extra macro directories to the
path searched by aclocal (which is part of automake), you can do this by
specifying these in the ACLOCAL_FLAGS environment variable, e.g.::

  ACLOCAL_FLAGS=-I/extra/macro/directory ./bootstrap

If you wish to prevent bootstrap from downloading and building the autotools
pass the --without-autotools option.  You can force it to delete the downloaded
and installed versions by passing --clean.

If you are tracking development in git, there will sometimes be changes
to the build system sources which require regeneration of the generated
makefiles and associated machinery.  We aim to make the build system
automatically regenerate the necessary files, but in the event that a build
fails after an update, it may be worth re-running the bootstrap script to
regenerate the build system from scratch, before looking for the cause of the
error elsewhere.

Tools required to build documentation
-------------------------------------

If you want to be able to build distribution tarballs (with "make dist") then
you'll also need some further tools.  If you don't want to have to install all
these tools, then pass --disable-documentation to configure to disable these
rules (the default state of this follows the setting of
--enable-maintainer-mode, so in a non-maintainer-mode tree, you can pass
--enable-documentation to enable these rules).  Without the documentation,
"make dist" will fail (to prevent accidentally distributing tarballs without
documentation), but you can configure and build.

The documentation tools are:

* doxygen (v1.8.8 is used for 1.3.x snapshots and releases; 1.7.6.1 fails to
  process trunk after PL2Weight was added).
* dot (part of Graphviz.  Doxygen's DOT_MULTI_TARGETS option apparently needs
  ">1.8.10")
* help2man
* rst2html or rst2html.py (in python-docutils on Debian/Ubuntu)
* pngcrush (optional - used to reduce the size of PNG files in the HTML
  apidocs)
* sphinx-doc (in python-sphinx and python3-sphinx on Debian/Ubuntu, or as
  sphinx via pip install)

For a recent version of Debian or Ubuntu, this command should install all the
required documentation tools::

    apt-get install doxygen graphviz help2man python-docutils pngcrush python-sphinx python3-sphinx

Documentation builds on OS X
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Mac OS X, if you're using homebrew, you'll want the following::

    brew install doxygen help2man graphviz pngcrush

(Ensure you're up to date with brew, as earlier packaging of graphviz
didn't properly install dot.)

You also need sphinx and docutils, which are python packages; you can
install them via pip::

    pip install sphinx docutils

You may find it easier to use homebrew to install python first, so
these packages are separate from the system python::

    brew install python

If you install both python (v2) and python3 (v3) via homebrew, you
will be able to build bindings for both; you'll then need to install
sphinx for python3::

    pip3 install sphinx

PDF versions of docs
~~~~~~~~~~~~~~~~~~~~

As of 1.3.2, we no longer build PDF versions of the API docs by default, but
you can build them yourself with::

    make -C docs apidoc.pdf

Additional tools are needed for these:

* gs (part of Ghostscript)
* pdflatex (in texlive-latex-base on Debian/Ubuntu)
* epstopdf (in texlive-extra-utils on Debian/Ubuntu)
* makeindex (in texlive-binaries on Debian/Ubuntu, or texlive-base-bin for older releases)

Note that pdflatex, epstopdf, gs, and makeindex must all currently be on your
path (as specified by the environmental variable PATH), since doxygen will look
for them there.

For a recent version of Debian or Ubuntu, this command should install these
extra tools::

    apt-get install ghostscript texlive-latex-base texlive-extra-utils texlive-binaries texlive-fonts-extra texlive-fonts-recommended texlive-latex-extra texlive-latex-recommended

On Mac OS X, if you're using macports you'll want the following:

  * texlive (pdflatex during build)
  * texlive-basic (for makeindex in configure)
  * texlive-latex-extra (latex style)

Alternatively, you can install MacTeX from http://www.tug.org/mactex/ instead
of texlive, texlive-basic and texlive-latex-extra.

The homebrew texlive package only supports 32 bit systems, so even if you're
using homebrew, you'll probably want to install MacTeX from
http://www.tug.org/mactex/ instead.

Autotools versions
------------------

* autoconf 2.68 is used to generate snapshots and releases.

  autoconf 2.64 is a hard minimum requirement.

  autoconf 2.60 is required for docdir support and AC_TYPE_SSIZE_T.

  autoconf 2.62 generates faster configure scripts and warns about unrecognised
  options passed to configure.

  autoconf 2.63 fixes a regression in AC_C_BIGENDIAN introduced in 2.62
  (Omega uses this macro).

  autoconf 2.64 generates smaller configure scripts by using shell functions.

* automake 1.15 is used to generate snapshots and releases.

  automake 1.12.2 is a hard minimum requirement.  This version fixes a
  security issue (CVE-2012-3386) in the generated `make distcheck` rules.

  automake 1.12 is needed to support using LOG_COMPILER to specify a testsuite
  driver (used by xapian-bindings).

* libtool 2.4.6 is used to generate snapshots and releases.

  libtool 2.2.8 is the current hard minimum requirement.

  libtool 2.2 is required for us to be able to override link_all_deplibs_CXX
  and sys_lib_dlsearch_path_spec in configure.  It also fixes some
  long-standing issues and is significantly faster.

Please tell us if you find that newer versions of any of these tools work or
fail to work.

There is a good GNU autotools tutorial at
<http://www.lrde.epita.fr/~adl/autotools.html>.

Building from git on Windows with MSVC
--------------------------------------

The windows build process is maintained in the xapian-maintainer-tools
directory in the Xapian git repository.  See the win32msvc/README file in that
directory for details of how to build from git.

Using a Vagrant-driven Ubuntu virtual machine
---------------------------------------------

Note: Vagrant support is experimental. Please report bugs in the
normal fashion, to http://trac.xapian.org/newticket, or ask for help
on the #xapian IRC channel on Freenode.

If you have Vagrant (http://www.vagrantup.com/, tested on version
1.5.2) and VirtualBox (https://www.virtualbox.org/, tested on version
4.3.10) installed, `vagrant up` will make a virtual machine suitable
for developing Xapian:

 * Ubuntu 13.04 with all packages needed to build Xapian and its
   documentation

 * eatmydata (to speed up test runs) and valgrind (for debugging
   memory allocations) both also installed

 * source code from this checkout in /vagrant; edit it on your host
   operating system and changes are reflected in the VM. The source
   tree is bootstrapped automatically (ensuring that the right
   versions of the build tools are available on the VM)

 * build tree in /home/vagrant/build, configured to install into
   /home/vagrant/install, with maintainer mode and documentation
   both enabled

Setting up can take a long time, as it downloads a minimal base box
and then installs all the required packages; once this is done you
don't have to wait so long if you need to reprovision the VM. (Once
Ubuntu 14.04 is released the plan is to build our own base box with
these packages already installed, which should make the process much
faster.)

`vagrant ssh` will log you into the VM, and you can type `cd build &&
make` to build Xapian. `make check` will run the tests.

(As noted above, in maintainer mode most changes that require
reconfiguration will happen automatically. If you need to do it by
hand you can either run the configure command yourself, or you can run
`vagrant provision`, which also checks for any system package
updates.)

The VM has a single 64 bit virtual processor, with 384M of memory; it
takes about 8G of disk space once up and running.

Use of C++ Features
===================

* As of Xapian 1.3.3, a compiler with decent support for C++11 is required to
  build Xapian.  We currently aim to allow users to use a non-C++11 compiler
  to build code which uses Xapian.

  There are now several compilers with good C++11 support, but there are a
  few shortfalls in commonly deployed versions of most of them.  Often we can
  work around this, and we should do where the effort is low compared to the
  gain (so a compiler version which is widely used is more worth supporting
  than one which is hardly used by anyone).

  However, we shouldn't have to jump through hoops to cater for compilers where
  their authors aren't putting in the effort to keep up with the language
  standards.

  Please avoid the following C++11 features for the time being:

  * ``std::to_string()`` - this is completely missing on current versions of
    mingw and cygwin - in the library, you can ``#include "str.h"`` and then
    use the ``str()`` function instead for most cases.  This is also usually
    faster than ``std::to_string()``.

* C++ features we currently assume:

  * We assume <sstream> is available.  GCC < 2.95.3 didn't have it but GCC
    2.95.3 includes a backported version.  We aren't aware of any other
    compilers still in use which lack it.

  * Non-".h" versions of standard ISO C++ headers (e.g. ``#include <list>``
    rather than ``#include <list.h>``).  We aren't aware of any compiler still
    in use which lacks these, and GCC 4.3 no longer has the old versions.  If
    there are any, we could add a directory full of forwarding headers to work
    around this issue.

  * Standard header ``<limits>`` (for ``numeric_limits<>``) - for GCC, this was
    added in GCC 3.0.

  * Standard header ``<streambuf>`` (GCC < 3.0 only has ``<streambuf.h>``).

  * Working auto_ptr in header ``<memory>`` (some old version of some compiler
    had a buggy implementation - the details are lost to history, but it may
    have been GCC 2.95, or perhaps EGCS).

* RTTI (dynamic_cast<>, typeid, etc):  Needing to use RTTI features in the
  library most likely indicates a design flaw, and you should avoid use
  of these features.  Where necessary, you can use a technique similar to
  Database::as_networkdatabase() to replace dynamic_cast<>.

* Exceptions: In hindsight, throwing exceptions in the library seems to have
  been a poor design decision.  GCC on Solaris can't cope with exceptions in
  shared libraries (though it appears this may have been fixed in more recent
  versions), and we've also had test failures on other platforms which only
  occur with shared libraries - possibly with a similar cause.  Exceptions can
  also be a pain to handle elegantly in the bindings.  We intend to investigate
  modifying the library to return error codes internally, and then offering the
  user the choice of exception throwing or error code returning API methods
  (with the exception being thrown by an inlined wrapper in the externally
  visible header files).  With this in mind, please don't complicate the
  internal handling of exceptions...

* "using namespace std;" and "using std::XXX;" - it's OK to use these in
  applications, library code, and internal library headers.  But in externally
  visible headers (such as anything included by "#include <xapian.h>") you MUST
  use explicit "std::" qualifiers - it's not acceptable to pull anything from
  namespace std into the namespace of an application which uses Xapian.

* Use C++ style casts (static_cast<>, reinterpret_cast<>, and const_cast<>)
  or constructor-syntax (e.g. ``double(value)``) in preference to C style
  casts.  The syntax of the C++ casts is ugly, but they do make the intent much
  clearer which is definitely a good thing.

* std::pair<> with an STL class as one (or both) of the members can produce
  very long symbols (over 4KB!) after name mangling - long enough to overflow
  the size limits of some vendor compilers or toolchains (so this can affect
  GCC if it is using the system ld or as).  Even where the compiler works, the
  symbol bloat in an unstripped build is probably best avoided, so it's
  preferable to use a simple two member struct instead.  The code is probably
  more readable anyway, and easier to extend if more members are needed later.

* We try to avoid putting the full definition of virtual methods in header
  files.  This is because current compilers can't (as far as we know) inline
  virtual methods, so putting the definition in the header file simply slows
  down compilation (and, because method definitions often require further
  header files to be included, this can result in many more files needing
  recompilation after a change to a header file than is really necessary).
  Just put the declaration in the header file, and put the definition in a .cc
  file with the same basename.

Include ordering for source files
---------------------------------

To help us move towards a consistent ordering of #include lines in source
files, please follow the following policy when ordering them:

* #include <config.h> should be first, and use <> not "" (as recommended by the
  autoconf manual).  Always include config.h from C/C++ source files, but don't
  include it from header files - the autoconf manual recommends that it should
  be included first, so including it from headers is either redundant, or may
  hide a missing config.h include in the source file the header was included
  from (better to get an error in this case).

* The header corresponding to the source file should be next. This means that
  compilation of the library ensures that each header with a corresponding
  source file is "self supporting" (i.e. it implicitly or explicitly includes
  all of the headers it requires).

* External xapian-core headers, alphabetically. When included from other
  external headers, use <> to reduce problems with finding headers in the
  user's source tree by mistake. In sources and internal headers, use "" (?) -
  practically this makes no difference as we have -I for srcdir and builddir,
  but <> suggests installed header files so "" seems more natural).

* Internal headers, alphabetically (using "").

* "Safe" versions of library headers (include these first to avoid issues if
  other library headers include the ones we want to wrap). Use "" and order
  alphabetically.

* Library headers, alphabetically.

* Standard C++ headers, alphabetically. Use the modern (no .h suffix) names.

C++ Portability Issues
======================

Web Resources
-------------

The "C++ FAQ Lite" covers many frequently asked C++ questions:
http://www.parashift.com/c++-faq-lite/

Header Portability Issues
-------------------------

<fcntl.h>:
----------

Don't directly '#include <fcntl.h>' - instead '#include "safefcntl.h"'.

The main reason for this is that when using certain compilers on certain
versions of Solaris, fcntl.h does '#define open open64'.  Sadly this breaks C++
code which has methods called open (as we do).  There's a cunning workaround
for this problem in common/safefcntl.h.

Also, safefcntl.h ensures the O_BINARY is defined (to 0 if not required) so
calls to open() and creat() can specify O_BINARY unconditionally for the
benefit of platforms which discriminate between text and binary files.

<windows.h>:
------------

Don't directly '#include <windows.h>' - instead '#include "safewindows.h"'
which reduces the bloat of header files included and prevents some of the
more egregious namespace pollution.  It also defines any constants we need
which might be missing in older versions of the mingw headers.

<winsock2.h>:
-------------

Don't directly '#include <winsock2.h>' - instead '#include "safewinsock2.h"'.
This ensure that safewindows.h is included before <winsock2.h> to avoid
winsock2.h including windows.h without our namespace pollution reducing
workarounds.

<errno.h>:
----------

Don't directly '#include <errno.h>' - instead '#include "safeerrno.h"' which
works around a problem with Compaq's C++ compiler.

<sys/select.h>:
---------------

Don't directly '#include <sys/select.h>' - instead '#include "safesysselect.h"'
which supports older UNIX platforms which predate POSIX 1003.1-2001 and works
around a problem on Solaris.

<sys/socket.h>:
---------------

Don't directly '#include <sys/socket.h>' - instead '#include "safesyssocket.h"'
which supports older UNIX platforms which predate POSIX 1003.1-2001 and works
on Windows too.

<sys/stat.h>:
-------------

Don't directly '#include <sys/stat.h>' - instead '#include "safesysstat.h"'
which under MSVC enables stat to work on files > 2GB, defines the missing
POSIX macros S_ISDIR and S_ISREG, pulls in <direct.h> for mkdir() (which is
provided by sys/stat.h under UNIX) and provides a compatibility wrapper for
mkdir() which takes 2 arguments (so code using mkdir can always just pass
two arguments).

<sys/wait.h>:
-------------

To get `WEXITSTATUS` or `WIFEXITED` defined, '#include "safesyswait.h"'.
Note that this won't provide `waitpid()`, etc on Microsoft Windows, since
these functions are only really useful to use when `fork()` is available.

<unistd.h>:
-----------

Don't directly '#include <unistd.h>' - instead '#include "safeunistd.h"'
- MSVC doesn't even HAVE unistd.h!

The various "safe" headers are maintained in xapian-core/common, but also used
by Omega.  Currently bootstrap sorts out setting up a copy of this subdirectory
via a secondary git checkout.

Warning-Free Compilation
------------------------

Compiling without warnings on every platform is our goal, though it's not
always possible to achieve.  For example, some GCC 3.x compilers produce the
occasional bogus warning (e.g.  warning that a variable may be used
uninitialised, despite it being initialised at the point of declaration!)

You should consider configure-ing with:

./configure CXXFLAGS=-Werror

when doing development work on Xapian.  This promotes warnings to errors,
which should ensure you at least don't introduce new warnings for the compiler
you're using.

If you configure with --enable-maintainer-mode, and are using GCC 4.1 or newer,
this is done for you automatically.  This is intended to be an aid rather than
a form of automated punishment - it's all too easy to miss a new warning as
once a file is compiled, you don't see it unless you modify that file or one of
its dependencies.

With Intel's C++ compiler, --enable-maintainer-mode also enables -Werror.
If you know the equivalent of -Werror for other compilers, please add a note
here, or tell us so that we can add a note.

Miscellaneous Portability Issues
--------------------------------

Make sure that the last line of any source file ends with a linefeed character
since it's undefined behaviour if it doesn't (most compilers accept it, though
at least GCC gives a warning).

Branch Prediction Hints
=======================

For compilers which support ``__builtin_expect()`` (GCC >= 3.0 and some others)
you can provide manual hints to assist branch prediction.  We've wrapped these
in macros which evaluate to just their argument for compilers which don't
support ``__builtin_expect()__``.

Within the xapian-core library code, you can mark the expressions in ``if`` and
``while`` statements as ``rare`` (if the condition is rarely true) or ``usual``
(if the condition is usually true).

For example::

    if (rare(something_unusual())) deal_with_it();

    while (usual(!end_condition()) keep_going();

It's easy to make incorrect assumptions about where hotspots are and which
branches are usually taken or not, so except for really obvious cases (such
as ``if (!consistency_check()) throw_exception();``) you should benchmark
that new ``rare`` and ``usual`` hints help rather than hinder before committing
them to the repository.  It's also likely to be a waste of effort to add them
outside of areas of code which are executed very frequently.

Don't expect miracles - the first 15 uses added saved approximately 1%.

If you know how to implement the ``rare`` and ``usual`` macros for other
compilers, please let us know.

Configure Options
=================

Especially for a library, compile-time options aren't a good solution for
how to integrate a new feature.  An increasingly large number of users install
pre-built binary packages rather than building from source, and unless the
package is capable of being split into modules, the packager has to choose a
set of compile-time options to use.  And they'll tend to choose either the
standard ones, or perhaps a broader set to try to keep everyone happy.  For a
library, similar issues occur when installing from source as well - the
sysadmin must choose the options which will keep all users happy.

Another problem with compile-time options is that it's hard to ensure that
a change doesn't break compilation under some combination of options without
actually building and running the test-suite on all combinations.  The fewer
compile-time options, the more likely the code will compile with every
combination of them.

So please think carefully before adding more compile-time options.  They're
probably OK for experimental features (but should go away once a feature is no
longer experimental).  Options to instrument a build for special purposes
(debug, profiling, etc) are also acceptable.  Disabling whole features probably
isn't (e.g. the --disable-backend-XXX options we already have are dubious,
though being able to disable the remote backend can be useful when trying to
get Xapian going on a platform).

Makefile Portability
====================

We don't want to force those building Xapian from the source distribution to
have to use GNU make.  Requiring GNU make for "make dist" isn't such a problem
but it's probably better to use portable constructs everywhere to avoid
problems when people move or copy code between targets.  If you do make use
of non-portable constructs where it's OK, add a comment noting the special
circumstances which justify doing so.

Here's an incomplete list of things to avoid:

* Don't use "$(RM)" - it's defined by GNU make, but using it actually harms
  portability as other makes don't define it.  Use plain "rm" instead.

* Don't use "%" pattern rules - these are GNU make specific.  Use an
  implicit rule (e.g. ".c.o:") if you can.  Otherwise, write out each version
  explicitly.

* Don't use "$<" except in implicit rules.  This is an annoying restriction,
  as using "$<" makes it much easier to make VPATH builds work.  But it's only
  portable in implicit rules.  Tips for rewriting - if it's a source file,
  write it as::

    $(srcdir)/foo.ext

  If it's a generated object file or similar, just write the name as is.  The
  tricky case is a generated file which isn't in git but is shipped in the
  distribution tarball, as such a file could be in either the source or build
  tree.  Use this trick to make sure it's found whichever directory it's in::

    `test -f foo.ext || echo '$(srcdir)/'`foo.ext

* Don't use "exit 0" to make a rule fail.  Use "false" instead.  BSD make
  doesn't like "exit 0" in a rule.

* Don't use make conditionals.  Automake offers conditionals which may be
  of use, and these are implemented to work with any make.  See the automake
  manual for details, and a few caveats.

* The list of portable utilities is:

    cat cmp cp diff echo egrep expr false grep install-info
    ln ls mkdir mv pwd rm rmdir sed sleep sort tar test touch true

  Note that versions of these (GNU versions in particular) support switches
  which aren't portable - notably, "test -r" isn't portable; neither is
  "cp -a".  And note that "mkdir -p" isn't portable - the semantics vary.
  The autoconf manual has some useful information about writing portable
  shell code (most of it not specific to autoconf)::

    http://www.gnu.org/software/autoconf/manual/autoconf.html#Portable-Shell

* Don't use "include" - it's not present in BSD make (at least some versions
  have ".include" instead, but that doesn't really seem to help...)  Automake
  provides a configure-time include, which may provide a replacement for some
  uses of "include".

* It appears that BSD make only supports VPATH for implicit rules (e.g.
  ".c.o:") - there's certainly a restriction there which is not present in GNU
  make.  We used to try to work around this, but now we use AM_MAINTAINER_MODE
  to disable rules which are only needed by those developing Xapian (these were
  the rules which caused problems).  And we recommend those developing Xapian
  use GNU make to avoid problems.

* Rules with multiple targets can cause problems for parallel builds.  These
  rules are really just a shorthand for multiple rules with the same
  prerequisites and commands, and it is fine to use them in this way.  However,
  a common temptation is to use them when a single invocation of a command
  generates multiple output files, by adding each of the output files as a
  target.  Eg, if a swig language module generates xapian_wrap.cc and
  xapian_wrap.h, it is tempting to add a single rule something like::

    # This rule has a problem
    xapian_wrap.cc xapian_wrap.h: xapian.i
            SWIG_commands

  This can result in SWIG_commands being run twice, in parallel.  If
  SWIG_commands generates any temporary files, the two invocations can
  interfere causing one of them to fail.

  Instead of this rule, one solution is to pick one of the output files as a
  primary target, and add a dependency for the second output file on the first
  output file::

    # This rule also has a problem
    xapian_wrap.h: xapian_wrap.cc
    xapian_wrap.cc: xapian.i
            SWIG_commands

  This ensures that make knows that only one invocation of SWIG_commands is
  necessary, but could result in problems if the invocation of SWIG_commands
  failed after creating xapian_wrap.cc, but before creating xapian_wrap.h.
  Instead, we recommend creating an intermediate target::
  
    # This rule works in most cases
    xapian_wrap.cc xapian_wrap.h: xapian_wrap.stamp
    xapian_wrap.stamp: xapian.i
            SWIG_commands
            touch $@

  Because the intermediate target is only touched after the commands have
  executed successfully, subsequent builds will always retry the commands if an
  error occurs.  Note that the intermediate target cannot be a "phony" target
  because this would result in the commands being re-run for every build.

  However, this rule still has a problem - if the xapian_wrap.cc and
  xapian_wrap.h files are removed, but the xapian_wrap.stamp file is not, the
  .cc and .h files will not be regenerated.   There is no simple solution to
  this, but the following is a recipe taken from the automake manual which
  works.  For details of *why* it works, see the section in the automake manual
  titled "Multiple Outputs"::

    # This rule works even if some of the output files were removed
    xapian_wrap.cc xapian_wrap.h: xapian_wrap.stamp
    ## Recover from the removal of $@.  A full explanation of these rules is in
    ## the automake manual under the heading "Multiple Outputs".
            @if test -f $@; then :; else \
              trap 'rm -rf xapian_wrap.lock xapian_wrap.stamp' 1 2 13 15; \
              if mkdir xapian_wrap.lock 2>/dev/null; then \
                rm -f xapian_wrap.stamp; \
                $(MAKE) $(AM_MAKEFLAGS) xapian_wrap.stamp; \
                rmdir xapian_wrap.lock; \
              else \
                while test -d xapian_wrap.lock; do sleep 1; done; \
                test -f xapian_wrap.stamp; exit $$?; \
              fi; \
            fi
    xapian_wrap.stamp: xapian.i
            SWIG_commands
            touch $@

* This is actually a robustness point, not portability per se.  Rules which
  generate files should be careful not to leave a partial file in place if
  there's an error as it will have a timestamp which leads make to believe it's
  up-to-date.  So this is bad:

  foo.cc: script.pl
	$PERL script.pl > foo.cc

  This is better:

  foo.cc: script.pl
	$PERL script.pl > foo.tmp
	mv foo.tmp foo.cc

  Alternatively, pass the output filename to the script and make sure you
  delete the output on error or a signal (although this approach can leave
  a partial file in place if the power fails).  All used Makefile.am-s and
  scripts have been checked (and fixed if required) as of 2003-07-10 (didn't
  check xapian-bindings).

* Another robustness point - if you add a non-file target to a makefile, you
  should also list it in ".PHONY".  Otherwise your target won't get remade
  reliably if someone creates a file with the same name in their tree.  For
  example:

  .PHONY: hello goodbye

  hello:
        echo hello

  goodbye:
        echo goodbye

And lastly a style point - using "@" to suppress echoing of commands being
executed removes choice from the user - they may want to see what commands
are being executed.  And if they don't want to, many versions of make support
the use "make -s" to suppress the echoing of commands.

Using @echo on a message sent to stdout or stderr is acceptable (since it
avoids showing the message twice).  Otherwise don't use "@" - it makes it
harder to track down problems in the makefiles.

Naming of Scripts
=================

Scripts generally should *not* have an extension indicating the language they
are currently implemented in (e.g. ``runtest`` rather than ``runtest.sh`` or
``runtest.pl``).  The problem with such an extension is that if we decide
to reimplement the script in a different language, we either have to rename
the script (which is annoying as people will be used to the name, and may
have embedded it in their own scripts), or we have a script with a confusing
name (e.g. a Python script with extension ``.pl``).

The above reasoning doesn't apply to scripts which have to be in a particular
language for some reason, though for consistency they probably shouldn't get
an extension either, unless there's a good reason to have one.

Use of Assert
=============

Use Assert to perform internal consistency checks, and to check for invalid
arguments to functions and methods (e.g. passing a NULL pointer when this isn't
permitted).  It should *NOT* be used to check for error conditions such as
file read errors, memory allocation failing, etc (since we want to perform such
checks in non-debug builds too).

File format errors should also not be tested with Assert - we want to catch
a corrupted database or a malformed input file in a non-debug build too.

There are several variants of Assert:

- Assert(P) -- asserts that expression P is true.

- AssertRel(a,rel,b) -- asserts that (a rel b) is true - rel can be a boolean
  relational operator, i.e. one of ``==``, ``!=``, ``>``, ``>=``, ``<``,
  ``<=``.  The message given if the assertion fails reports the values of
  a and b, so ``AssertRel(a,<,b);`` is more helpful than ``Assert(a < b);``

- AssertEq(a,b) -- shorthand for AssertRel(a,==,b).

- AssertEqDouble(a,b) -- asserts a and b differ by less than DBL_EPSILON

- AssertParanoid(P) -- a particularly expensive assertion.  If you want a build
  with Asserts enabled, but without a great performance overhead, then
  passing --enable-assertions=partial to configure and AssertParanoids
  won't be checked, but Asserts will.  You can also use AssertRelParanoid
  and AssertEqParanoid.

- CompileTimeAssert(P) -- this has now been removed, since we require C++11
  support from the compuiler, and C++11 added ``static_assert``.

Marking Features as Deprecated
==============================

In the API headers, a feature (a class, method, function, enum, typedef, etc)
can be marked as deprecated by using the XAPIAN_DEPRECATED() or
XAPIAN_DEPRECATED_CLASS macros.  Note that you can't deprecate a preprocessor
macro.

For compilers with a suitable mechanism (currently GCC 3.1 or later, and
MSVC 7.0 or later) this causes compile-time warning messages to be emitted for
any use of the deprecated feature.  For compilers without support, the macro
just expands to its argument.

Sometimes a deprecated feature will also be removed from the library itself
(particularly something like a typedef), but if the feature is still used
inside the library (for example, so we can define class methods), then use
XAPIAN_DEPRECATED_EX() or XAPIAN_DEPRECATED_CLASS_EX instead, which will only
issue a warning in user code (this relies on user code including xapian.h
and library code including individual headers)

You must add this line to any API header which uses XAPIAN_DEPRECATED() or
XAPIAN_DEPRECATED_CLASS::

    #include <xapian/deprecated.h>

When marking a feature as deprecated, document the deprecation in
docs/deprecation.rst.  When actually removing deprecated features, please tidy
up by removing the inclusion of <xapian/deprecated.h> from any file which no
longer marks any features as deprecated.

The XAPIAN_DEPRECATED() macro should wrap the whole declaration except for the
semicolon and any "definition" part, for example::

    XAPIAN_DEPRECATED(int old_function(double arg));

    class Foo {
      public:
        XAPIAN_DEPRECATED(int old_method());

        XAPIAN_DEPRECATED(int old_const_method() const);

        XAPIAN_DEPRECATED(virtual int old_virt_method()) = 0;

        XAPIAN_DEPRECATED(static int old_static_method());

        XAPIAN_DEPRECATED(static const int OLD_CONSTANT) = 42;
    };

Mark a class as deprecated by inserting ``XAPIAN_DEPRECATED_CLASS`` after the
class keyword like so::

    class XAPIAN_DEPRECATED_CLASS Foo {
      public:
        Foo() { }

        // ...
    };

With recent versions of GCC (4.4.7 allows this, 3.3.5 doesn't), you can
simply mark a method defined inline in a class with ``XAPIAN_DEPRECATED()``
like so:
    
    class Foo {
      public:
        // This fails to compile with GCC 3.3.5, so don't do this!
        XAPIAN_DEPRECATED(int old_inline_method()) { return 42; }
    };
    
Xapian 1.3.x and later require at least GCC 4.6, so you can just use the
approach above.  Xapian 1.2.x aims to support GCC 3.1 and later, so in the
unlikely event of needing to adjust deprecation markers in 1.2.x, you need to
rewrite the above like so:

    class Foo {
      public:
        XAPIAN_DEPRECATED(int old_inline_method());
    };

    inline int Foo::old_inline_method() { return 42; }

Submitting Patches:
===================

If you have a patch to fix a problem in Xapian, or to add a new feature,
please send it to us for inclusion.  Any major changes should be discussed
on the xapian-devel mailing list first:
<http://xapian.org/lists>

Also, please read the following section on licensing of patches before
submitting a patch.

We find patches in unified diff format easiest to read.  If you're using
git, then "git diff" is good (or "git format-patch" for a patch series).  If
you're working from a tarball, you can unpack a second clean copy of the files
and compare the two versions with "diff -pruN" (-p reports the function name
for each chunk, -r acts recursively, -u does a unified diff, and -N shows
new files in the diff).  Alternatively "ptardiff" (which comes with perl, at
least on Debian and Ubuntu) can diff against the original tarball, unpacking
it on the fly.

Please set the width of a tab character in your editor to 8 spaces, and use
Unix line endings (i.e. LF, not CR+LF).  Failing to do so will make it much
harder for us to merge in your changes.

We don't currently have a formal coding standards document, but please try
to follow the style of the existing code.  In particular:

* Indent C++ code by 4 spaces for a new indentation level, and set your editor
  to tab-fill indentation (with a tab being 8 spaces wide).

  As an exception, "public", "protected" and "private" declarations in classes
  and structs should be indented by 2 spaces, and the following code should be
  indented by 2 more spaces::

    class Foo {
      public:
        method();
    };

  The rationale for this exception is that class definitions in header files
  often have fairly long lines, so losing an indent level to the access
  specifier tends to make class definitions less readable.

  The default access for a class is always "private", so there's no need
  to specify that explicitly - in other words, write this::
  
    class Foo {
        int internal_method();

      public:
        int external_method();
    };

  Don't write this::

    class Foo {
      private:
        int internal_method();

      public:
        int external_method();
    };

  If a class only contains public methods and data, consider declaring it as a
  "struct" (the only difference in C++ is that the default access for a
  struct is "public").

* Put a space before the "(" after control flow constructs like "for", "if",
  "while", etc.  Don't put a space before the "(" in function calls.  So
  write "if (strlen(p) > 10)" not "if(strlen (p) > 10)".

* When "if", "else", "for", "while", "do," "switch", "case", "default", "try",
  or "catch" is followed by a block enclosed in braces, the opening brace
  should be on the same line, like so::

    if (x > 12) {
        foo(x);
        x = 12;
    } else {
        bar(x);
    }

  The rationale for this is that it conserves vertical space (allowing more
  code to fit on screen) without reducing readability.

* If you have an empty loop body, use `{ }` rather than `;` as the former
  stands out more clearly to the reader (but also consider if the code might be
  clearer written a different way).

* Prefer "++i;" to "i++;", "i += 1;", or "i = i + 1".  For simple integer
  variables these should generate equivalent (if not identical) code, but if i
  is an iterator object then the pre-increment form can be more efficient in
  some cases with some compilers.  It's simpler and more consistent to always
  use the pre-increment form (unless you make use of the old value which the
  post-increment form returns).  For the same reasons, prefer "--i;" to "i--;",
  "i -= 1;", or "i = i - 1;".

* Prefer "container.empty()" to "container.size() == 0" (and
  "!container.empty()" to "container.size() != 0" or "container.size() > 0").
  Finding the size of a container may not be a constant time operation for
  all containers (e.g. std::list may not be, and indeed isn't for GCC - see
  https://gcc.gnu.org/onlinedocs/libstdc++/manual/containers.html#sequences.list.size).
  Also the "empty()" form makes the intent of the test more explicit.

* Prefer not to use "else" when the control flow is diverted elsewhere at the
  end of the "if" block (e.g. by "return", "continue", "break", "throw").  This
  eliminates a level of indentation from the code in the "else" block, and
  typically makes the control flow logic clearer.  For example::

    if (x == 0) {
        foo();
        return;
    }

    while (x--) {
        bar();
    }

  rather than::

    if (x == 0) {
        foo();
        return;
    } else {
        while (x--) {
            bar();
        }
    }

* For standard ISO C headers, prefer the C++ form for ISO C headers (e.g.
  "#include <cstdlib>" rather than "#include <stdlib.h>") unless there's a good
  reason (e.g. portability) to do otherwise.  Be sure to document such
  exceptions to avoid another developer changing them to the standard form.
  Global exceptions: <signal.h> (lots of POSIX stuff which e.g. Sun's compiler
  doesn't provide in <csignal>).

* For standard ISO C++ headers, *always* use the ISO C++ form '#include <list>'
  (pre-ISO compilers used '#include <list.h>', but GCC has generated a warning
  for this form for years, and GCC 4.3 dropped support entirely).

* Some guidelines for efficient use of std::string:

  + When passing an empty string to a method expecting ``const std::string &``
    prefer ``std::string()`` to ``""`` or ``std::string("")`` as the first form
    is more likely to directly use a special "empty string representation" (it
    does with GCC at least).

  + To make a string object empty, ``s.resize(0)`` (if you want to keep the
    current reserved space) or ``s = string()`` (if you don't) seem the best
    options.

  + Use ``std::string::assign()`` rather than building a temporary string
    object and assigning that.  For example, ``foo = std::string(ptr, len);``
    is better written as ``foo.assign(ptr, len);``.

  + It's generally better to build up strings using ``+=`` rather than
    combining series of components with ``+``.  So ``foo = a + " and " + c`` is
    better written as ``foo = a; foo += " and "; foo += c;``.  It's possible
    for compilers to handle the former without a lot of temporary string
    objects by returning a proxy object to allow the concatenation to happen
    lazily, but not all compilers do this, and it's likely to still have some
    overhead.  Note that GCC 4.1 seems to produce larger code in some cases for
    the latter approach, but it's a definite win with GCC 4.4.

  * ``std::string(1, '\0')`` seems to be slightly more efficient than
    ``std::string("", 1)`` for constructing a std::string containing a single
    ASCII nul character.

* Prefer ``new SomeClass`` to ``new SomeClass()``, since the latter tends to
  lead one to write ``SomeClass foo();` which is a function prototype, and not
  equivalent to the variable definition ``SomeClass foo``.  However, note that
  ``new SomePODType()`` is *not* the same as ``new SomePODType`` (if
  SomePODType is a POD (Plain Old Data) type) - the former will zero-initialise
  scalar members of SomePODType.

* When catching an exception which is an object, do it by const reference, so
  like this::

      try {
	  foo();
      } catch (const ErrorClass &e) {
	  bar(e);
      }

  Catching by value is bad because it "slices" the object if an object of a
  derived type is thrown.  Even if derived types aren't a worry, it also causes
  the copy constructor to be called needlessly.

  See also: http://www.parashift.com/c++-faq-lite/exceptions.html#faq-17.7

  A const reference is preferable to a non-const reference as it stops the
  object being inadvertently modified.  In the rare cases when you want to
  modify the caught object, a non-const reference is OK.

We will do our best to give credit where credit is due - if we have used
patches from you, or received helpful reports or advice, we will add your name
to the AUTHORS file (unless you specifically request us not to).  If you see we
have forgotten to do this, please draw it to our attention so that we can
address the omission.

Licensing of patches
====================

If you want a patch to be considered for inclusion in the Xapian sources, you
must own the copyright on this patch.  Employers often claim copyright on code
written by their employees (even if the code is written in their spare time),
so please check with your employer if this applies.  Be aware that even if you
are a student your university may try and claim some rights on code which you
write.

Patches which are submitted to Xapian will only be included if the copyright
holder(s) dual-license them under each of the following licences:

 - GPL version 2 and all later versions (see the file "COPYING" for details).
 - MIT/X license::

 Copyright (c) <year> <copyright holders>

 Permission is hereby granted, free of charge, to any person obtaining a copy
 of this software and associated documentation files (the "Software"), to
 deal in the Software without restriction, including without limitation the
 rights to use, copy, modify, merge, publish, distribute, sublicense, and/or
 sell copies of the Software, and to permit persons to whom the Software is
 furnished to do so, subject to the following conditions:

 The above copyright notice and this permission notice shall be included in
 all copies or substantial portions of the Software.

 THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
 FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
 IN THE SOFTWARE.

The current distribution of Xapian contains many files which are only licensed
under the GPL, but we are working towards being able to distribute Xapian under
a more permissive license, and are not willing to accept patches which we will
have to rewrite before this can happen.

Tips for Submitting a Good Patch
================================

1) Make sure that the documentation is updated
----------------------------------------------

 * API classes, methods, functions, and types must be documented by
   documentation comments alongside the declaration in ``include/xapian/*.h``.
   These are collated by doxygen - see doxygen's documentation for details
   of the supported syntax.  We've decided to prefer to use @ rather than \
   to introduce doxygen commands (the choice is essentially arbitrary, but
   \ introduces C/C++ escape sequences so @ is likely to make for easier to
   read mark up for C/C++ coders).

 * The documentation comments don't give users a good overview, so we also
   need documentation which gives a good overview of how to achieve particular
   tasks.  In particularly, major new functionality should have its own "topic"
   document, or extend an existing topic document if more appropriate.

 * Internal classes, etc should also be documented by documentation comments
   where they are declared.

2) Make sure the tests are right
--------------------------------

 * If you're adding a feature, also add feature tests for it.  These both
   ensure that the feature isn't broken to start with and detect if later
   changes stop it working as intended.

 * If you've fixed a bug, make sure there's a regression test which
   fails on the existing code and succeeds after your changes.

 * Make sure all existing tests continue to pass.

If you don't know how to write tests using the Xapian test rig, then
ask.  It's reasonably simple once you've done it once.  There is a brief
introduction to the Xapian test system in ``docs/tests.html``.

3) Make sure the attributions are right
---------------------------------------

 * If necessary, modify the copyright statement at the top of any
   files you've altered. If there is no copyright statement, you may
   add one (there are a couple of Makefile.am's and similar that don't
   have copyright statements; anything that small doesn't really need
   one anyway, so it's a judgement call).  If you've added files which
   you've written from scratch, they should include the GPL boilerplate
   with your name only.

 * If you're not in there, add yourself to the AUTHORS file.

4) Commit
---------

 * Commit:
  
   + If there's a trac ticket or other reference for the bug, mention it in the
     commit message - it's a great help to future developers trying to work out
     why a change was made.

5) Consider backporting
-----------------------

 * If there's an active release branch, check if the bug is present in that
   branch, and if the fix is appropriate to backport - if the fix breaks ABI
   compatibility or is very invasive, you need to fix it in a different way
   for the release branch, or decide not to backport the fix.

6) Update trac
--------------

 * If there's a related trac ticket, update it (if the issue is completely
   addressed by the changes you've made, then close it).

 * Update the release notes for the most recent release with a copy of the
   patch.  If the commit from git applies cleanly, you can just link to
   it.  If it fails to apply, please attach an adjusted patch which does.
   If there are conflicts in test cases which aren't easy to resolve, it is
   acceptable to just drop those changes from the patch if we can still be
   confident that the issue is actually fixed by the patch.
 
API Structure Notes
===================

We use reference counted pointers for most API classes.  These are implemented
using Xapian::Internal::intrusive_ptr, the implementation of which is exposed
for efficiency, and because it's unlikely we'll need to change it frequently,
if at all.

For the reference counted classes, the API class (e.g. Xapian::Enquire) is
really just a wrapper around a reference counted pointer.  This points to an
internal class (e.g. Xapian::Enquire::Internal).  The reference counted
pointer is a member variable of the API class called internal.  Conceptually
this member is private, though it typically isn't declared as private (this
is to avoid littering the external headers with friend declarations for
non-API classes).

There are a few exceptions to the reference counted structure, such as
MSetIterator and ESetIterator which have an exposed implementation.  Tests show
this makes a substantial difference to speed (it's ~20% faster) in typical
cases of iterator use.

The postfix operator++ for iterators should be implemented inline in terms
of the prefix form as described by Joe Buck on the gcc mailing list
- excerpt from http://article.gmane.org/gmane.comp.gcc.devel:50201 ::

	class some_iterator {
	public:
	    // ...
	    some_iterator& operator++();

	    some_iterator operator++(int) {
		some_iterator tmp = *this;
		operator++();
		return tmp;
	    }
	};

    The compiler is allowed to assume that the copy constructor only does
    a copy, and to optimize away unneeded copy operations.  The result
    in this case should be that, for some_iterator above, using the
    postfix operator without using the result should give code equivalent
    to using the prefix operator.

    Now, for [GCC 3.4], you'll find that the dead uses of tmp are only
    completely optimized away if tmp has only one data member that can fit in a
    register.  [GCC 4.0 will do] better, and you should find that this style
    comes very close to eliminating any penalty from "incorrect" use of the
    postfix form.

Xapian's PostingIterator, TermIterator, PositionIterator, and ValueIterator all
have only one data member which fits in a register.

Handy tips for aiding development
=================================

If you are find you are repeatedly changing the API headers (in include/)
during development, then you may become annoyed that the docs/ subdirectory
will rebuild the doxygen documentation every time you run "make" since this
takes a while.  You can disable this temporarily (if you're using GNU make),
by creating a file "docs/GNUmakefile" containing these two lines::

%:
	@echo "Skipping 'make $@' in docs"

Note that the whitespace at the start of the second line needs to be a
single "tab" character!

Don't forget to remove (or rename) this and check the documentation builds
before committing or generating a patch though!

If you are using an editor or other tool capable of running syntax checks as you
work there you can use the `make` target 'check-syntax'. For 'emacs' users this
works well with 'flymake'. Usage from a shell::

    make check-syntax check_sources=api/omdatabase.cc


How to make a release
=====================

This is a (hopefully complete) list of the jobs which need doing:

* Email Fabrice Colin and Tim Brody so they can check RPM packaging.

* Check if `config/config.guess` and `config/config.sub` need updating to
  more recent versions from http://git.savannah.gnu.org/gitweb/?p=config.git

* Check the revision currently specified in the bootstrap for the common
  subdirectory.  Unless there's a good reason, we should release
  xapian-core and omega with synchronised versions of the shared files.

* Make sure that any new/changed/removed API methods in xapian-core have been
  wrapped/updated/removed in xapian-bindings.

* Update the lists of deprecated/removed API methods in docs/deprecation.rst

* Update the NEWS files using information from the ChangeLog files

* Update the version in configure.ac for each module (xapian-core, omega, and
  xapian-bindings), and the library version info in xapian-core's configure.ac

* Make sure the submitters of fixed bugs are mentioned in the "thanks" list in
  xapian-core/AUTHORS.  Check the list for the appropriate milestone::

   http://trac.xapian.org/query?col=id&col=summary&col=reporter&milestone=1.0.14

* On atreus, tag the source trees for the new revision - use the
  git-tag-release script, running it with the new version number, for example:

  xapian-maintainer-tools/git-tag-release 1.0.14

  This script also generates tarballs for the new release and copies them
  across to the website.

* Add the new version to the list of versions in trac:
  http://trac.xapian.org/admin/ticket/versions

* Add a new milestone for the version after this one:
  http://trac.xapian.org/admin/ticket/milestones

* Mark the current milestone as completed.  In order to do so, any unfixed bugs
  with this milestone will need to be moved to another milestone (most likely
  the milestone you just added).

* Update the wiki:

  Create a new page http://wiki.xapian.org/ReleaseNotes/X.Y.Z and link it into
  http://wiki.xapian.org/ReleaseNotes in place of the old current release link,
  which should be moved to the archived section.

  Also update the roadmap at http://wiki.xapian.org/RoadMap by recording the
  date of this release and adding an entry for the next release with an
  estimated release date.

* Update the website: `generate` in the CVS module www.xapian.org contains the
  latest version and the date it was released.

* Run /home/olly/tmp/xapian-website-update/update_website.sh

* Announce the new version on xapian-discuss

* Have a nice cup of tea!

How to make Debian packages for a new release
=============================================

Debian control files are stored in separate git repositories:

* http://anonscm.debian.org/cgit/collab-maint/xapian-bindings.git
* http://anonscm.debian.org/cgit/collab-maint/xapian-core.git
* http://anonscm.debian.org/cgit/collab-maint/xapian-omega.git

To package a new upstream release, these should be updated as follows:

* If there are any patch files in "debian/patches", check if these have been
  incorporated into the new release, and if so remove them and update
  "debian/patches/series".

* Update the debian/changelog file, being sure to keep it in the
  standard Debian format (the easiest way is to use the dch utility
  like so: "dch -v 1.2.19-1".  The new version number should be the
  version number of the release followed by "-1" (i.e., a debian
  patch number of 1).  The changelog message should indicate that
  there is a new upstream release, and should mention any significant
  changes in the new release.

* Tag using: ``git tag -s -m 1.2.19-1 1.2.19-1``

* FIXME: Document how to make source packages, or update
  ``make-source-packages``.

* FIXME: Document how to build binary packages, or update ``build-packages``.

* Test the packages.

* Run ``debsign build/*_amd64.changes`` to GPG sign the packages.

* Run ``dput build/*_amd64.changes`` to upload them to Debian.

* For the Ubuntu backports::

   ./backport-source-packages xapian-core 1.2.19-1 ubuntu
   ./backport-source-packages xapian-omega 1.2.19-1 ubuntu
   ./backport-source-packages xapian-bindings 1.2.19-1 ubuntu

  And once libsearch-xapian-perl is uploaded to Debian unstable::

   ./backport-source-packages libsearch-xapian-perl 1.2.19.0-1 ubuntu

  Then sign::

   debsign build/*99*_source.changes

  Upload::

   dput xapian-backports build/xapian-core*99*_source.changes

  Wait for that to have a chance to build, and then::

   dput xapian-backports build/xapian-[bo]*99*_source.changes
   dput xapian-backports build/libsearch-xapian-perl*_source.changes

.. vim: syntax=rst