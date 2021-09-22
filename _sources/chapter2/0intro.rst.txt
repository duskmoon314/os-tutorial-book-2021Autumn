引言
====================

本章导读
--------------------

本章展现了操作系统一系列功能：

- 通过批处理支持多个程序的自动加载和运行
- 操作系统利用硬件特权级机制，实现对操作系统自身的保护

**批处理系统** (Batch System) 出现于计算资源匮乏的年代，其核心思想是：
将多个程序打包到一起输入计算机；当一个程序运行结束后，计算机会 *自动* 执行下一个程序。

应用程序难免会出错，如果一个程序的错误导致整个操作系统都无法运行，那就太糟糕了。*保护* 操作系统不受出错程序破坏的机制被称为 **特权级** (Privilege) 机制，它实现了用户态和内核态的隔离。

本章在上一章的基础上，让我们的 OS 内核能以批处理的形式一次运行多个应用程序，同时利用特权级机制，令 OS 不因出错的用户态程序而崩溃。将待执行的程序嵌入 OS 内核之中是十分粗暴的，也不符合我们对操作系统的认知。这同时也意味着我们将开始使用独立的测例文件，并把它们打包到 os 之中。

实践体验
--------------------

.. tabs::

    .. group-tab:: Rust

        获取本章代码：

        .. code-block:: console

            $ git clone https://github.com/LearningOS/rCore-Tutorial-2021Autumn.git
            $ cd rCore-Tutorial-2021Autumn
            $ git checkout ch2

        在 qemu 模拟器上运行本章代码：

        .. code-block:: console

            $ cd os
            $ make run LOG=INFO

        批处理系统自动加载并运行了所有的用户程序，尽管某些程序出错了：

        .. code-block:: 
        
            [rustsbi] RustSBI version 0.2.0-alpha.4
            .______       __    __      _______.___________.  _______..______   __
            |   _  \     |  |  |  |    /       |           | /       ||   _  \ |  |
            |  |_)  |    |  |  |  |   |   (----`---|  |----`|   (----`|  |_)  ||  |
            |      /     |  |  |  |    \   \       |  |      \   \    |   _  < |  |
            |  |\  \----.|  `--'  |.----)   |      |  |  .----)   |   |  |_)  ||  |
            | _| `._____| \______/ |_______/       |__|  |_______/    |______/ |__|

            [rustsbi] Implementation: RustSBI-QEMU Version 0.0.1
            [rustsbi-dtb] Hart count: cluster0 with 1 cores
            [rustsbi] misa: RV64ACDFIMSU
            [rustsbi] mideleg: ssoft, stimer, sext (0x222)
            [rustsbi] medeleg: ima, ia, bkpt, la, sa, uecall, ipage, lpage, spage (0xb1ab)
            [rustsbi] pmp0: 0x80000000 ..= 0x800fffff (rwx)
            [rustsbi] pmp1: 0x80000000 ..= 0x807fffff (rwx)
            [rustsbi] pmp2: 0x0 ..= 0xffffffffffffff (---)
            [rustsbi] enter supervisor 0x80200000
            [kernel] Hello, world!
            [ INFO] [kernel] num_app = 6
            [ INFO] [kernel] app_0 [0x8020b040, 0x8020f868)
            [ INFO] [kernel] app_1 [0x8020f868, 0x80214090)
            [ INFO] [kernel] app_2 [0x80214090, 0x80218988)
            [ INFO] [kernel] app_3 [0x80218988, 0x8021d160)
            [ INFO] [kernel] app_4 [0x8021d160, 0x80221a68)
            [ INFO] [kernel] app_5 [0x80221a68, 0x80226538)
            [ INFO] [kernel] Loading app_0
            [ERROR] [kernel] PageFault in application, core dumped.
            [ INFO] [kernel] Loading app_1
            [ERROR] [kernel] IllegalInstruction in application, core dumped.
            [ INFO] [kernel] Loading app_2
            [ERROR] [kernel] IllegalInstruction in application, core dumped.
            [ INFO] [kernel] Loading app_3
            [ INFO] [kernel] Application exited with code 1234
            [ INFO] [kernel] Loading app_4
            Hello, world from user mode program!
            [ INFO] [kernel] Application exited with code 0
            [ INFO] [kernel] Loading app_5
            3^10000=5079(MOD 10007)
            3^20000=8202(MOD 10007)
            3^30000=8824(MOD 10007)
            3^40000=5750(MOD 10007)
            3^50000=3824(MOD 10007)
            3^60000=8516(MOD 10007)
            3^70000=2510(MOD 10007)
            3^80000=9379(MOD 10007)
            3^90000=2621(MOD 10007)
            3^100000=2749(MOD 10007)
            Test power OK!
            [ INFO] [kernel] Application exited with code 0
            Panicked at src/batch.rs:68 All applications completed!

    .. group-tab:: C

        本章我们引入了用户程序，我们可以通过 ``make user`` 生成用户程序，最终将 ``.bin`` 文件放在 ``user/target/bin`` 目录下。

        .. code-block:: console

            $ git checkout ch2

        .. code-block:: console

            $ make user BASE=1 CHAPTER=2
            $ make run

        也可以直接运行打包好的测试程序。make test 会完成　make user 和 make run 两个步骤（自动设置 CHAPTER），我们可以通过 BASE 控制是否生成留做练习的测例。

        .. code-block:: console

            $ make test BASE=1

        如果你发现自己的 user 目录是空的，这是由于在 clone 的时候没有增加 ``--recursive`` 参数导致 submodule 没有初始化。解决方案如下：

        .. code-block:: console

            $ git submodule init 
            $ git submodule update

        如果顺利的话，我们可以看到批处理系统自动加载并运行所有的程序并且正确在程序出错的情况下保护了自身：

        .. code-block::

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
            Hello world from user mode program!
            Test hello_world OK!
            3^10000=5079
            3^20000=8202
            3^30000=8824
            3^40000=5750
            3^50000=3824
            3^60000=8516
            3^70000=2510
            3^80000=9379
            3^90000=2621
            3^100000=2749
            Test power OK!
            string from data section
            strinstring from stack section
            strin
            Test write1 OK!
            ALL DONE

        可以看到 4 个基础测试程序都可以正常运行。

本章代码树
--------------------

.. tabs::

    .. group-tab:: Rust

        .. code-block::

            .
            ├── ... (配置文件等)
            ├── os
            │  ├── build.rs (新增：生成 link_app.S 将应用作为一个数据段链接到内核)
            │  ├── Cargo.lock
            │  ├── Cargo.toml
            │  ├── Makefile (修改：构建内核之前先构建应用)
            │  └── src
            │     ├── batch.rs (新增：实现了一个简单的批处理系统)
            │     ├── console.rs
            │     ├── entry.asm
            │     ├── lang_items.rs
            │     ├── link_app.S (构建产物，由 os/build.rs 输出)
            │     ├── linker.ld
            │     ├── logging.rs
            │     ├── main.rs (修改：主函数中需要初始化 Trap 处理并加载和执行应用)
            │     ├── sbi.rs
            │     ├── sync (新增：包装了RefCell，暂时不用关心)
            │     │  ├── mod.rs
            │     │  └── up.rs
            │     ├── syscall (新增：系统调用子模块 syscall)
            │     │  ├── fs.rs(包含文件 I/O 相关的 syscall)
            │     │  ├── mod.rs(提供 syscall 方法根据 syscall ID 进行分发处理)
            │     │  └── process.rs(包含任务处理相关的 syscall)
            │     └── trap (新增：Trap 相关子模块 trap)
            │        ├── context.rs
            │        ├── mod.rs
            │        └── trap.S
            └── user (新增：应用测例保存在 user 目录下)
               ├── Cargo.lock
               ├── Cargo.toml
               ├── Makefile
               └── src
                  ├── bin (基于用户库 user_lib 开发的应用，每个应用放在一个源文件中)
                  │  └── ... 
                  ├── console.rs
                  ├── lang_items.rs
                  ├── lib.rs (用户库 user_lib)
                  ├── linker.ld (应用的链接脚本)
                  └── syscall.rs (包含 syscall 方法生成实际用于系统调用的汇编指令，
                                  各个具体的 syscall 都是通过 syscall 来实现的)

            ❯ tokei os
            ===============================================================================
            Language            Files        Lines         Code     Comments       Blanks
            ===============================================================================
            Assembly                1           12           11            0            1
            GNU Style Assembly      2           64           63            0            1
            Makefile                1           52           36            4           12
            TOML                    1           12            9            1            2
            -------------------------------------------------------------------------------
            Rust                   14          507          435           14           58
            |- Markdown             1           11            0            9            2
            (Total)                            518          435           23           60
            ===============================================================================
            Total                  19          647          554           19           74
            ===============================================================================

    .. group-tab:: C

        .. code-block::

            .
            ├── ... (配置文件等)
            ├── nfs
            │  └── fs
            ├── os
            │  ├── console.c
            │  ├── console.h
            │  ├── const.h
            │  ├── defs.h
            │  ├── entry.S
            │  ├── kernel.ld
            │  ├── kernelld.py
            │  ├── loader.c
            │  ├── loader.h
            │  ├── log.h
            │  ├── main.c
            │  ├── pack.py
            │  ├── printf.c
            │  ├── printf.h
            │  ├── riscv.h
            │  ├── sbi.c
            │  ├── sbi.h
            │  ├── string.c
            │  ├── string.h
            │  ├── syscall.c
            │  ├── syscall.h
            │  ├── syscall_ids.h
            │  ├── trampoline.S
            │  ├── trap.c
            │  ├── trap.h
            │  └── types.h
            ├── scripts
            │  ├── kernelld.py
            │  └── pack.py
            └── user

            ❯ tokei os
            ===============================================================================
            Language            Files        Lines         Code     Comments       Blanks
            ===============================================================================
            GNU Style Assembly      2          144          132            0           12
            C                       8          434          364           19           51
            C Header               13          906          759           39          108
            Python                  2          110          101            0            9
            ===============================================================================
            Total                  25         1594         1356           58          180
            ===============================================================================