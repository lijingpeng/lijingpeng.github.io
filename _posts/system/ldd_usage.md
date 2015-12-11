date: 2014-06-06
title: ldd 命令的介绍和使用方法
tags: linux, ldd, 依赖关系
category: dev
------------
ldd 能够显示可执行模块的dependency，其原理是通过设置一系列的环境变量，如下：LD_TRACE_LOADED_OBJECTS、LD_WARN、LD_BIND_NOW、LD_LIBRARY_VERSION、LD_VERBOSE等。当LD_TRACE_LOADED_OBJECTS环境变量不为空时，任何可执行程序在运行时，它都会只显示模块的dependency，而程序并不真正执行。

它的执行原理就是通过ld-linux.so（elf动态库的装载器）来实现的。我们知道，ld-linux.so模块会先于executable模块程序工作，并获得控制权，因此当上述的那些环境变量被设置时，ld-linux.so选择了显示可执行模块的dependency。
例如：
```bash
frank@linux:~/dev$ ldd a.out
	linux-vdso.so.1 =>  (0x00007fff3bffe000)
	libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f1faa400000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f1faa03a000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f1fa9d33000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f1faa72b000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f1fa9b1d000)
```

实际上可以直接执行ld-linux.so模块，如：
```bash
/lib/ld-linux.so.2 --list program
```
