---
title           : "TEB KPCR GS in x64"
date            : 2012-02-08
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

在x86中，如果折腾TEB或者KPCR，可能需要处理fs:[0x???]这样的地址，在x86中，咱们可以通过dg命令来手动查看：

    kd> dg @fs
	                                  P Si Gr Pr Lo
	Sel    Base     Limit     Type    l ze an es ng Flags
	---- -------- -------- ---------- - -- -- -- -- --------
	0030 ffdff000 00001fff Data RW Ac 0 Bg Pg P  Nl 00000c93
	
ffdff000+0x???就是要找的地址了。
当然，也可以从GDTR开始从头完全手动找，不过要麻烦一些。

到了x64，TEB和KPCR都改由gs来索引了，上述dg或者GDTR方法就不灵了。

    1: kd> dg @gs
                                                    P Si Gr Pr Lo
    Sel        Base              Limit          Type    l ze an es ng Flags
    ---- ----------------- ----------------- ---------- - -- -- -- -- --------
    002B 00000000`00000000 00000000`ffffffff Data RW Ac 3 Bg Pg P  Nl 00000cf3

Base为0是因为flat模式。
那么x64中我们如何手动查看gs:[0x???]呢？

首先需要说明的是，在x64中，gs的base内容已经挪到MSR(Model-Specific Registers) (boxcounter: 可参考注1)，需要使用如下方法：

    1: kd> rdmsr 0xC0000101
    msr[c0000101] = fffff880`009f2000

此时gs:[0x???]指示的地址是fffff880`009f2000+0x???
其中0xC0000101就是gs的地址，每个MSR寄存器都有一个地址(boxcounter: 可参考注2)。
再来校验一下：

    1: kd> !pcr
    KPCR for Processor 1 at fffff880009f2000:
    ...[省略余下无关输出内容]

注：

1. 出自《AMD64 Architecture Programmer's Manual Volume 2 [System Programming]》的“2.2.3 Segmentation”的“The FS and GS descriptor base-address fields are expanded to 64 bits and used in effective-address
ulations. The 64 bits of base address are mapped to model-specific registers (MSRs), and can .only be loaded using the WRMSR instruction.”
2. 出自《Intel? 64 and IA-32 Architectures Software Developer's Manual [Volume 3B - System Programming Guide, Part 2]》中的“Appendix B Model-Specific Registers (MSRs)”的“Table B-2.  IA-32 Architectural MSRs (Contd.)”

参考资料：

1. AMD64 Architecture Programmer's Manual
2. Intel? 64 and IA-32 Architectures Software Developer's Manual
