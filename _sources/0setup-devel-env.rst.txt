第零章：实验环境配置
=====================

.. toctree::
   :hidden:
   :maxdepth: 4

本节我们将完成环境配置并成功运行 rCore-Tutorial / uCore-Tutorial 。整个流程分为下面几个部分：

- OS 环境配置
- 开发环境配置
   - Rust 开发环境配置
   - C 开发环境配置
- Qemu 模拟器安装
- 试运行框架代码

如果你在环境配置中遇到了无法解决的问题，请在本节讨论区留言，我们会尽力提供帮助。

OS 环境配置
-------------------------------

目前，实验主要支持 Ubuntu18.04/20.04 操作系统。使用 Windows10 和 macOS 的读者，可以安装一台 Ubuntu18.04 虚拟机 (如 VMware 或 VirtualBox) 或 Docker 进行实验。

Windows10 用户可以通过系统内置的 **WSL2** 虚拟机（请不要使用 WSL1）来安装 Ubuntu 18.04 / 20.04 。读者请自行在互联网上搜索相关安装教程，或 `适用于 Linux 的 Windows 子系统安装指南 (Windows 10) <https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package>`_ 。

.. note::

   **Docker 开发环境**

   我们也提供 Docker 镜像用于熟悉 Docker 的同学使用。如果你不熟悉或者第一次听说，可以先参考 `Docker 官网 <https://www.docker.com/get-started>`_ 进行安装。

   .. tabs::

      .. group-tab:: Rust

         感谢 dinghao188 和张汉东老师帮忙配置好的 Docker 开发环境，进入 Docker 开发环境之后不需要任何软件工具链的安装和配置，可以直接将 tutorial 运行起来，目前应该仅支持将 tutorial 运行在 Qemu 模拟器上。

         使用方法如下（以 Ubuntu18.04 为例）：

         1. 通过 ``su`` 切换到管理员账户 ``root`` ；
         2. 在 ``rCore-Tutorial`` 根目录下 ``make docker`` 进入到 Docker 环境；
         3. 进入 Docker 之后，会发现当前处于根目录 ``/`` ，我们通过 ``cd mnt`` 将当前工作路径切换到 ``/mnt`` 目录；
         4. 通过 ``ls`` 可以发现 ``/mnt`` 目录下的内容和 ``rCore-Tutorial-v3`` 目录下的内容完全相同，接下来就可以在这个环境下运行 tutorial 了。例如 ``cd os && make run`` 。

      .. group-tab:: C

         - 拉取我们的Docker镜像文件：

         .. code-block:: bash

            docker pull nzpznk/oslab-c-env
            docker pull registry.cn-hangzhou.aliyuncs.com/nzpznk/oslab-c-env

         - 查看下载的镜像文件：docker image ls (nzpznk/oslab-c-env or registry.cn-hangzhou.aliyuncs.com/nzpznk/oslab-c-env)

         - 使用下载的镜像文件创建一个名字叫container_name(可自己指定)的容器，并获得容器的bash shell: docker run -it --name container_name image_name /bin/bash

         - 一些常用的指令：

            .. code-block:: bash

               # 检查运行中的容器
               docker ps 
               # 检查所有容器（包括停止了的）
               docker ps -a
               # 停止 / 启动容器
               docker stop/start container_name
               # 获取一个运行中的docker容器的bash shell:
               docker exec -it container_name /bin/bash
               # 删除一个已经停止的容器:
               docker rm container_name

         - 之后就是把我们宿主机的文件挂载在docker容器上.在使用 docker run 启动容器时，你可以将目录挂载到容器上，这样就可以从docker容器访问到本地主机的某个文件夹.

            .. code-block:: bash

               docker run -it --name container_name --mount type=bind,src=[absolute path of folder in host machine],dst=[absolute path in container] image_name /bin/bas

使用 macOS 进行实验理论上也是可行的，但本章节仅介绍 Ubuntu 下的环境配置方案。

.. note::

   经初步测试，使用 M1 芯片的 macOS 也可以运行本实验的框架，即我们的实验对平台的要求不是很高。但我们仍建议同学配置 Ubuntu 环境，以避免未知的环境问题。

开发环境配置
-------------------------------------------

