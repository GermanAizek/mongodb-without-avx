# Building Mongodb without avx

Python Prerequisites
---------------

In order to build MongoDB, Python 3.7+ is required, and several Python
modules must be installed. Python 3 is included in macOS 10.15 and later.
For earlier macOS versions, Python 3 can be installed using Homebrew or
MacPorts or similar.

To install the required Python modules, run:

    $ python3 -m pip install -r etc/pip/compile-requirements.txt

Installing the requirements inside a python3 based virtualenv
dedicated to building MongoDB is recommended.

Note: In order to compile C-based Python modules, you'll also need the
Python and OpenSSL C headers. Run:

* Fedora/RHEL - `dnf install python3-devel openssl-devel`
* Ubuntu (20.04 and newer)/Debian (Bullseye and newer) - `apt install python-dev-is-python3 libssl-dev`
* Ubuntu (18.04 and older)/Debian (Buster and older) - `apt install python3.7-dev libssl-dev`

Debian/Ubuntu
--------------

To install dependencies on Debian or Ubuntu systems:

    # apt-get install build-essential

Patching
--------------

```
git clone --recurse-submodules https://github.com/GermanAizek/mongodb-without-avx.git
cd mongo
```

Now you need to choose a patch for your requirements.

 - o2_patch.diff - default optimization
 - o3_patch.diff - maximum optimization (big binary file)
 - os_patch.diff - size binary optimization

Example:

```
patch -p1 SConstruct < ../o3_patch.diff
```

Done! Now we perform default building that you need.

SCons
---------------

If you only want to build the database server `mongod`:

    $ python3 buildscripts/scons.py install-mongod

***Note***: For C++ compilers that are newer than the supported
version, the compiler may issue new warnings that cause MongoDB to
fail to build since the build system treats compiler warnings as
errors. To ignore the warnings, pass the switch
`--disable-warnings-as-errors` to scons.

    $ python3 buildscripts/scons.py install-mongod --disable-warnings-as-errors

***Note***: On memory-constrained systems, you may run into an error such as `g++: fatal error: Killed signal terminated program cc1plus`. To use less memory during building, pass the parameter `-j1` to scons. This can be incremented to `-j4`, `-j8`, and higher as appropriate to find the fastest working option on your system. `-j` it's count CPU threads (if CPU does not have multithreading, then its number cores)

    $ python3 buildscripts/scons.py install-mongod -j1

To install `mongod` directly to `/opt/mongo`

    $ python3 buildscripts/scons.py DESTDIR=/opt/mongo install-mongod

To create an installation tree of the servers in `/tmp/unpriv` that
can later be copied to `/usr/priv`

    $ python3 buildscripts/scons.py DESTDIR=/tmp/unpriv PREFIX=/usr/priv install-servers

If you want to build absolutely everything (`mongod`, `mongo`, unit
tests, etc):

    $ python3 buildscripts/scons.py install-all-meta


SCons Targets
--------------

The following targets can be named on the scons command line to build and
install a subset of components:

* `install-mongod`
* `install-mongos`
* `install-core` (includes *only* `mongod` and `mongos`)
* `install-servers` (includes all server components)
* `install-devcore` (includes `mongod`, `mongos`, and `jstestshell` (formerly `mongo` shell))
* `install-all` (includes a complete end-user distribution and tests)
* `install-all-meta` (absolutely everything that can be built and installed)

***NOTE***: The `install-core` and `install-servers` targets are *not*
guaranteed to be identical. The `install-core` target will only ever include a
minimal set of "core" server components, while `install-servers` is intended
for a functional end-user installation. If you are testing, you should use the
`install-core` or `install-devcore` targets instead.
