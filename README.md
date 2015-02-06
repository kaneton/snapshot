# Kaneton

The educational snapshot contains everything necessary for students to undertake the kaneton assignments.

This README contains information about how to set up the kaneton development environment and build your first kaneton microkernel image to finally run it wherever you want: emulator, real computer etc.

For more informations, please visit http://kaneton.opaak.org/


<h2 id="requirements">Requirements</h2>

Note that the kaneton development environment relies on two well-known tools:

- GNU Make
- Python (2.7).

If your system does not have one of those, you will not be able to use kaneton for development purposes.


<h2 id="profile">Profile</h2>

This step consists in specifying the kaneton development environment with your own configuration values.

As a user `foo`, you should create the following directory:

> environment/profile/user/foo/

This directory should contain, at least, a file named `foo.conf`
specifying your configuration variables. The following gives an
example of what this file should at least contain:

``` bash
_MBL_SCRIPT_          =       ${_MBL_DIR_}/grub/grub.py

_BOOT_MODE_           =       image
_BOOT_DEVICE_         =       floppy
```

This configuration tells the kaneton environment system to build kernel images for floppy devices based on the GRUB multi-bootloader. The kaneton microkernel image will be generated at the repository root directory.

This file is not mandatory but without it, you would not be able to build a complete bootable image, though you could still build the microkernel image.

It is highly recommended to overload the C flags variable in order to enable additional warning features of your compiler. For instance, comparison of enumerated types:

``` bash
_CC_FLAGS_            +=      -Wenum-compare
```

<h2 id="environment">Environment Variables</h2>

kaneton benefits from a complete and extremely sophisticated development environment.

This environment allows, amongst others, the developer to choose which platform and architecture the microkernel must be built for in a completely cooperative way.

The development environment needs to know which user profile to use, which architecture and platform to use when compiling the microkernel, which compiler to use in order to produce the right assembly language and so forth.

Therefore, every developer has to provide the following UNIX environment variables:

* KANETON_USER contains the name of the user profile;
* KANETON_HOST contains the host profile name which is composed of a couple: operating system/microprocessor;
* KANETON_PLATFORM holds the name of the target platform;
* KANETON_ARCHITECTURE holds the name of the target architecture.
* KANETON_PYTHON contains the path of the python binary. This variable is used once in order to generate the actual user's development environment;

The very common case consists in compiling kaneton on for an IBM-PC with an IA-32 microprocessor. In such cases, the following values should be used:

``` bash
export KANETON_USER="foo"
export KANETON_HOST="linux/ia32"
export KANETON_PLATFORM="ibm-pc"
export KANETON_ARCHITECTURE="ia32/educational"
export KANETON_PYTHON="/usr/bin/python"
```

One should be careful with the kaneton user name. For instance, this name should not contain any colon `:` as it would interfer with Makefiles rules. Therefore, a student or group referred to as, say `epita::2011::group42`, should not use this name when it comes to the kaneton environment user profile i.e `environment/profile/user/epita::2011::group42/` is invalid.


<h2 id="initializing">Initializing</h2>

Once the UNIX environment variables are set, you can go to the kaneton root directory and use the following command in order to actually generate your development environment:

``` bash
$> make initialize
```

In addition to setting up the development environment, this command will generate the header dependencies in order to optimize the future compilations.

Finally, function prototypes will also be generated. Indeed, kaneton relies on an automatic prototype generation mechanism which makes it easier to write code i.e without worrying about the prototypes. Students will find a specific prototype section in header files. For instance, the header file 'kaneton/core/include/set.h' contains the following section at the end:

``` c
/*
 * ---------- prototypes ---------------------------------------------
 *
 *      ../../core/set/set.c
 *      ../../core/set/set-array.c
 *      ../../core/set/set-ll.c
 *      ../../core/set/set-bpt.c
 *      ../../core/set/set-pipe.c
 *      ../../core/set/set-stack.c
 */

/*
 * ../../core/set/set.c
 */

t_error                 set_dump(void);

[... more prototypes ...]

/*
 * eop
 */
```

The section prototypes indicates the files from which the prototypes should be extracted. As a result, all the text located in the header, between the `/* --- prototypes ----- /` **and the** `/* eop */` tags, will be removed and replaced by the function prototypes. Students should therefore never put code at this location.


<h2 id="compiling">Compiling</h2>

Now, every script, Make file etc. can behave properly according to the profiles you specified.

The next step consists in compiling the kaneton microkernel implementation you have. This is done through the following command:

``` bash
$> make kaneton
```

If everything goes fine, you should see the microkernel binary file appears at the following location:

>  kaneton/kaneton

You can also use the following command for building everything necessary including the microkernel, the bootloader as well as any dependency:

``` bash
$> make
```

*Note:* If you do not want to create a boot image then obviously, you do not have to do anything else.


<h2 id="building">Building</h2>

This step consists in initializing the boot device with the multi-bootloader specified in your configuration file; probably GRUB.

The command to use this time is:

``` bash
$> make build
```

Once again, the script ask you to confirm the values the kaneton development environment has detected. Many of these are irrelevant has it depends on your boot mode.

If you specified an "image" boot mode, then the only other relevant variable is the image path. Otherwise, look closely at the network-related variables.