.. tabs::

   .. group-tab:: Rust

      首先安装 Rust 版本管理器 rustup 和 Rust 包管理器 cargo，可以使用官方安装脚本：

      .. code-block:: bash

         curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

      如果因网络问题通过命令行下载脚本失败了，可以在浏览器地址栏中输入 `<https://sh.rustup.rs>`_ 将脚本下载到本地运行。

      或者使用字节跳动提供的镜像源：

      .. code-block:: bash

         curl --proto '=https' --tlsv1.2 -sSf https://rsproxy.cn/rustup-init.sh | sh

      使用官方安装脚本时一些可能的加速安装的方式：

      .. tabs::

         .. group-tab:: RsProxy 源

            建议直接用字节跳动提供的安装脚本

            .. code-block:: bash

               export RUSTUP_DIST_SERVER="https://rsproxy.cn"
               export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
               curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

         .. group-tab:: tuna 源

            使用校园网同学可考虑此方案

            .. code-block:: bash

               export RUSTUP_DIST_SERVER=https://mirrors.tuna.edu.cn/rustup
               export RUSTUP_UPDATE_ROOT=https://mirrors.tuna.edu.cn/rustup/rustup
               curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

         .. group-tab:: ustc 源

            似乎现在体验最好的不再是 ustc 源了……

            .. code-block:: bash

               export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
               export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
               curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

         .. group-tab:: 代理

            如果你打开这一栏，可能也不需要参考下面这几行了

            .. code-block:: bash

               # 请根据自身配置灵活调整下面的链接
               export https_proxy=http://127.0.0.1:1080
               export http_proxy=http://127.0.0.1:1080
               export ftp_proxy=http://127.0.0.1:1080

      安装中全程选择默认选项即可。

      安装完成后，我们可以重新打开一个终端来让新设置的环境变量生效，可以手动将环境变量设置应用到当前终端，
      只需输入以下命令：

      .. code-block:: bash

         source $HOME/.cargo/env

      确认一下我们正确安装了 Rust 工具链：

      .. code-block:: bash

         # 这里以 nightly 为例，默认安装的应是 stable 版本
         > rustc --version
         rustc 1.57.0-nightly (97032a6df 2021-09-08)

      最好把 Rust 包管理器 cargo 镜像地址 crates.io 也替换，来加速三方库的下载。
      打开或新建 ``~/.cargo/config`` 文件，并修改内容：

      .. tabs::

         .. group-tab:: RsProxy 源

            目前字节跳动提供的镜像体验真的很不错

            .. code-block:: toml

               [source.crates-io]
               replace-with = 'rsproxy'

               [source.rsproxy]
               registry = "https://rsproxy.cn/crates.io-index"

               # Optional
               [registries.rsproxy]
               index = "https://rsproxy.cn/crates.io-index"

               [net]
               git-fetch-with-cli = true

         .. group-tab:: tuna 源

            相比之下，tuna 源在校园网外的体验就不是那么好

            .. code-block:: toml

               [source.crates-io]
               replace-with = 'tuna'

               [source.tuna]
               registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

         .. group-tab:: ustc 源

            ustc 源仍是一个好选择

            .. code-block:: toml

               [source.crates-io]
               registry = "https://github.com/rust-lang/crates.io-index"
               replace-with = 'ustc'
               [source.ustc]
               registry = "git://mirrors.ustc.edu.cn/crates.io-index"

      推荐 JetBrains Clion + Rust 插件或者 Visual Studio Code 搭配 rust-analyzer 和 RISC-V Support 插件进行代码阅读和开发。

      .. note::

         * JetBrains Clion 是付费商业软件，但对于学生和教师，只要在 JetBrains 网站注册账号，可以享受一定期限（半年左右）的免费使用的福利。
         * Visual Studio Code 是开源软件。
         * 当然，采用 VIM，Emacs 等传统的编辑器也是没有问题的。

   .. group-tab:: C

      首先，我们需要 RISC-V 配套的 gcc。你可以使用包管理工具：

      .. tabs::

         .. group-tab:: Ubuntu

            apt 真的非常好用

            .. code-block:: bash

               apt install gcc-riscv64-unknown-elf

            .. note::

               在 18.04 中，apt 安装的版本是 7.4.0，可能会有版本过低的问题；

               在 20.04 中，apt 安装的版本是 9.3.0，应该不会有问题；

               在更新的 20.10 和 21.04 中，apt 安装的版本为 10 以上，应该不会有问题。

         .. group-tab:: macOS

            用 macOS 还是建议安个 homebrew 呢

            .. code-block:: bash

               # 添加 riscv/riscv tap 以安装
               brew tap riscv/riscv
               brew install riscv-gnu-toolchain

            .. note::

               相比 apt，homebrew 安装的版本已经新到 11.1.0 了

         .. group-tab:: Windows

            你在干什么！快去换个类 Unix 系统！

      也可以选择一个常用的位置，放置 gcc 的可执行二进制文件。位置可以自己指定，以下仅作为参考 (并以 Ubuntu 为例)。

      .. code-block:: bash

         cd /usr/local
         # 下载预编译的 RISC-V 工具链
         sudo wget https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.12/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14.tar.gz
         # 解压缩
         tar xzvf riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14.tar.gz
         # 重命名文件夹
         mv riscv64-unknown-elf-gcc-10.2.0-2020.12.8-x86_64-linux-ubuntu14 riscv64-unknown-elf-gcc

      这就算“安装”完成了。不过我们还要把 gcc 的二进制文件路径添加到环境变量 PATH 中，以在任意目录直接运行。将以下指令添加到 home 下的 ``.bashrc`` 中既可以一劳永逸的解决 (如果你使用的是其他路径，请替换 ``/usr/local`` 为你设置的路径；如果你使用 zsh 或其他 shell，请修改为 ``.zshrc`` 或对应文件)：

      .. code-block:: bash

         export PATH="/usr/local/riscv64-unknown-elf-gcc/bin:$PATH"

      接下来，安装用于交叉编译到 musl-gcc，类似上方的步骤：

      .. code-block:: bash

         cd /usr/local
         sudo wget https://musl.cc/riscv64-linux-musl-cross.tgz
         tar xzvf riscv64-linux-musl-cross.tgz
         echo 'export PATH="/usr/local/riscv64-linux-musl-cross/bin:$PATH"' >> ~/.bashrc

      .. note::

         事实上，在我们的实验中，musl-gcc 可以被 riscv64-unknown-elf-gcc 替代。但目前用户态的测例程序仍在使用 musl-gcc 编译，预计会在不远的将来进行替换。

      此外，我们的项目使用 cmake 构建，也需要安装：

      .. code-block:: bash

         sudo apt install cmake

