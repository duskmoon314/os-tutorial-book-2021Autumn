chapter7 练习
====================

- 本节难度： **理解文件系统比较费事，编程难度适中** 

本章任务
--------------------

.. tabs::

    .. group-tab:: Rust

        - 在 os 目录下 ``make run TEST=1`` 加载所有测例， ``test_usertest`` 打包了所有你需要通过的测例，你也可以通过修改这个文件调整本地测试的内容。
        - 你的内核必须前向兼容，能通过前一章的所有测例。
        - 完成本章问答作业。
        - 完成本章编程作业。
        - 最终，完成实验报告并 push 你的 ch7 分支到远程仓库。

    .. group-tab:: C

        - ``ch7b_usertest`` ``ch7_mergetest`` 
        - merge ch6 的改动，然后再次测试 ch6_usertest 和 ch7b_usertest。
        - 完成本章问答作业。
        - 完成本章编程作业。
        - 最终，完成实验报告并 push 你的 ch7 分支到远程仓库。

编程作业
--------------------

硬链接
++++++++++++++++++++

你的电脑桌面是咋样的？是放满了图标吗？反正我的 windows 是这样的。显然很少人会真的把可执行文件放到桌面上，桌面图标其实都是一些快捷方式。或者用 unix 的术语来说：软链接。为了减少工作量，我们今天来实现软链接的兄弟： `硬链接 <https://en.wikipedia.org/wiki/Hard_link>`_ 。

硬链接要求两个不同的目录项指向同一个文件，在我们的文件系统中也就是两个不同名称目录项指向同一个磁盘块。本节要求实现三个系统调用 ``sys_linkat`` ``sys_unlinkat`` ``sys_stat`` 。注意在测例中 ``sys_open`` 的接口定义也发生了变化。
  
**linkat**：

    * syscall ID: 37
    * 功能：创建一个文件的一个硬链接， `linkat标准接口 <https://linux.die.net/man/2/linkat>`_ 。
    * Ｃ接口： ``int linkat(int olddirfd, char* oldpath, int newdirfd, char* newpath, unsigned int flags)``
    * Rust 接口： ``fn linkat(olddirfd: i32, oldpath: *const u8, newdirfd: i32, newpath: *const u8, flags: u32) -> i32``
    * 参数：
        * olddirfd，newdirfd: 仅为了兼容性考虑，本次实验中始终为 AT_FDCWD (-100)，可以忽略。
        * flags: 仅为了兼容性考虑，本次实验中始终为 0，可以忽略。
        * oldpath：原有文件路径
        * newpath: 新的链接文件路径。
    * 说明：
        * 为了方便，不考虑新文件路径已经存在的情况 (属于未定义行为)，除非链接同名文件。
        * 返回值：如果出现了错误则返回 -1，否则返回 0。
    * 可能的错误
        * 链接同名文件。

**unlinkat**:

    * syscall ID: 35
    * 功能：取消一个文件路径到文件的链接, `unlinkat标准接口 <https://linux.die.net/man/2/unlinkat>`_ 。
    * Ｃ接口： ``int unlinkat(int dirfd, char* path, unsigned int flags)``
    * Rust 接口： ``fn unlinkat(dirfd: i32, path: *const u8, flags: u32) -> i32``
    * 参数：
        * dirfd: 仅为了兼容性考虑，本次实验中始终为 AT_FDCWD (-100)，可以忽略。
        * flags: 仅为了兼容性考虑，本次实验中始终为 0，可以忽略。
        * path：文件路径。
    * 说明：
        * 为了方便，不考虑使用 unlink 彻底删除文件的情况。
    * 返回值：如果出现了错误则返回 -1，否则返回 0。
    * 可能的错误
        * 文件不存在。

