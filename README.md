# 

% Linux 内核同步机制

% Wu Hengyao

% \today



\# 第一章 原子操作{\#ch1}



\#\# 应用场景



\#\#\# UP架构



在操作内存和设备寄存器时经常会遇到 R-M-W\(读-修改-写\)的操作，在串行系统中不会存在什么问题，但是在多任务或者SMP系统中可能会和我们的预期不符。



\| Thread1 \| Thread2 \|

\| :--- \| :--- \|

\| 读 \| \|

\| \| 读 \|

\| \| 修改 \|

\| \| 写 \|

\| 修改 \| \|

\| 写 \| \|



\#\#\# SMP架构



\| Thread1\(CPU1\) \| Thread2\(CPU2\) \|

\| :--- \| :--- \|

\| 读 \| \|

\| \| 读 \|

\| \| 修改 \|

\| \| 写 \|

\| 修改 \| \|

\| 写 \| \|



\#\# SWP原语



在ARMv6之前ARM架构不支持SMP，也没有引入LDREX/STREX的独占指令，但是ARM还是给我们提供了硬件的原子操作指令SWP和SWPB，它是比较特殊的内存访问指令在不必关中断的情况下原子的读取内存值到CPU寄存器，将寄存器的值写到内存\(在SWP单条指令中实现了load和store的操作\)。



SWP虽然实现了原子操作但是也有它的弊端：



-	中断延迟，如果在SWP期间触发了中断，CPU必须处理完SWP的LOAD和STORE操作才能去响应中断

-	降低了多核系统性能，例如其中一个核通过SWP访问共享内存，其它和要访问共享内存的其他数据也需要等待SWP完成释放总线



由于这几种原因在ARMv6及以后架构中ARM公司并不推荐使用SWP指令，纵观 Linux 内核代码，即使在ARMv6前也没有使用SWP实现原子操作。



\#\# LEREX/STREX 原语



ARMv6架构引入了Load-Exclusive\(LDREX\)和Store-Exclusive\(STREX\)同步原语，通过exclusive monitors跟踪独占访问，需要注意的是Load-Exclusive和Store-Exclusive只能用于Normal内存访问。



\#\#\# LDREX



LDREX从内存中加载一个字到寄存器，初始化exclusive monitor状态跟踪同步操作。



\#\#\# STREX



STREX条件的存储一个字到内存，如果exclusive monitor允许存储，则更新内存并且返回0，操作成功，否则返回1，更新内存失败。



\#\# Exclusive monitors



Exclusive monitor维护了一个简单的状态机，包括Open和Exclusive两种可能的状态，为了支持多处理器间的同步系统必须支持local和golbal两组monitor。Load-Exclusive将状态机Open状态更新为Exclusive状态。Store-Exclusive访问 monitor 判断是否可完成STORE操作，Store-Exclusive只有当monitors处于Exclusive状态时才能成功更新内存。



  !\[Local and global monitors in a multi-core system\]\(./images/monitors.png\){ \#id .class width=20% }



\#\#\# Local monitors



\#\#\# Global monitor



\#\# ARM架构Atomic实现



\#\#\# UP 架构实现



\`\`\`

	static inline void atomic\_add\(int i, atomic\_t \*v\)

	{

		unsigned long flags;



		raw\_local\_irq\_save\(flags\);

		v-&gt;counter += i;

		raw\_local\_irq\_restore\(flags\);

	}	

\`\`\`



在ARMv6之前的UP架构Linux并没有采用SWP实现原子操作，而是关中断方式。



\#\#\# SMP 架构实现



\`\`\`

	static inline void atomic\_add\(int i, atomic\_t \*v\)

	{

		unsigned long tmp;						

		int result;							

										

		prefetchw\(&v-&gt;counter\);						

		\_\_asm\_\_ \_\_volatile\_\_\("@ atomic\_" \#op "\n"		

	"1:	ldrex	%0, \[%3\]\n"						

	"	add		%0, %0, %4\n"					

	"	strex	%1, %0, \[%3\]\n"						

	"	teq	%1, \#0\n"						

	"	bne	1b"							

		: "=&r" \(result\), "=&r" \(tmp\), "+Qo" \(v-&gt;counter\)		

		: "r" \(&v-&gt;counter\), "Ir" \(i\)					

		: "cc"\);							

	}		

\`\`\`



\# 第二章



\#\# 第二章第 1 节



- 列表1

- 列表2



\#\# 第二章第 2 节



- \*\*加粗\*\*

- \*斜体\*

- 删除



\#\# 第二章第 3 节



- \[链接：泰晓科技首页\]\(http://tinylab.org\)



\#\# 第二章第 4 节



- 泰晓科技域名地址



  !\[Domain Name\]\(./images/logo-login.png\)



\#\# 第二章第 5 节



这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。这里是一段很长的内容，用于测试能够自动进行中文换行。连续重复多次。



\# 第三章



\#\# 表格用法



  \| 篇数  \| 作者  \|

  \|------:\|:------\|

  \|   24 \| test1 \|

  \|    6 \| test2 \|

  \|    5 \| test3 \|

  \|    4 \| test4 \|

  \|    4 \| test5 \|

  \|    3 \| test6 \|

  \|    2 \| test7 \|

  \|    2 \| test8 \|

  \|    2 \| test9 \|

  \|    1 \| test10\|

  \|    1 \| test11\|



\#\# 命令用法



- 结果统计与排序



	cat doc.md \| grep "^-" \| sort \| uniq -c \| sort -k1 -g -r



- 文档生成



    pandoc -f markdown doc.md -o doc.pdf

		--toc -N --latex-engine=xelatex -V mainfont="WenQuanYi Micro Hei"



\#\# 文本内命令



这里是文本内的 \`command\`。



\#\# 上标/脚注



如何明确指定上标\[^1\]。



\[^1\]: 这里是一个上标。



\#\# 链接



这里是一个\[链接\]\[2\]。



\[2\]: http://tinylab.org "泰晓科技首页"



\#\# 内部链接



点击\[这里\]\(\#ch1\)回到第一章。