Qemu 模拟器安装
----------------------------------------

我们需要使用 Qemu 进行实验。你可以直接使用包管理工具进行安装：

.. tabs::

   .. group-tab:: Ubuntu

      apt 真的非常好用

      .. code-block:: bash

         apt install qemu

      .. attention::

         在 18.04 和 20.04 中，apt 安装的版本都过老，无法使用，请参考下面的手动编译部分；

         在 20.10 和 21.04 中，apt 安装的版本为 5+，应该可以使用。但系统我们并未做过测试。

   .. group-tab:: macOS

      用 macOS 还是建议安个 homebrew 呢

      .. code-block:: bash

         brew install qemu

      .. note::

         相比 apt，homebrew 安装的版本已经新到 6.1.0 了

         在 macOS 上 QEMU 不支持用户态模拟，即不存在 ``qemu-riscv64``。这对实验没有影响，只是第一章的某些示例可能无法执行。

   .. group-tab:: Windows

      你在干什么！快去换个类 Unix 系统！

下面讲解手动编译安装 Qemu。

本实验主要在 QEMU 5 上进行开发，而很多 Linux 发行版的软件包管理器默认软件源中的版本过低 (你也在上面包管理安装方法的注解中看到了)，像 macOS 的又较新 (可能存在我们尚不知道的兼容性问题)，因此从源码手动编译安装是非常靠谱的方法。

首先我们安装依赖包，获取 QEMU 的源代码并手动编译 (以下以 Ubuntu 系统为例，如果你使用 macOS 或其他 Linux 发行版，请参考 `Linux 上使用 QEMU <https://wiki.qemu.org/Hosts/Linux>`_ 或 `macOS 上使用 QEMU <https://wiki.qemu.org/Hosts/Mac>`_)：

