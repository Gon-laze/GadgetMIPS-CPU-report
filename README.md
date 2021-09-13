# 介绍

**GadgetMIPS**为北方工业大学一队2021年“龙芯杯”参赛作品，同时由于比赛时间冲突，因此本项目同时也是北方工业大学2021硬件设计实验的替代作品。本工程基于实现一个真正可运行的、高效的、功能相对完善的CPU的理念而设计，并通过了“龙芯杯”相关的性能与功能测试，成功在实验箱上完成运行。

**GadgetMIPS**为2020年**HikariMIPS**的修订与扩充版。**HikariMIPS**实现的功能指标为：

* 五级流水线设计；
* 基本MIPS指令实现；
* 异常、中断机制；
* 仿SRAM——AXI总线桥接；
* cache的尝试挂载；

在此基础上，**GadgetMIPS**的设计理念增添了：

* 接入icache、dcache，并进行组相联优化；
* 访存/缓存转换的cache流水化
* 信号前提处理优化；
* TLB实现；
* 操作系统运行；

部分理念因比赛限制及性能限制，未能在**GadgetMIPS**源代码上列装。最终结果经共同协助努力，在北方工业大学二队的作品**hhhhMIPS**上得到了较为完整地实现。



**GadgetMIPS**：[https://gitee.com/tzwkearn/ncut\_cpu\_1/tree/develop/](https://gitee.com/tzwkearn/ncut_cpu_1/tree/develop/)

**hhhhMIPS**：[https://gitee.com/tzwkearn/ncut\_cpu\_2/tree/tlb/](https://gitee.com/tzwkearn/ncut_cpu_2/tree/tlb/)

**GadgetMIPS的beta版本**：[https://gitee.com/tzwkearn/ncut\_cpu\_1/tree/master/](https://gitee.com/tzwkearn/ncut_cpu_1/tree/master/)

