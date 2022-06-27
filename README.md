# xenomai

#### 介绍
​	Xenomai是一个Linux内核的实时开发框架。它希望通过无缝地集成到Linux环境中来给用户空间应用程序提供全面的，与接口无关的硬实时性能。

​	Xenomai项目始于2001年8月。2003年它和RTAI项目合并推出了RTAI/fusion。RTAI/fusion是Linux平台上的具有工业生产级别的实时自由软件开发平台，它基于Xenomai的抽象实时操作系统内核。2005年的时候RTAI/fusion项目又从RTAI中独立出来作为Xenomai项目。

​	Xenomai基于一个抽象的实时操作系统核心。Xenomai给用户程序提供的多个接口（可以是不同的）实时操作系统的接口。所有通用的系统调用都是由这个核心来实现的。这些用户接口又被称作“skins”，完成Xenomai实时开发架构部署的Linux，可以将RTOS的应用程序进行移植，以Linux原生应用程序的方式运行RTOS应用（VxWorks, pSOS, VRTX, uITRON, POSIX）。



#### 软件架构

​	Xenomai可以支持多种硬件架构，详细支持[列表](https://source.denx.de/Xenomai/xenomai/-/wikis/Supported_Hardware)



#### xenomai3安装教程

##### 内核编译

1. I-pipe(Interrupt pipeline)源码获取

   ​	Cobalt实时核心依赖于I-pipe补丁， 它引入了一个单独的、高优先级的执行阶段，用于在收到IRQ后立即运行中断处理程序，这不会被常规内核工作延迟。

   ​	首先获取内核源码，通过以下地址获取I-pipe补丁。

   已发布的I-pipe补丁地址: https://xenomai.org/downloads/ipipe

   I-pipe仓库：

   - **ARM**:  https://xenomai.org/gitlab/ipipe-arm
   
   - **ARM64**: https://xenomai.org/gitlab/ipipe-arm64
   - **PPC32**: https://xenomai.org/gitlab/ipipe-ppc32
   - **x86**: https://xenomai.org/gitlab/ipipe-x86
   
   i-pipe老版本源码获取：https://xenomai.org/gitlab/ipipe。
2. I-pipe补丁合入

   切换到内核源码根目录，通过`patch`工具合入I-pipe补丁，大致步骤如下：

   ```bash
   # cd $KERNEL_ROOT
   # patch -p1 < ../ipipe-core-xxxx-xxxx.patch
   ```

3. xenomai补丁生成

   确认上一步合入补丁无误，然后切换至xenomai源码目录，使用脚本生成补丁：

   ```bash
   # cd $XENOMAI_ROOT
   # ./prepare-kernel.sh --linux=$KERNEL_ROOT --arch=xxx --outpatch=$OUTPATH/cobalt-core-xxxx-xxxx.patch 
   # cd $KERNEL_ROOT
   # patch -p1 < $OUTPATH/cobalt-core-xxxx-xxxx.patch
   ```

4. 内核配置及编译

   ```bash
   # make ARCH=xxx O=$build_root/linux xxxx_defconfig
   # make ARCH=xxx O=$build_root/linux menuconfig
   ```

   进入menuconfig界面，会看影响xenomai实时性的警告信息：

   ```bash
    *** WARNING! Page migration (CONFIG_MIGRATION) may increase *** 
    *** latency. ***                                                         
    *** WARNING! At least one of APM, CPU frequency scaling, ACPI 'processor' ***
    *** or CPU idle features is enabled. Any of these options may ***     
    *** cause troubles with Xenomai. You should disable them. ***     
   ```

   进行如下配置：

   ```bash
   General setup  --->
    	Preemption Model (Preemptible Kernel (Low-Latency Desktop))  ---> 
    		(X) Low-Latency Desktop
    	(-xeno-3.2.1)Local version - append to kernel release
    		
   Processor type and features  --->
    	Processor family (Core 2/newer Xeon)  ---> 
    		(X) Core 2/newer Xeon 
    	[*] Multi-core scheduler support                                                             [ ] CPU core priorities scheduler support 
    	
   Power management and ACPI options  --->
   	CPU Frequency scaling  --->
   		[ ] CPU Frequency scaling
   	[*] ACPI (Advanced Configuration and Power Interface) Support  ---> 
   		< > Processor 
    	CPU Idle  --->
    		[ ] CPU idle PM support
   
   Memory Management options  --->
   	[ ] Contiguous Memory Allocator
   	[ ] Transparent Hugepage Support
   	[ ] Allow for memory compaction
   	[ ] Page migration 
   
   #x86
   Microsoft Hyper-V guest support  --->                                              
   	< > Microsoft Hyper-V client drivers 
   ```

   内核编译安装：

   ```bash
   # make ARCH=xxx O=$build_root/linux -j`nproc`
   # make ARCH=xxx O=$build_root/linux modules_install 
   # make ARCH=xxx O=$build_root/linux install
   ```



##### Xenomai3安装

1. 配置

   切换到Xenomai源码目录，如果没有`configure` 脚本 和 Makefiles 文件，尝试使用如下命令：

   ```bash
   # ./scripts/bootstrap
   ```

   configure配选项如下：

   ```bash
   # ./configure --help
   ...
   Fine tuning of the installation directories:
     --bindir=DIR            user executables [EPREFIX/bin]
     --sbindir=DIR           system admin executables [EPREFIX/sbin]
     --libexecdir=DIR        program executables [EPREFIX/libexec]
     --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
     --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
     --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
     --runstatedir=DIR       modifiable per-process data [LOCALSTATEDIR/run]
     --libdir=DIR            object code libraries [EPREFIX/lib]
     --includedir=DIR        C header files [PREFIX/include]
     --oldincludedir=DIR     C header files for non-gcc [/usr/include]
     --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
     --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
     --infodir=DIR           info documentation [DATAROOTDIR/info]
     --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
     --mandir=DIR            man documentation [DATAROOTDIR/man]
     --docdir=DIR            documentation root [DATAROOTDIR/doc/xenomai]
     --htmldir=DIR           html documentation [DOCDIR]
     --dvidir=DIR            dvi documentation [DOCDIR]
     --pdfdir=DIR            pdf documentation [DOCDIR]
     --psdir=DIR             ps documentation [DOCDIR]
   
   Program names:
     --program-prefix=PREFIX            prepend PREFIX to installed program names
     --program-suffix=SUFFIX            append SUFFIX to installed program names
     --program-transform-name=PROGRAM   run sed PROGRAM on installed program names
   
   System types:
     --build=BUILD     configure for building on BUILD [guessed]
     --host=HOST       cross-compile to build programs to run on HOST [BUILD]
     --target=TARGET   configure for building compilers for TARGET [HOST]
   
   Optional Features:
     --disable-option-checking  ignore unrecognized --enable/--with options
     --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
     --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
     --enable-dependency-tracking
                             do not reject slow dependency extractors
     --disable-dependency-tracking
                             speeds up one-time build
     --enable-silent-rules   less verbose build output (undo: "make V=1")
     --disable-silent-rules  verbose build output (undo: "make V=0")
     --enable-maintainer-mode
                             enable make rules and dependencies not useful (and
                             sometimes confusing) to the casual installer
     --enable-shared[=PKGS]  build shared libraries [default=yes]
     --enable-static[=PKGS]  build static libraries [default=yes]
     --enable-fast-install[=PKGS]
                             optimize for fast installation [default=yes]
     --disable-libtool-lock  avoid locking (might break parallel builds)
     --enable-debug          Enable debug mode in programs
     --disable-demo          Disable demonstration code
     --disable-testsuite     Disable testsuite
     --enable-lores-clock    Enable low resolution clock
     --enable-clock-monotonic-raw
                             Use CLOCK_MONOTONIC_RAW for timings
     --enable-assert         Enable runtime assertions
     --enable-async-cancel   Enable asynchronous cancellation
     --enable-condvar-workaround
                             Enable workaround for broken PI with condvars in
                             glibc
     --enable-lazy-setsched  Enable lazy scheduling parameter update
     --enable-pshared        Enable shared multi-processing for capable skins
     --enable-registry       Export real-time objects to a registry
     --enable-smp            Enable SMP support
     --enable-sanity         Enable sanity checks at runtime
     --enable-x86-vsyscall   Assume VSYSCALL enabled for issuing syscalls
     --enable-doc-build      Build Xenomai documentation
     --enable-verbose-latex  Disable LaTeX non-stop mode
     --enable-valgrind-client
                             Enable Valgrind client API
     --enable-fortify        Enable _FORTIFY_SOURCE
     --enable-dlopen-libs    Allow dynamic loading of Xenomai libraries
     --enable-tls            Enable thread local storage
     --enable-kernelshark-plugin
                             build KernelShark plugin
     --enable-libtraceevent-plugin
                             build libtraceevent plugin
     --enable-so-suffix      enable soname suffix (for Mercury only)
   
   Optional Packages:
     --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
     --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
     --with-core=<cobalt | mercury>
                             build for dual kernel or single image
     --with-cc=compiler      use specific C compiler
     --with-cxx=compiler     use specific C++ compiler
     --with-gnu-ld           assume the C compiler uses GNU ld [default=no]
     --with-pic[=PKGS]       try to use only PIC/non-PIC objects [default=use
                             both]
     --with-aix-soname=aix|svr4|both
                             shared library versioning (aka "SONAME") variant to
                             provide on AIX, [default=aix].
     --with-sysroot[=DIR]    Search for dependent libraries within DIR (or the
                             compiler's sysroot if not specified).
     --with-localmem=<heapmem | tlsf>
                             Select process-local memory allocator
     --with-testdir=<test-exec-dir>
                             location for test executables (defaults to $bindir)
     --with-demodir=<demo-program-dir>
                             location for demo programs (defaults to
                             $prefix/demo)
   ```

2. 编译安装	

	```bash
	# ./scripts/bootstrap
	# ./configure --with-pic --with-core=cobalt --enable-smp --disable-tls --enable-dlopen-libs --disable-clock-monotonic-raw
	# make -j`nproc`
	# make install
	# echo '
	### Xenomai
	export XENOMAI_ROOT_DIR=/usr/xenomai
	export XENOMAI_PATH=/usr/xenomai
	export PATH=$PATH:$XENOMAI_PATH/bin:$XENOMAI_PATH/sbin
	export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$XENOMAI_PATH/lib/pkgconfig
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$XENOMAI_PATH/lib
	export OROCOS_TARGET=xenomai
	' >> ~/.xenomai_rc
	# echo 'source ~/.xenomai_rc' >> ~/.bashrc
	# source ~/.bashrc
	```



#### 使用说明

##### 应用程序编译

Xenomai3应用程序是使用libcobalt库替换glibc，常用CFLAGS，LDLIBS等参数，可以使用`xeno-config`工具获取，获取命令如下：

```bash
# xeno-config --skin=native --cflags
-I/usr/xenomai/include/trank -D__XENO_COMPAT__ -I/usr/xenomai/include/cobalt -I/usr/xenomai/include -D_GNU_SOURCE -D_REENTRANT -fasynchronous-unwind-tables -D__COBALT__ -I/usr/xenomai/include/alchemy

# xeno-config --skin=rtdm --ldflags --auto-init
-Wl,--no-as-needed -Wl,@/usr/xenomai/lib/cobalt.wrappers -Wl,@/usr/xenomai/lib/modechk.wrappers  /usr/xenomai/lib/xenomai/bootstrap.o -Wl,--wrap=main -Wl,--dynamic-list=/usr/xenomai/lib/dynlist.ld -L/usr/xenomai/lib -lcobalt -lmodechk -lpthread -lrt  
```

简单的Makefile如下：

```makefile
###### CONFIGURATION ######

### List of applications to be build
APPLICATIONS = test

###### USER SPACE BUILD (no change required normally) ######
ifneq ($(APPLICATIONS),)

### Default Xenomai installation path
XENO ?= /usr/xenomai

XENOCONFIG=$(shell PATH=$(XENO):$(XENO)/bin:$(PATH) which xeno-config 2>/dev/null)

### Sanity check
ifeq ($(XENOCONFIG),)
all::
	@echo ">>> Invoke make like this: \"make XENO=/path/to/xeno-config\" <<<"
	@echo
endif


CC=$(shell $(XENOCONFIG) --cc)

CFLAGS=$(shell $(XENOCONFIG) --skin=native --cflags)

LDLIBS=$(shell $(XENOCONFIG) --skin=rtdm --ldflags --auto-init)
#LDLIBS=$(shell $(XENOCONFIG) --skin=native --ldflags)
	
all:: $(APPLICATIONS)

clean::
	$(RM) $(APPLICATIONS) *.o

endif
```



#### 参与贡献

1. Fork 本仓库

2. 新建 Feat_xxx 分支

3. 提交代码

4. 新建 Pull Request

   

> **以上观点如有纰漏，请留言指正。**