.. code-block:: bash

   # 有可能你使用的 Ubuntu 缺少基本的工具，如 make conf，可以先执行这一条。不过可以先尝试执行后面的，有问题再回到这一条：
   sudo apt install autoconf automake autotools-dev build-essential bison flex tmux python3 curl wget

   # 安装编译所需的依赖
   sudo apt install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev libnfs-dev libiscsi-dev

   # 下载源码
   # 应该也可以换为 5.1.0 5.2.0 版本
   wget https://download.qemu.org/qemu-5.0.0.tar.xz

   # 解压
   tar xvJf qemu-5.0.0.tar.xz

   # 编译并安装 RISC-V 支持
   cd qemu-5.0.0
   ./configure --target-list=riscv64-softmmu,riscv64-linux-user
   make -j

.. note::

   注意，上面的依赖包可能并不完全，比如在 Ubuntu 18.04 上：

   - 出现 ``ERROR: pkg-config binary 'pkg-config' not found`` 时，可以安装 ``pkg-config`` 包；
   - 出现 ``ERROR: glib-2.48 gthread-2.0 is required to compile QEMU`` 时，可以安装
     ``libglib2.0-dev`` 包；
   - 出现 ``ERROR: pixman >= 0.21.8 not present`` 时，可以安装 ``libpixman-1-dev`` 包。

   如果你想获得尽可能多的 QEMU 特性，可以安装这些包 (应该没有必要)：

   .. code-block:: bash

      # all recommended additional packages for maximum code coverage can be installed like this
      sudo apt install git-email
      sudo apt install libaio-dev libbluetooth-dev libbrlapi-dev libbz2-dev
      sudo apt install libcap-dev libcap-ng-dev libcurl4-gnutls-dev libgtk-3-dev
      sudo apt install libibverbs-dev libjpeg8-dev libncurses5-dev libnuma-dev
      sudo apt install librbd-dev librdmacm-dev
      sudo apt install libsasl2-dev libsdl1.2-dev libseccomp-dev libsnappy-dev libssh2-1-dev
      sudo apt install libvde-dev libvdeplug-dev libvte-2.90-dev libxen-dev liblzo2-dev
      sudo apt install valgrind xfslibs-dev

之后我们可以在同目录下 ``sudo make install`` 将 QEMU 安装到 ``/usr/local/bin`` 目录下，但这样会引起冲突 (如和包管理工具安装的版本冲突)。可以考虑编辑 ``~/.bashrc`` (或你使用的 shell 的配置文件)，将编译好的 QEMU 添加到环境变量 PATH 中 (请根据你下载编译 QEMU 的位置更改下面几行的路径)：

.. code-block:: bash

   export PATH="/path/to/qemu-5.0.0:$PATH"
   export PATH="/path/to/qemu-5.0.0/riscv64-softmmu:$PATH"
   export PATH="/path/to/qemu-5.0.0/riscv64-linux-user:$PATH"

.. note::

   事实上，当你编译完成后，应该可以把可执行文件 ``qemu-system-riscv64`` 和 ``qemu-riscv64`` 从文件夹中复制出来，只添加这两个文件到环境变量中。其余文件可以删除以腾出空间。

随后即可在终端中 ``source ~/.bashrc`` 更新环境变量 PATH，或者重启一个新的终端。此时便可以确定 QEMU 的版本

.. code-block:: bash

   qemu-system-riscv64 --version
   qemu-riscv64 --version

试运行框架代码
------------------------------------------------------------

