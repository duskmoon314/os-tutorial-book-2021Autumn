chapter2 练习
====================

- 本节难度： **低** 

本节任务
--------------------

.. attention::

    注意 ch2 并不对应一次 lab 提交，本节任务在 ch3 最终提交。

.. tabs::

    .. group-tab:: Rust

        - 完成问答作业第一题。
        - 运行 ch2 分支的框架代码，观察三个导致错误的程序，描述他们导致的现象，完成问答作业第二题。
        - 阅读状态切换相关函数，思考并完成问答作业第三题。

    .. group-tab:: C

        - 运行 ch2 分支的框架代码，确认 ``BASE=1``　时 os 运行正常。
        - 理解目前 os 加载用户程序的整体逻辑。在 ``user`` 目录下执行 ``make CHAPTER=2_bad``　生成三个会导致错误的程序，运行并描述他们导致的现象。完成问答作业第一题。
        - 阅读状态切换相关函数，思考并完成问答作业第二题。
        - 完成问答作业第三题。

编程练习
--------------------
无

问答作业
--------------------



1. 程序陷入内核的原因有中断和异常（系统调用），请问 RISC-V 64 支持哪些中断 / 异常？如何判断进入内核是由于中断还是异常？请描述陷入内核时的几个重要寄存器及其值。为了方便 os 处理，Ｍ 态软件会将 S 态异常/中断委托给 S 态软件，请指出有哪些寄存器记录了委托信息，RustSBI 委托了哪些异常/中断？（RustSBI 在启动时输出了什么？）

2. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。请同学们可以自行测试这些内容 (参考 `C 前三个测例 <https://github.com/DeathWish5/riscvos-c-tests/tree/main/user/src>`_ `Rust 前三个测例 <https://github.com/LearningOS/rCore-Tutorial-2021Autumn/tree/ch2/user/src/bin>`_) ，描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

3. “陷入”是不同状态交互的重要方式，请回答如下几个问题：

.. tabs::

    .. group-tab:: Rust

        深入理解 `trap.S <https://github.com/LearningOS/rCore-Tutorial-2021Autumn/blob/ch2/os/src/trap/trap.S>`_ 中两个函数 ``__alltraps`` 和 ``__restore`` 的作用，并回答如下问题:

        1. L40：刚进入 ``__restore`` 时，``a0`` 代表了什么值。请指出 ``__restore`` 的两种使用情景。

        2. L46-L51：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
            
            .. code-block:: riscv

                ld t0, 32*8(sp)
                ld t1, 33*8(sp)
                ld t2, 2*8(sp)
                csrw sstatus, t0
                csrw sepc, t1
                csrw sscratch, t2

        3. L53-L59：为何跳过了 ``x2`` 和 ``x4``？ 

            .. code-block:: riscv

                ld x1, 1*8(sp)
                ld x3, 3*8(sp)
                .set n, 5
                .rept 27
                    LOAD_GP %n
                    .set n, n+1
                .endr

        4. L63：该指令之后，``sp`` 和 ``sscratch`` 中的值分别有什么意义？

            .. code-block:: riscv

                csrrw sp, sscratch, sp

        5. ``__restore``：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

        6. L13：该指令之后，``sp`` 和 ``sscratch`` 中的值分别有什么意义？

            .. code-block:: riscv

                csrrw sp, sscratch, sp

        7. 从 U 态进入 S 态是哪一条指令发生的？


    .. group-tab:: C
        
        请结合用例理解 `trampoline.S <https://github.com/LearningOS/uCore-Tutorial-v2/blob/ch2/os/trampoline.S>`_ 中两个函数 ``userret`` 和 ``uservec`` 的作用，并回答如下几个问题:

        1. L79: 刚进入 ``userret`` 时，``a0``、``a1`` 分别代表了什么值。 

        2. L87-L88: ``sfence`` 指令有何作用？为什么要执行该指令，当前章节中，删掉该指令会导致错误吗？

            .. code-block:: riscv

                csrw satp, a1
                sfence.vma zero, zero

        3. L96-L125: 为何注释中说要除去 ``a0``？哪一个地址代表 ``a0``？现在 ``a0`` 的值存在何处？

            .. code-block:: riscv

                # restore all but a0 from TRAPFRAME
                ld ra, 40(a0)
                ld sp, 48(a0)
                ld t5, 272(a0)
                ld t6, 280(a0)

        4. `userret`：中发生状态切换在哪一条指令？为何执行之后会进入用户态？

        5. L29： 执行之后，``a0`` 和 ``sscratch`` 中各是什么值，为什么？

            .. code-block:: riscv

                csrrw a0, sscratch, a0     

        6. L32-L61: 从 trapframe 第几项开始保存？为什么？是否从该项开始保存了所有的值，如果不是，为什么？
            
            .. code-block:: riscv

                sd ra, 40(a0)
                sd sp, 48(a0)
                ...
                sd t5, 272(a0)
                sd t6, 280(a0)

        7. 进入 S 态是哪一条指令发生的？

        8. L75-L76: ``ld t0, 16(a0)`` 执行之后，``t0`` 中的值是什么，解释该值的由来？

            .. code-block:: riscv

                ld t0, 16(a0)
                jr t0