Note that this step only needs to be performed once. Once this is done, the kaneton microkernel can be installed as many times as you want one it. Obviously, if you remove the kaneton image or if you modify some boot-specific configuration variables, you would have to re-build the image.


<h2 id="installing">Installing</h2>

The final step consists in copying the kaneton microkernel binary as well as with its modules and configuration files onto the boot device or image, depending on your configuration.

``` bash
$> make install
```

If everything went fine, the device should be ready for usage.


<h2 id="launching">Launching</h2>

You can test your freshly built kaneton microkernel via an emulator for instance.

The command below illustrates that through QEMU:

``` bash
$> qemu -fda environment/profile/user/${KANETON_USER}/${KANETON_USER}.img
```


<h2 id="testing">Testing</h2>

To be able to test your implementation against tests written by the kaneton contributors, you first need to retrieve a test capability.

If you are a registered student, you should receive two capabilities: the first one is personal and can be used to submit k0; the other one is for the group and must be used to test and submit the other stages.

Once the capability acquired, the student willing to use it must place it in his user profile directory at:

> environment/profile/user/${KANETON_USER}/${KANETON_USER}.cap

After this, the test client can be used, as shown below:

``` bash
$> cd kaneton/test/client/
kaneton/test/client$> make information
[...]
kaneton/test/client$> make test-xen::k1
[...]
kaneton/test/client$> make submit-k1
[...]
```


<h2 id="pitfalls">Pitfalls</h2>

This section contains common pitfalls though many of those should no longer occur because kaneton has probably been fixed to prevent them.

### /bin/sh: initialize.py: not found

This error occurs when the kaneton environment is being initialized, for example by invoking `make initialize`. This error means that the system was unable to execute the `initialize.py` script.

The reason is probably that the `KANETON_PYTHON` environment variable has not been set.

Note that the other environment variables must also be set.


### /bin/sh: gmake not found

This error means that the configuration is expecting to use the `gmake` binary instead of the default `make` but it seems that this binary does not exist on the system.

If `make --version` indicates that the default make is a GNU Make, then this binary should be used.

You should therefore specify the following line in your user profile `environment/profile/user/${KANETON_USER}/${KANETON_USER}.conf`:

``` bash
  _MAKE_                =          make
```

Likewise, if `gmake` is located somewhere specific and the system cannot find it, you should specify the full path to its location:

``` bash
  _MAKE_                =          /home/boo/local/bin/make
```

At this point, since the kaneton environment has no way of running make, remember to re-initialize the environment through make initialize in order to make sure that your modifications are processed.


### undefined reference to `___stack_chk_fail'

This error occurs on recent versions of GCC when the kaneton environment performs the linking process.

In order to overcome this problem, the student should add the -fno-stack-protector option to the compilation flags, through the modification of the user configuration file, as shown below:

``` bash
  _CC_FLAGS_            +=       -fno-stack-protector
```


### no rule to make target 'kaneton.h', needed by 'early.o'

This error means that the kaneton environment is unable to locate the include files.

Activating the verbose output - by adding `_OUTPUT_ = ${_OUTPUT_VERBOSE_}` in your user configuration file - will show that the building system is not specifying any include directory. For instance, the output of the verbose could be as follows:

``` bash
$> make
[...]

[COMPILE-C]              libc.c
    gcc -fno-stack-protector -c libc.c -o libc.o
make[3]: *** No rule to make target 'kaneton.h', needed by 'early.o'. Stop.
```

Obviously, the include directory is missing in the compilation flags. One explanation may be that, in your user configuration file, you overwrote the `_CC_FLAGS_` variables. For instance, the following line in your configuration file would remove any previous value of `_CC_FLAGS_` and set `_CC_FLAGS_` to the value `-fno-stack-protector` which would in turn incur the error message above:

``` bash
  _CC_FLAGS_            =       -fno-stack-protector
```

The line below should be used instead. Note the use of `+=` instead of `=`:

``` bash
  _CC_FLAGS_            +=       -fno-stack-protector
```


### undefined reference to `__builtin_stdarg_start'

This error occurs on recent versions of GCC because the name of the builtin has changed.

To fix this problem, the student would have to edit the header file `kaneton/libc/include/stdarg.h` and replace the line:

``` c
#define va_start(v, l)         __bultin_stdarg_start((v), l)
```

with:

``` c
#define va_start(v, l)         __bultin_va_start((v), l)
```


### /bin/sh: Syntax error: "(" unexpected

This problem probably comes from the locales. Indeed, the kaneton environment sometimes run programs and expects those programs to return or display messages in a specific format. Unfortunately the language alters this format.

Setting the UNIX environment variable `LANG` to US or C should correct this problem:


``` bash
export LANG=US
```


### ImportError: No module named bulletin

Such an error may occurs when launching the test client for instance, as shown next:

``` bash
$> make information
    python  client.py  information
Traceback (most recent call last):
  File "client.py", line 30, in <module>
    import ktp
  File "/data/mycure/home/ZZZ/kaneton/test/packages/ktp/__init__.py", line 38, in <module>
    import bulletin
ImportError: No module named bulletin
    cd /home/mycure/ZZZ/kaneton/test/client
make: *** [information] Error 42
```

This probably means that you are using Python 3 or above. To fix this, please use Python 2 and/or update your configuration file by specifying the path to your Python binary:

``` bash
_PYTHON_                        =       python2
```