.. tabs::

   .. group-tab:: Rust

      .. code-block:: bash

         git clone https://github.com/LearningOS/rCore-Tutorial-2021Autumn

      只需在 ``os`` 目录下 ``make run`` 即可。在内核加载完毕之后，可以看到目前可用的应用程序。 ``usertests`` 打包了其中的很大一部分，我们可以运行它，只需输入在终端中输入它的名字即可。

      之后，可以先按下 ``Ctrl+A`` ，再按下 ``X`` 来退出 Qemu。

      .. attention::

         请务必执行 ``make run``，这将为你安装一些上文没有提及的 Rust 包依赖。

         如果卡在了

         .. code-block::

            Updating git repository `https://github.com/rcore-os/riscv`

         请通过更换 hosts 等方式解决科学上网问题，或者将 riscv 项目下载到本地，并修改 os/Cargo.toml 中的 riscv 包依赖路径

         .. code-block::

            [dependencies]
            riscv = { path = "YOUR riscv PATH", features = ["inline-asm"] }

   .. group-tab:: C

      .. code-block:: bash

         git clone https://github.com/DeathWish5/uCore-Tutorial-v2.git --recursive
         cd uCore-Tutorial-v2

      其他的章节需要处理用户代码，我们可以先运行 ch1 分支：

      .. code-block::

         > git checkout ch1
         > make run LOG=debug

         [rustsbi] RustSBI version 0.1.1
         .______       __    __      _______.___________.  _______..______   __
         |   _  \     |  |  |  |    /       |           | /       ||   _  \ |  |
         |  |_)  |    |  |  |  |   |   (----`---|  |----`|   (----`|  |_)  ||  |
         |      /     |  |  |  |    \   \       |  |      \   \    |   _  < |  |
         |  |\  \----.|  `--'  |.----)   |      |  |  .----)   |   |  |_)  ||  |
         | _| `._____| \______/ |_______/       |__|  |_______/    |______/ |__|

         [rustsbi] Platform: QEMU (Version 0.1.0)
         [rustsbi] misa: RV64ACDFIMSU
         [rustsbi] mideleg: 0x222
         [rustsbi] medeleg: 0xb1ab
         [rustsbi-dtb] Hart count: cluster0 with 1 cores
         [rustsbi] Kernel entry: 0x80200000

         hello wrold!
         [ERROR 0]stext: 0x0000000080200000
         [WARN 0]etext: 0x0000000080201000
         [INFO 0]sroda: 0x0000000080201000
         [DEBUG 0]eroda: 0x0000000080202000
         [DEBUG 0]sdata: 0x0000000080202000
         [INFO 0]edata: 0x0000000080202000
         [WARN 0]sbss : 0x0000000080212000
         [ERROR 0]ebss : 0x0000000080212000
         [PANIC 0] os/main.c:39: ALL DONE

      忽略掉编译输出后，你应该得到如上的输出，这表示 uCore 已经成功运行。

      .. note::

         **退出 qemu 的方法**

         如果是正常推出，uCore 会自动关闭 qemu，但如果 os 跑飞了，我们不能通过 ``Ctrl + C`` 来推出。此时可以先按下 ``Ctrl+A`` ，再按下 ``X`` 来退出 Qemu。

恭喜你完成了实验环境的配置，可以开始阅读教程的正文部分了！

GDB 调试支持*
------------------------------

.. attention::

   使用 GDB debug 并不是必须的，你可以暂时跳过本小节。

   如果你是使用 C 进行本实验，在完成相关环境配置后，你已经配置好了 gdb，不需要再看本小节。



在 ``os`` 目录下 ``make debug`` 可以调试我们的内核，这需要安装终端复用工具 ``tmux`` ，还需要基于 riscv64 平台的 gdb 调试器 ``riscv64-unknown-elf-gdb`` 。该调试器包含在 riscv64 gcc 工具链中，工具链的预编译版本可以在如下链接处下载：

- `Ubuntu 平台 <https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.12/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-ubuntu14.tar.gz>`_
- `macOS 平台 <https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.12/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-apple-darwin.tar.gz>`_
- `Windows 平台 <https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.12/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-w64-mingw32.tar.gz>`_
- `CentOS 平台 <https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.12/riscv64-unknown-elf-toolchain-10.2.0-2020.12.8-x86_64-linux-centos6.zip>`_

解压后在 ``bin`` 目录下即可找到 ``riscv64-unknown-elf-gdb`` 以及另外一些常用工具 ``objcopy/objdump/readelf`` 等。

当然，你也可以使用非常方便的包管理工具！

.. tabs::

   .. group-tab:: Ubuntu

      apt 真的非常好用

      .. code-block:: bash

         apt install gcc-riscv64-unknown-elf

      .. note::

         在 18.04 中，apt 安装的版本是 7.4.0，可能会有版本过低的问题；

         在 20.04 中，apt 安装的版本是 9.3.0，应该不会有问题；

         在更新的 20.10 和 21.04 中，apt 安装的版本为 10 以上，应该不会有问题。

   .. group-tab:: macOS

      用 macOS 还是建议安个 homebrew 呢

      .. code-block:: bash

         brew install riscv-gnu-toolchain

      .. note::

         相比 apt，homebrew 安装的版本已经新到 11.1.0 了

   .. group-tab:: Windows

      你在干什么！快去换个类 Unix 系统！




