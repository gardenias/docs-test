.. _hgmozilla_installing:

====================
Installing Mercurial
====================

Having a modern Mercurial installed is important. Features, bug fixes,
and performance enhancements are always being added to Mercurial.
Staying up to date on releases is important to getting the most out of
your tools.

.. note::

   Mercurial has a strong commitment to backwards compatibility.

   If you are scared that upgrading will break workflows or command
   behavior, don't be. It is very rare for Mercurial to intentionally
   break backwards compatibility.

Recommended Versions
====================

Mozilla recommends running the latest stable release of Mercurial. The
latest stable release is always listed at
`https://www.mercurial-scm.org/ <https://www.mercurial-scm.org/>`_.
**As of August 2016, the latest stable release is 3.9.**

.. danger::

   Mercurial versions before 3.7.3 have known vulnerabilities that can
   lead to arbitrary code execution when pulling from repositories.
   Version 3.7.3 or newer should always be used.

Mercurial makes a major *X.Y* release every three months, typically around
the first of the month. Release months are February, May, August, and
November. A *X.Y.Z* point release is performed each month after or as
needed (if a severe issue is encountered).

If you are conservative about software updates, it is OK to wait to
upgrade until the *X.Y.1* point release following a major version bump.

Installing on Windows
=====================

If you are a Firefox developer, you should install Mercurial indirectly
through `MozillaBuild <https://wiki.mozilla.org/MozillaBuild>`_.

If you are not a Firefox developer, download a Windows installer
`direct from the Mercurial project <https://www.mercurial-scm.org/downloads>`_.

Installing on OS X
==================

Mercurial is not installed on OS X by default. You will need to install
it from a package manager or install it from source.

mach bootstrap
--------------

If you have a clone of a Firefox repository, simply run ``mach bootstrap``
to install/upgrade Mercurial. Keep in mind this will install all
packages required for Firefox development. If this is not wanted,
follow a set of instructions below.

Homebrew
--------

Homebrew typically keeps their Mercurial package up to date. Install
Mercurial through Homebrew by running::

  $ brew install mercurial

You may want to run ``brew update`` first to ensure your package
database is up to date.

MacPorts
--------

MacPorts typically keeps their Mercurial package up to date. Install
through MacPorts by running::

  $ port install mercurial

From Source
-----------

See the section below about how to install Mercurial from source.

Installing on Linux, BSD, and other UNIX-style OSs
==================================================

The instructions for installing Mercurial on many popular distributions
are available on `Mercurial's web site <https://www.mercurial-scm.org/downloads>`_.
However, many distros don't keep their Mercurial package reasonably
current. You often need to perform a source install.

Installing from Source
======================

Installing Mercurial from source is simple and should not be dismissed
because it isn't coming from a package.

Download a `source archive <https://www.mercurial-scm.org/downloads>`_
from Mercurial. Alternatively, clone the Mercurial source code and check
out the version you wish to install::

  $ hg clone https://selenic.com/repo/hg
  $ cd hg
  $ hg up 3.9

Once you have the source code, run ``make`` to install Mercurial::

  $ make install

If you would like to install Mercurial to a custom prefix::

  $ make install PREFIX=/usr/local
  $ make install PREFIX=/home/gps

.. note::

   Mercurial has some Python C extensions that make performance-critical
   parts of Mercurial significantly faster. You may need to install a
   system package such as ``python-dev`` to enable you to build Python C
   extensions.

.. tip::

   Are you concerned about a manual Mercurial install polluting your
   filesystem? Don't be.

   A Mercurial source install is fully self-contained. If you install to
   a prefix, you only need a reference to the ``PREFIX/bin/hg`` executable
   to run Mercurial. You can create a symlink to ``PREFIX/bin/hg`` anywhere
   in ``PATH`` and Mercurial should *just work*.

Verifying Your Installation
===========================

To verify Mercurial is installed properly and has a basic configuration
in place, run::

  $ hg debuginstall

If it detects problems, correct them.

If you have a clone of the Firefox repository, you are highly encouraged
to run `mach mercurial-setup` to launch an interactive wizard that will
help you optimally configure Mercurial for use at Mozilla.

Reasons to Upgrade
==================

General Advice
--------------

Mercurial releases tend to be faster and have fewer bugs than previous
releases. These are compelling reasons to stay up to date.

Avoid Mercurial versions older than 3.7.3 due to issues below.

Security Issues
---------------

Versions of Mercurial before 3.7.3 are vulnerable to multiple security
issues that can lead to executing arbitrary code when cloning or
pulling from repositories. Avoid versions older than 3.7.3!

Cloning and Pulling Performance
-------------------------------

Mercurial 3.6 contains a number of enhancements to performance of
cloning and pull operations, especially on Windows. Clone times for
mozilla-central on Windows can be several minutes faster with 3.6.

Revset Performance
------------------

Mercurial 3.5 and 3.6 contained a number of performance improvements to
revision sets. If you are a user of ``hg wip`` or ``hg smartlog``, these
commands will likely be at least 4x faster on Mercurial 3.6.

Revsets are used internally by Mercurial. So these improvements result
in performance improvements for a hodgepodge of operations.

Tags Cache Performance
----------------------

Mercurial 3.4 contains improvements to the tags cache that prevent
it from frequently doing CPU-intensive computations in some workflows.

.. important::

   Users of evolve will have horrible performance due to the tags
   cache implementation in versions older than 3.4 and should upgrade
   to 3.4+.

Performance Issues with Large Repositories
------------------------------------------

Mercurial 3.0 through 3.1.1 contained a significant performance
regression that manifests when cloning or pulling tens of thousands
of changesets. These versions of Mercurial should be avoided
when interacting with large repositories, such as mozilla-central.

Mercurial 3.3 introduced a class of performance regressions most
likely encountered as part of running ``hg blame`` or ``hg graft``.
The regressions are largely fixed in 3.4.

CVE-2014-9390
-------------

Mercurial versions older than 3.2.3 should be avoided due to a security
issue (CVE-2014-9390) impacting Windows and OS X users.

Supporting Old Versions
-----------------------

Mozilla has written a handful of Mercurial extensions. Supporting
N versions of Mercurial is easier than supporting N+1 versions,
especially as Mercurial's API is rapidly evolving. It is extra work
to support old versions when new versions work just fine.

Newer Wire Protocol
-------------------

Mercurial 3.5 featured a new wire protocol that performs pushes and
pulls more efficiently.

Cloning from Pre-Generated Bundle Files
---------------------------------------

Mercurial 3.6 supports transparently cloning from pre-generated bundle
files. When you clone from hg.mozilla.org, many of the larger
repositories will be served from a CDN. This results in a faster
and more reliable clone.
