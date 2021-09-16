

引言
====================

本章导读
--------------------

大多数程序员的职业生涯都从 ``Hello, world!`` 开始。

.. code-block::

   printf("Hello world!\n");
   cout << "Hello world!\n";
   print("Hello world!")
   System.out.println("Hello world!");
   echo "Hello world!"
   println!("Hello world!");
   console.log("Hello world!");

然而，要用几行代码向世界问好，并不像表面上那么简单。 
``Hello, world!`` 程序能够编译运行，靠的是以 **编译器** 为主的开发环境和以 **操作系统** 为主的执行环境。

在本章中，我们将抽丝剥茧，一步步让 ``Hello, world!`` 程序脱离其依赖的执行环境，编写一个能打印 ``Hello, world!`` 的 OS。这趟旅途将让我们对应用程序及其执行环境有更深入的理解。

.. attention::
   实验指导书存在的目的是帮助读者理解框架代码。
   
   为便于测试，完成编程实验时，请以框架代码为基础，不必跟着文档从零开始编写内核。

本章我们将从操作系统最简单但也最重要的 println 入手，实现一个裸机上的 println 和带色彩的 LOG (如 info、warn 和 error 等功能)，这对程序等开发和调试至关重要。作为操作系统实验“正文”的第一章，本章的所有代码已经帮大家写好，没有需要亲自编写代码的部分。但它作为第一章又是尤为重要的一章：使用 Rust 进行实验的同学要熟悉如何脱离标准库、如何编译到裸机平台；使用 C 进行实验的同学要熟悉整个框架是如何编译的。简而言之，本章是带着大家熟悉选择的语言及对应框架，帮助大家顺利进行之后的实验。

系统调用
--------------------

在实验开始之前，大家要熟悉一下系统调用 (syscall) 的概念。相信大家在汇编的课程中一定接触过这个名词。我们 OS 课程中的 syscall 的意义也是一样的，它是操作系统提供给软件的一系列接口，使得软件能够使用系统的功能。syscall 本质上属于一种异常/中断，它在 riscv 的汇编指令中以 ecall 的形式出现。

本章的 println 在 console 中打印字符，也需要调用到 syscall。syscall 的种类有很多，操作系统通过区分 syscall 的 id 来判断是哪一个syscall。

实践体验
--------------------

.. tabs::

    .. group-tab:: Rust

        首先我们要让脱离了标准库的程序能输出 (即支持 ``println!``)，这对程序的开发和调试至关重要。我们先在用户态下实现该功能，在 `此处 <https://github.com/zhanghx0905/rust-no-std-examples>`_ 获取相关代码。之后我们把程序移植到内核态，构建在裸机上支持输出的最小运行时环境。

        获取本章代码：

        .. code-block:: console

            $ git clone https://github.com/LearningOS/rCore-Tutorial-2021Autumn.git
            $ cd rCore-Tutorial-2021Autumn
            $ git checkout ch1

        运行本章代码，并设置日志级别为 ``TRACE``：

        .. code-block:: console

            $ cd os
            $ make run LOG=TRACE

        预期输出：

        .. figure:: color-demo-rs.png
            :align: center

        除了 ``Hello, world!`` 之外还有一些额外的信息，最后关机。

    .. group-tab:: C

        获取本章代码：

        .. code-block:: console

            $ git checkout ch1

        在 qemu 模拟器上运行本章代码，看看一个小应用程序是如何在QEMU模拟的计算机上运行的：

        .. code-block:: console

            $ make run LOG=trace

        如果顺利的话，以 qemu 平台为例，将输出：

        .. figure:: color-demo-c.png
            :align: center

        除了 ``Hello, world!`` 之外还有一些额外的信息，最后关机。

.. note::

    RustSBI 是啥？

    TODO: 添加 RustSBI 的相关章节

本章代码树
--------------------

.. tabs::

    .. group-tab:: Rust

        .. code-block::

            .
            ├── bootloader (内核依赖的运行在 M 特权级的 SBI 实现，本项目中我们使用 RustSBI)
            │  └── rustsbi-qemu.bin
            ├── Dockerfile
            ├── LICENSE
            ├── Makefile
            ├── os
            │  ├── Cargo.toml (cargo 项目配置文件)
            │  ├── Makefile
            │  └── src
            │     ├── console.rs (将打印字符的 SBI 接口进一步封装实现更加强大的格式化输出)
            │     ├── entry.asm (设置内核执行环境的的一段汇编代码)
            │     ├── lang_items.rs (需要我们提供给 Rust 编译器的一些语义项，目前包含内核 panic 时的处理逻辑)
            │     ├── linker.ld (控制内核内存布局的链接脚本以使内核运行在 qemu 虚拟机上)
            │     ├── logging.rs (为本项目实现了日志功能)
            │     ├── main.rs (内核主函数)
            │     └── sbi.rs (封装底层 SBI 实现提供的 SBI 接口)
            ├── README.md
            └── rust-toolchain (整个项目的工具链版本)

            ===============================================================================
            Language            Files        Lines         Code     Comments       Blanks
            ===============================================================================
            Assembly                1           12           11            0            1
            Dockerfile              1           40           31            5            4
            Makefile                2           57           40            4           13
            Markdown                1            1            0            1            0
            Rust                    5          186          155           10           21
            TOML                    1           10            7            1            2
            ===============================================================================
            Total                  11          306          244           21           41
            ===============================================================================
    
    .. group-tab:: C

        .. code-block::

            .
            ├── bootloader (内核依赖的运行在 M 特权级的 SBI 实现，本项目中我们使用 RustSBI)
            │  └── rustsbi-qemu.bin
            ├── LICENSE
            ├── Makefile
            ├── os
            │  ├── console.c
            │  ├── console.h
            │  ├── defs.h
            │  ├── entry.S
            │  ├── kernel.ld
            │  ├── log.h
            │  ├── main.c
            │  ├── printf.c
            │  ├── printf.h
            │  ├── riscv.h
            │  ├── sbi.c
            │  ├── sbi.h
            │  └── types.h
            └── README.md

            ===============================================================================
            Language            Files        Lines         Code     Comments       Blanks
            ===============================================================================
            GNU Style Assembly      1           12           11            0            1
            C                       4          174          152            3           19
            C Header                7          486          359           39           88
            Makefile                1          107           83            3           21
            Markdown                1            2            0            2            0
            ===============================================================================
            Total                  14          781          605           47          129
            ===============================================================================
