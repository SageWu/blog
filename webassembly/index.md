# WebAssembly

## 介绍

类似汇编语言，运行在基于栈的虚拟机上的指令集，以二进制格式存在。给C/C++、Rust等语言提供一个跨平台的编译目标，服务于web。

WebAssembly：C/C++ => emscripten(LLVM, Binaryen) => wasm。使用Clang前端转为IR，LLVM优化器优化IR，Binaryen优化并压缩wasm。\
优点是体积小、速度快。\
asm.js：JavaScript的严格子集，变量为静态类型（整数和浮点数）且取消垃圾回收机制。为经过优化的高效、低级目标语言。执行引擎会有所不同，调用WebGL通过GPU执行。\
优点是可读性高，不存在兼容问题。

## LLVM

模块化编译工具链，source => scanner => parsing => intermediate representation => IR optimizer => semantic analysis => target code => compiled code。\
其中最为出名的是IR，为低级编程语言，类似汇编语言，强类型的RISC指令集，无限临时寄存器数量。\
包含很多子项目，如LLVM Core、Clang、LLDB、libc++、libc++ ABI。
