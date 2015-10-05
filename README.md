# nghttp2-minimal-cmake
Minimal cmake support to build core nghttp2 library on Windows

This should of course live in the nghttp2 repository but the current state is "quick and dirty" and I intend to make some improvements before submitting it to the nghttp2 project.  (See future plans below.)

Instructions
------------

1. Download and unpack nghttp2-1.3.4.zip (the only version I tested) from https://nghttp2.org/.
2. Copy the `CMakeLists.txt` file from this repository to the top-level nghttp2 source directory.
3. Copy the `nghttp2.rc` file from this repository to the lib subdirectory of the top-level nghttp2 source directory.
4. Open a Visual Studio command prompt with `cmake` in `PATH` and run the following commands from a scratch (build) directory:
```
cmake -DCMAKE_INSTALL_PREFIX=c:\path\to\install -G "NMake Makefiles" -DCMAKE_BUILD_TYPE=RelWithDebInfo c:\path\to\nghttp2-1.3.4
nmake
nmake install
```

History
-------

* 2015-10-05
  * Add .rc file and related build processing
  * Clean up version string manipulation
* 2015-10-04
  * Automatically configure `nghttp2ver.h`
* 2015-10-03
  * Initial version

Future plans
------------

1. Name the libraries differently for debug vs. release, similar to `lib\Makefile.msvc`.
2. Investigate how `_DEBUG` should get defined for MSVC debug builds.  (The issue is easy to see when running `exiftool` on a nghttp2 Debug DLL -- the version shows "release".)
3. Investigate compile/link warnings and see if they indicate something is wrong with the build.
4. See what other nghttp2 features could/should be supported and see how the cmake build might support it.
5. Propose it to the nghttp2 project.
 
(Or just the last step if I don't find time to work on it further.)

Pull requests and issues
------------------------
Feel free to send pull requests for problems you find and fix.
