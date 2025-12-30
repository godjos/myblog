---
title: spack基础教程
copyright: true
donate: true
toc:
  enable: true
  number: true
  auto_open: true
comments: true
sticky: 0
warning:
  enable: false
  days: 180
date: 2022-12-30 11:04:15
updated: 2025-12-30 11:04:15
permalink:
keywords:
description:
summary:
categories:
tags:
cover:
top_img:
password:
---

>参考资料：
>[Spack 入门指南 – refraction-ray (re-ra.xyz)](https://re-ra.xyz/Spack-%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/)
>[Spack：超算上最好的包管理器 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/426743137)
>[Spack — Spack 0.20.0.dev0 documentation](https://spack.readthedocs.io/en/latest/)

## spack 安装

```bash
$ git clone -c feature.manyFiles=true https://github.com/spack/spack.git
$ cd spack/bin
$ ./spack install libelf
```

## spack 使用

1. 加载 spack 环境

```bash
# For bash/zsh/sh
$ . spack/share/spack/setup-env.sh

# For tcsh/csh
$ source spack/share/spack/setup-env.csh

# For fish
$ . spack/share/spack/setup-env.fish
```

spack 文件夹架构

```bash
~/spack/                  <- installation root
   bin/
      spack               <- main spack executable

   etc/
      spack/              <- Spack config files.
                             Can be overridden by files in ~/.spack.

   var/
      spack/              <- build & stage directories
          repos/             <- contains package repositories
             builtin/        <- pkg repository that comes with Spack
                repo.yaml    <- descriptor for the builtin repository
                packages/    <- directories under here contain packages
          cache/          <- saves resources downloaded during installs

   opt/
      spack/            <- packages are installed here

   lib/
      spack/
         docs/          <- source for this documentation
         env/           <- compiler wrappers for build environment

         external/      <- external libs included in Spack distro
         llnl/          <- some general-use libraries

         spack/         <- spack module; contains Python code
            cmd/        <- each file in here is a spack subcommand
            compilers/  <- compiler description files
            test/       <- unit test modules
            util/       <- common code
   share/
      spack/
         setup-env.sh   <- spack activate script
         templates/
            modules/    <- jinja templates for module file rendering
         modules/       <- module files for installed softwares
```

* 每个用户自己安装的 spack，可以通过设定 upstream 的方式，自然的继承管理员的 spack 中已安装的所有 packages，upstream 的设置请参考[文档的这里](https://spack.readthedocs.io/en/latest/chain.html)。
* lib/spack/spack 对应的 spack 框架 python 源码部分和 var/spack/repos 对应的不同软件的安装脚本部分。而命令行调用需要的 spack binary 文件则是 bin/spack。

2. 安装 spack 软件
```shell
# 发现已经安装的包
spack external finds
# 或者以python为例
spack external find python
```

```bash

# 最简单的安装方式：以安装 tar 为例

$ spack info tar # check the information of a given package, tar here
AutotoolsPackage:   tar

Description:
    GNU Tar provides the ability to create tar archives, as well as various
    other kinds of manipulation.

Homepage: https://www.gnu.org/software/tar/

Tags:
    None

Preferred version:
    1.30    https://ftpmirror.gnu.org/tar/tar-1.30.tar.gz

# stdout below is omitted

$ spack spec tar # check the compiling spec and dependents
Input spec
--------------------------------
tar

Concretized
--------------------------------
tar@1.30%gcc@7.4.0 arch=linux-ubuntu18.04-x86_64
# the stdout indicates that tar with version 1.30 compiled by gcc 7.4.0 would be installed if spack install tar is called

$ spack install tar # install tar based on the above spec
$ spack find tar # show all versions of tar installed by spack
==> 1 installed package
-- linux-ubuntu18.04-x86_64 / gcc@7.4.0 -------------------------
tar@1.30
$ spack location -i tar # show the full path of installed tar
/home/user/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/tar-1.30-chre7vfevfhfn2bkv6lchiuoqaqwyav4
```

```bash

# 更复杂的安装方式，以安装 hpctoolkit 为例
$ spack info hpctoolkit
AutotoolsPackage:   hpctoolkit

Description:
    HPCToolkit is an integrated suite of tools for measurement and analysis
    of program performance on computers ranging from multicore desktop
    systems to the nation's largest supercomputers. By using statistical
    sampling of timers and hardware performance counters, HPCToolkit
    collects accurate measurements of a program's work, resource
    consumption, and inefficiency and attributes them to the full calling
    context in which they occur.

Homepage: http://hpctoolkit.org

Preferred version:
    2022.05.15    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 8ac72d9963c4ed7b7f56acb65feb02fbce353479

Safe versions:
    develop       [git] https://github.com/HPCToolkit/hpctoolkit.git on branch develop
    master        [git] https://github.com/HPCToolkit/hpctoolkit.git on branch master
    2022.05.15    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 8ac72d9963c4ed7b7f56acb65feb02fbce353479
    2022.04.15    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit a92fdad29fc180cc522a9087bba9554a829ee002
    2022.01.15    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 0238e9a052a696707e4e65b2269f342baad728ae
    2021.10.15    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit a8f289e4dc87ff98e05cfc105978c09eb2f5ea16
    2021.05.15    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 004ea0c2aea6a261e7d5d216c24f8a703fc6c408
    2021.03.01    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 68a051044c952f0f4dac459d9941875c700039e7
    2020.08.03    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit d9d13c705d81e5de38e624254cf0875cce6add9a
    2020.07.21    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 4e56c780cffc53875aca67d6472a2fb3678970eb
    2020.06.12    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit ac6ae1156e77d35596fea743ed8ae768f7222f19
    2020.03.01    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 94ede4e6fa1e05e6f080be8dc388240ea027f769
    2019.12.28    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit b4e1877ff96069fd8ed0fdf0e36283a5b4b62240

Deprecated versions:
    2019.08.14    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 6ea44ed3f93ede2d0a48937f288a2d41188a277c
    2018.12.28    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit 8dbf0d543171ffa9885344f32f23cc6f7f6e39bc
    2018.11.05    [git] https://github.com/HPCToolkit/hpctoolkit.git at commit d0c43e39020e67095b1f1d8bb89b75f22b12aee9

Variants:
    Name [Default]      When    Allowed values    Description
    ================    ====    ==============    ============================================================================

    all-static [off]    --      on, off           Needed when MPICXX builds static binaries for the compute nodes.
    cray [off]          --      on, off           Build for Cray compute nodes, including hpcprof-mpi.
    cuda [off]          --      on, off           Support CUDA on NVIDIA GPUs (2020.03.01 or later).
    debug [off]         --      on, off           Build in debug (develop) mode.
    level_zero [off]    --      on, off           Support Level Zero on Intel GPUs (2022.04.15 or later).
    mpi [off]           --      on, off           Build hpcprof-mpi, the MPI version of hpcprof.
    papi [on]           --      on, off           Use PAPI instead of perfmon for access to the hardware performance counters.
    rocm [off]          --      on, off           Support ROCM on AMD GPUs (2022.04.15 or later).
    viewer [on]         --      on, off           Include hpcviewer.

Build Dependencies:
    boost  dyninst    gotcha  hsa-rocr-dev  intel-xed  libmonitor  libunwind  memkind  oneapi-level-zero  python           roctracer-dev  zlib
    cuda   gnuconfig  hip     intel-tbb     libdwarf   libpfm4     mbedtls    mpi      papi               rocprofiler-dev  xerces-c

Link Dependencies:
    binutils  bzip2  dyninst   gotcha  hsa-rocr-dev  intel-xed  libmonitor  libunwind  mpi                papi             roctracer-dev  xz
    boost     cuda   elfutils  hip     intel-tbb     libdwarf   libpfm4     mbedtls    oneapi-level-zero  rocprofiler-dev  xerces-c       zlib

Run Dependencies:
    hpcviewer  memkind

$ spack spec hpctoolkit
Input spec
--------------------------------
hpctoolkit

Concretized
--------------------------------
hpctoolkit@2022.05.15%gcc@10.3.0~all-static~cray~cuda~debug~level_zero~mpi+papi~rocm+viewer arch=linux-centos7-skylake_avx512
    ^binutils@2.38%gcc@10.3.0~gas~gold~headers~interwork~ld+libiberty~lto+nls+plugins libs=shared,static arch=linux-centos7-skylake_avx512
        ^diffutils@3.8%gcc@10.3.0 arch=linux-centos7-skylake_avx512
            ^libiconv@1.16%gcc@10.3.0 libs=shared,static arch=linux-centos7-skylake_avx512
        ^gettext@0.21%gcc@10.3.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-centos7-skylake_avx512
            ^bzip2@1.0.8%gcc@10.3.0~debug~pic+shared arch=linux-centos7-skylake_avx512
            ^libxml2@2.9.13%gcc@10.3.0~python arch=linux-centos7-skylake_avx512
                ^pkgconf@1.8.0%gcc@10.3.0 arch=linux-centos7-skylake_avx512
                ^xz@5.2.5%gcc@10.3.0+pic libs=shared,static arch=linux-centos7-skylake_avx512
                ^zlib@1.2.12%gcc@10.3.0+optimize+pic+shared patches=0d38234 arch=linux-centos7-skylake_avx512
            ^ncurses@6.2%gcc@10.3.0~symlinks+termlib abi=none arch=linux-centos7-skylake_avx512
            ^tar@1.34%gcc@10.3.0 zip=pigz arch=linux-centos7-skylake_avx512
                ^pigz@2.7%gcc@10.3.0 arch=linux-centos7-skylake_avx512
                ^zstd@1.5.2%gcc@10.3.0+programs compression=none libs=shared,static arch=linux-centos7-skylake_avx512
    ^boost@1.79.0%gcc@10.3.0+atomic+chrono~clanglibcpp~container~context~contract~coroutine+date_time~debug~exception~fiber+filesystem+graph~graph_parallel~icu~iostreams~json~locale~log~math~mpi+multithreaded~nowide~numpy~pic~program_options~python~random+regex~serialization+shared~signals~singlethreaded~stacktrace+system~taggedlayout~test+thread+timer~type_erasure~versionedlayout~wave cxxstd=98 patches=a440f96 visibility=global arch=linux-centos7-skylake_avx512
    ^dyninst@12.1.0%gcc@10.3.0~ipo+openmp~stat_dysect~static build_type=RelWithDebInfo arch=linux-centos7-skylake_avx512
        ^cmake@3.23.1%gcc@10.3.0~doc+ncurses+ownlibs~qt build_type=Release arch=linux-centos7-skylake_avx512
            ^openssl@1.1.1o%gcc@10.3.0~docs~shared certs=system arch=linux-centos7-skylake_avx512
                ^perl@5.34.1%gcc@10.3.0+cpanm+shared+threads arch=linux-centos7-skylake_avx512
                    ^berkeley-db@18.1.40%gcc@10.3.0+cxx~docs+stl patches=b231fcc arch=linux-centos7-skylake_avx512
                    ^gdbm@1.19%gcc@10.3.0 arch=linux-centos7-skylake_avx512
                        ^readline@8.1%gcc@10.3.0 arch=linux-centos7-skylake_avx512
        ^elfutils@0.186%gcc@10.3.0+bzip2~debuginfod~nls+xz arch=linux-centos7-skylake_avx512
            ^m4@1.4.19%gcc@10.3.0+sigsegv patches=9dc5fbd,bfdffa7 arch=linux-centos7-skylake_avx512
                ^libsigsegv@2.13%gcc@10.3.0 arch=linux-centos7-skylake_avx512
        ^intel-tbb@2020.3%gcc@10.3.0~ipo+shared+tm build_type=RelWithDebInfo cxxstd=default patches=62ba015,ce1fb16,d62cb66 arch=linux-centos7-skylake_avx512
        ^libiberty@2.37%gcc@10.3.0+pic arch=linux-centos7-skylake_avx512
    ^hpcviewer@2022.03%gcc@10.3.0 arch=linux-centos7-skylake_avx512
        ^openjdk@11.0.15_10%gcc@10.3.0 arch=linux-centos7-skylake_avx512
    ^intel-xed@2022.04.17%gcc@10.3.0~debug+pic arch=linux-centos7-skylake_avx512
        ^python@3.6.8%gcc@10.3.0+bz2+ctypes+dbm~debug+ensurepip+libxml2+lzma+nis~optimizations+pic+pyexpat~pythoncmd+readline+shared+sqlite3+ssl+tix+tkinter~ucs4+uuid+zlib patches=c129e34 arch=linux-centos7-skylake_avx512
    ^libdwarf@20180129%gcc@10.3.0 arch=linux-centos7-skylake_avx512
    ^libmonitor@2021.11.08%gcc@10.3.0~commrank~dlopen+hpctoolkit arch=linux-centos7-skylake_avx512
    ^libunwind@1.6.2%gcc@10.3.0~block_signals~conservative_checks~cxx_exceptions~debug~debug_frame+docs+pic+tests+weak_backtrace+xz~zlib components=none libs=shared,static arch=linux-centos7-skylake_avx512
    ^memkind@1.13.0%gcc@10.3.0 arch=linux-centos7-skylake_avx512
        ^autoconf@2.69%gcc@10.3.0 patches=35c4492,7793209,a49dd5b arch=linux-centos7-skylake_avx512
        ^automake@1.16.5%gcc@10.3.0 arch=linux-centos7-skylake_avx512
        ^libtool@2.4.7%gcc@10.3.0 arch=linux-centos7-skylake_avx512
        ^numactl@2.0.14%gcc@10.3.0 patches=4e1d78c,62fc8a8,ff37630 arch=linux-centos7-skylake_avx512
    ^papi@6.0.0.1%gcc@10.3.0~cuda+example~infiniband~lmsensors~nvml~powercap~rapl~rocm~rocm_smi~sde+shared~static_tools arch=linux-centos7-skylake_avx512
    ^xerces-c@3.2.3%gcc@10.3.0 cxxstd=default netaccessor=curl transcoder=iconv arch=linux-centos7-skylake_avx512
        ^curl@7.83.0%gcc@10.3.0~gssapi~ldap~libidn2~librtmp~libssh~libssh2~nghttp2 libs=shared,static tls=openssl arch=linux-centos7-skylake_avx512
```

为了指定复杂的版本，编译器，依赖等要求，其名称可以进行一定语法的扩展，比如 `^` 开头表示依赖，`@` 开头表示版本号，`%` 开头表示编译器，`~` 开头的 syntax，表示不包含后续依赖之意等。
如果有一些编译选项不存在，而你需要设置，可以使用更高阶的方法，`spack edit <pack-name>`

```bash
# 直接编辑 spack 安装的python脚本
$ spack edit hpctoolkit
```

安装软件只需要 `spack install <package> <variants>` 即可，其中 `<variants>` 可选，代表了相应的编译参数。
```
$ spack install hpctoolkit@2022.05.15%gcc@10.3.0
```

## 加载 spack 软件

```bash
# 有两种方式可以加载已安装软件的环境
spack load tar
# or
module load tar
```
`spack load/unload` 命令是对于 `module load/unload` 命令的包装

## 高级用法

### 默认配置文件

修改默认的spack配置文件，以配置适合自己的安装环境，Spack 的配置文件利用 yaml 格式

**config**

config.yaml 中的内容大致是一些基本的路径设置，比如缓存的路径，module file 模板的路径，临时路径等等，一般来讲不太需要调整。且该文件每个选项的含义，都在 `$SPACK_ROOT/etc/spack/default/config.yaml` 中有具体的注释说明。或者也可以参考[文档的这里](https://spack.readthedocs.io/en/latest/config_yaml.html)。

**compilers**

关于编译器组的配置，每组都包括 c,c++,f77,f90 等一组编译器的路径和相应的编译 flag 与环境变量等，用于提供给 spack install 软件编译时使用和选择。见[文档的这里](https://spack.readthedocs.io/en/latest/tutorial_configuration.html#compiler-configuration)。

```yaml
compilers:
- compiler:
    spec: gcc@4.8.5
    paths:
      cc: /usr/bin/gcc
      cxx: /usr/bin/g++
      f77: /usr/bin/gfortran
      fc: /usr/bin/gfortran
    flags: {}
    operating_system: centos7
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
- compiler:
    spec: gcc@10.3.0
    paths:
      cc: /home/mrz/software/spack-0.18.1/source/spack-0.18.1/opt/spack/linux-centos7-haswell/gcc-4.8.5/gcc-10.3.0-vojg74yg2ezgz6q5i4ier6k44bdkbtqw/bin/gcc
      cxx: /home/mrz/software/spack-0.18.1/source/spack-0.18.1/opt/spack/linux-centos7-haswell/gcc-4.8.5/gcc-10.3.0-vojg74yg2ezgz6q5i4ier6k44bdkbtqw/bin/g++
      f77: /home/mrz/software/spack-0.18.1/source/spack-0.18.1/opt/spack/linux-centos7-haswell/gcc-4.8.5/gcc-10.3.0-vojg74yg2ezgz6q5i4ier6k44bdkbtqw/bin/gfortran
      fc: /home/mrz/software/spack-0.18.1/source/spack-0.18.1/opt/spack/linux-centos7-haswell/gcc-4.8.5/gcc-10.3.0-vojg74yg2ezgz6q5i4ier6k44bdkbtqw/bin/gfortran
    flags: {}
    operating_system: centos7
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
```

**packages**

packages.yaml 可能是 spack 配置文件中最复杂，但也很有必要的一部分，将依赖的版本形成一个默认配置文件。见[文档的这里](https://spack.readthedocs.io/en/latest/tutorial_configuration.html#configuring-package-preferences)。

```yaml
# packages.yaml
packages:
  all:
    compiler: [clang, gcc, intel, pgi, xl, nag]
    providers:
      mpi: [mpich, openmpi]
  permissions:
    read: world
    write: user
  hdf5:
    variants: ~mpi
  zlib:
    paths:
      zlib@1.2.8%gcc@5.4.0 arch=linux-ubuntu16.04-x86_64: /usr
    buildable: False
```

all 中 compiler 指定了 compiler 的默认顺序，一般默认第一组编译器编译。provider 则指定了一系列接口的提供者。通过诸如 mpi，lapack 这种接口，使得 spack 软件安装依赖的表达更加灵活。`packages.yaml` 还可以就具体软件给出需要的编译参数和版本等，相当于把待安装软件进一步通过配置的方式固定下来。

### 离线安装 spack

spack 依赖 `clingo` 来管理不同的版本安装，离线安装除了安装 spack 外，还需要装好 `Bootstrapping clingo` 。此外，还需要将需要安装的软件生成 mirro 再加载。

1. 在联网机器上，生成 `clingo` 安装引导

```bash
$ spack bootstrap mirror --binary-packages /opt/bootstrap
==> Adding "clingo-bootstrap@spack+python %apple-clang target=x86_64" and dependencies to the mirror at /opt/bootstrap/local-mirror
==> Adding "gnupg@2.3: %apple-clang target=x86_64" and dependencies to the mirror at /opt/bootstrap/local-mirror
==> Adding "patchelf@0.13.1:0.13.99 %apple-clang target=x86_64" and dependencies to the mirror at /opt/bootstrap/local-mirror
==> Adding binary packages from "https://github.com/alalazo/spack-bootstrap-mirrors/releases/download/v0.1-rc.2/bootstrap-buildcache.tar.gz" to the mirror at /opt/bootstrap/local-mirror
```

2. 在联网机器上，生成 安装软件的 mirro

```bash
$ spack mirror create -D -d spack-mirror-dir <package_name>@<version>
# 例如
$ spack mirror create -D -d spack-mirror-gcc-4.8.5 gcc@4.8.5
```

参数中

* `-D`：下载所有依赖
* `-d DIRECTORY`：下载到指定目录

3. 将上述文件夹压缩打包，上传到无网环境中并解包，执行下述命令加载引导和 mirro

加载引导

```bash
$ spack bootstrap add --trust local-sources /opt/bootstrap/metadata/sources
$ spack bootstrap add --trust local-binaries /opt/bootstrap/metadata/binaries
```

加载软件镜像

```bash
$ spack mirror add --scope site MIRROR_NAME file://<path-to-mirror-dir>
# 例如
$ spack mirror add --scope site gcc-4.8.5 file://./gcc-4.8.5
```

然后使用如下命令可以查看已经添加的镜像

```bash
$ spack mirror list
```

在不需要的时候可以用如下命令移除镜像

```bash
spack mirror remove --scope site MIRROR_NAME
# 例如
spack mirror remove --scope site gcc-4.8.5
```

### 创建虚环境

这一部分，和 conda 等的虚拟环境的概念很类似。可以通过创建和激活一个虚拟环境，在环境中安装的相应 package 只在环境中有效，用来维持一个协调的开发环境。

-   `spack env create <env-name>`: 创建一个新虚拟环境
-   `spack env list`: 显示所有的虚拟环境
-   `spack env activate <env-name>`: 激活名为 env-name 的虚拟环境
-   `spack env status`: 显示当前虚拟环境
-   `spack env deactive`: 退出当前虚拟环境，回到原始 spack 环境
