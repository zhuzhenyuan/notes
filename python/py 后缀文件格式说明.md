> .py 文件

Python 最常用格式就是 .py (另一较常用格式为 .pyw)，由 python.exe 解释，可在控制台下运行。

> .pyw

pyw是另一种源码扩展名，跟py唯一的区别是在windows下双击pyw扩展名的源码会调用pythonw.exe执行源码，  
这种执行方式不会有命令行窗口。主要用于GUI程序发布时不需要看到控制台信息的情况。

> .pyc

在执行python代码时经常会看到同目录下自动生成同名的pyc文件。  
这是python源码编译后的字节码，一般会在代码执行时自动生成你代码中引用的py文件的pyc文件。  
这个文件可以直接执行，用文本编辑器打开也看不到源码。

> .pyo

pyo是跟pyc类似的优化编码后的文件。

> .pxd文件

* .pxd 文件是由 Cython 编程语言 "编写" 而成的 Python 扩展模块头文件。
* .pxd 文件类似于 C 语言的 .h 头文件，.pxd 文件中有 Cython 模块要包含的 Cython 声明 (或代码段)。
* .pxd 文件可共享外部 C 语言声明，也能包含 C 编译器内联函数。.pxd 文件还可为 .pyx 文件模块提供 Cython 接口，  
以便其它 Cython 模块可使用比 Python 更高效的协议与之进行通信。可用 cimport 关键字将 .pxd 文件导入 .pyx 模块文件中。


> .pyx文件

* .pyx 文件是由 Cython 编程语言 "编写" 而成的 Python 扩展模块源代码文件。
* .pyx 文件类似于 C 语言的 .c 源代码文件，.pyx 文件中有 Cython 模块的源代码。

不像 Python 语言可直接解释使用的 .py 文件，.pyx 文件必须先被编译成 .c 文件，  
再编译成 .pyd (Windows 平台) 或 .so (Linux 平台) 文件，才可作为模块 import 导入使用。


> .pyd文件

* .pyd 文件是非 Python，由其它编程语言 "编写-编译" 生成的 Python 扩展模块。

Python 要导入 .pyd 文件，实际上是在 .pyd 文件中封装了一个 module。在 python 中使用时，  
把它当成 module 来用就可以了，即："import 路径名.modulename" 即可，路径名为 .pyd 文件所在的路径。