**fstat**:

    * syscall ID: 80
    * 功能：获取文件状态。
    * Ｃ接口： ``int fstat(int fd, struct Stat* st)``
    * Rust 接口： ``fn fstat(fd: i32, st: *mut Stat) -> i32``
    * 参数：
        * fd: 文件描述符
        * st: 文件状态结构体

        .. tabs::

            .. group-tab:: Rust

                .. code-block:: rust

                    #[repr(C)]
                    #[derive(Debug)]
                    pub struct Stat {
                        /// 文件所在磁盘驱动器号
                        pub dev: u64,
                        /// inode 文件所在 inode 编号
                        pub ino: u64,
                        /// 文件类型
                        pub mode: StatMode,
                        /// 硬链接数量，初始为1
                        pub nlink: u32,
                        /// 无需考虑，为了兼容性设计
                        pad: [u64; 7],
                    }
                    
                    /// StatMode 定义：
                    bitflags! {
                        pub struct StatMode: u32 {
                            const NULL  = 0;
                            /// directory
                            const DIR   = 0o040000;
                            /// ordinary regular file
                            const FILE  = 0o100000;
                        }
                    }

            .. group-tab:: C

                .. code-block:: c

                    struct Stat {
                        uint64 dev,     // 文件所在磁盘驱动器号
                        uint64 ino,     // inode 文件所在 inode 编号
                        uint32 mode,    // 文件类型
                        uint32 nlink,   // 硬链接数量，初始为1
                    }

                    // 文件类型只需要考虑:
                    #define DIR 0x040000		// directory
                    #define FILE 0x100000		// ordinary regular file

正确实现后，你的 os 应该能够正确运行 file* 对应的测试用例，在 shell 中执行 usertest 来执行测试。

.. hint::

    - 需要给 inode 和 dinode 都增加 link 的计数，但强烈建议不要改变整个数据结构的大小，事实上，推荐你修改一个 pad。
    - (C) os 和 nfs 的修改需要同步，只不过 nfs 比较简单，只需要初始化 link 计数为 1 就行 (可以通过修改 ``ialloc`` 来实现)。
    - 理论上讲，unlink 有删除文件的语义，如果 link 计数为 0，需要删除 inode 和对应的数据块，但我们没有设置相关的测试，如果仅仅是想拿到测试分数，可以不实现这一点。
    - 理论上讲，想要测试文件系统的属性需要重启机器，但我们没有这样做。一方面是为了测试的方便，另一方面也留了一条后路。。。。。。。。。

.. note::

    **如何调试 easy-fs** (使用 Rust 完成实验的同学请看)

    如果你在第一章练习题中已经借助 ``log`` crate 实现了日志功能，那么你可以直接在 ``easy-fs`` 中引入 ``log`` crate，通过 ``log::info!/debug!`` 等宏即可进行调试并在内核中看到日志输出。具体来说，在 ``easy-fs`` 中的修改是：在 ``easy-fs/Cargo.toml`` 的依赖中加入一行 ``log = "0.4.0"``，然后在 ``easy-fs/src/lib.rs`` 中加入一行 ``extern crate log`` 。

    你也可以完全在用户态进行调试。仿照 ``easy-fs-fuse`` 建立一个在当前操作系统中运行的应用程序，将测试逻辑写在 ``main`` 函数中。这个时候就可以将它引用的 ``easy-fs`` 的 ``no_std`` 去掉并使用 ``println!`` 进行调试。


问答作业
--------------------

1. 目前的文件系统只有单级目录，假设想要支持多级文件目录，请描述你设想的实现方式，描述合理即可。

2. 在有了多级目录之后，我们就也可以为一个目录增加硬链接了。在这种情况下，文件树中是否可能出现环路(软硬链接都可以，鼓励多尝试)？你认为应该如何解决？请在你喜欢的系统上实现一个环路，描述你的实现方式以及系统提示、实际测试结果。

报告要求
--------------------

- [暂未支持] ``lab5.pdf`` CI 网站提交，注明姓名学号。 
- 注意目录要求，报告命名 ``lab5.md`` 或 ``lab5.pdf``，位于 ``reports`` 目录下。命名错误视作没有提交。不需要删除 ``lab1.md/pdf`` ``lab2.md/pdf`` ``lab3.md/pdf`` ``lab4.md/pdf``。
- 完成 ch7 问答作业。
- [可选，不占分] 你对本次实验设计及难度的看法。