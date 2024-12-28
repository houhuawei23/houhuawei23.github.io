---
layout: page
title: NUDT SysY Compiler
description: for CSC-2024 Compiler Design Contest
img: assets/img/repositories/NUDT-SysY-Compiler.png
importance: 1
category: work
related_publications: false
---

github repository:

- [houhuawei23/nudt-sysyc-csc2024](https://github.com/houhuawei23/nudt-sysyc-csc2024)

Contributors 开发者:

- [侯华玮](https://github.com/houhuawei23), [汤翔晟](https://github.com/TernaryExplorer), [杨俯众](https://gitee.com/westme10n), [简泽鑫](https://github.com/xinchen-jzx)

<a href="https://github.com/houhuawei23/nudt-sysy-compiler-csc2024-origin/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=houhuawei23/nudt-sysy-compiler-csc2024-origin" />
</a>

Made with [contributors-img](https://github.com/lacolaco/contributors-img)

---

- 主要开发周期：2024.02 - 2024.08
- 总代码行数 5W+：
  - 手写代码 3W+，根据自定义模板规则自动生成代码 2W+；
- 使用 [ANTLR4](https://github.com/antlr/antlr4) 生成 C++ 前端（Lexer、Parser），生成 AST
- 参照模仿 [LLVM MLIR](https://mlir.llvm.org/)、[CMMC](https://github.com/dtcxzyw/cmmc) 设计实现了两级中间表示（IR 和 MIR），通过全部功能测试样例：
  - 词法分析、语法分析、语义分析、目标代码生成等；
- 实现若干重要的编译优化技术：
  - 死代码删除、循环剥离、循环并行化等；
- 2024 年全国大学生计算机系统能力大赛-编译系统设计赛-全国总决赛二等奖

更多详细信息请查看 [Github](https://github.com/houhuawei23/nudt-sysyc-csc2024) 仓库.