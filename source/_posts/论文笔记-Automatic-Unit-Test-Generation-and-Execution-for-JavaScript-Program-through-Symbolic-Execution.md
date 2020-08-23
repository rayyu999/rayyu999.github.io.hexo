---
title: >-
  论文笔记 Automatic Unit Test Generation and Execution for JavaScript Program
  through Symbolic Execution
date: 2020-04-16 15:29:36
categories: Papers
tags: [JavaScript, 测试]
---

[Hideo Tanida](https://www.semanticscholar.org/author/Hideo-Tanida/2725857), [Guodong Li](https://www.semanticscholar.org/author/Guodong-Li/1720073), [Indradeep Ghosh](https://www.semanticscholar.org/author/Indradeep-Ghosh/3291522), [Tadahiro Uehara](https://www.semanticscholar.org/author/Tadahiro-Uehara/3081128)

Published in ICSEA 2014

https://scholar.google.com/scholar?hl=zh-CN&as_sdt=0%2C5&q=Automatic+Unit+Test+Generation+and+Execution+for+JavaScript+Program+through+Symbolic+Execution&btnG=

<!-- more -->


## 摘要及介绍

### 摘要

​		考虑到对更具交互性的Web/移动应用程序的需求，JavaScript有望成为一种使用更广泛的编程语言。虽然JavaScript代码的可靠性将更加重要，但与其他语言相比，该语言的测试技术仍然不足。本文提出了一种为JavaScript代码自动生成高覆盖率单元测试的技术。该技术使用JavaScript代码的符号执行引擎，以及自动为不感兴趣的代码生成存根的存根/驱动程序生成引擎。本文的方法允许全自动生成用于高覆盖率的JavaScript代码单元测试的输入数据，从而以较少的工作量确保目标代码的质量。



### 主要贡献

* 本文提出了一种在工具**SymJS**上通过符号执行为JavaScript代码生成测试输入的技术；
* 对比现有工具，本文的约束求解器**PASS**[10]允许为具有各种复杂字符串操作的程序生成测试；
* 并且不需要对目标代码进行任何修改，而现有的符号执行器需要修改和多次运行，因此适用于现实中的开发；
* 本文的自动生成存根/驱动程序代码允许完全自动生成测试数据。



### 现有的工具

​		现有的JavaScript代码测试工具包括**Kudzu**[8]和**Jalangi**[9]。kudzu自动生成程序函数的输入数据，目的是自动发现目标中的安全漏洞。Jalangi允许在正常的具体执行下修改路径约束，以便获得与以前运行不同的结果。然而，由于字符串约束处理方面的限制，以及需要手动创建用于测试的驱动程序/存根，这些工具不能应用于现实中JavaScript代码的单元测试。



## 背景及基础知识

### 符号执行

​		符号执行（Symbolic Execution）是一种程序分析技术，可以通过分析程序源代码来得到测试输入。通俗来讲，比如说游戏中一个角色在佩戴各种装备之后的面板数值为：攻击力xxx、防御力xxx...；此时通过佩戴装备到获得属性的过程就好比一个程序，佩戴装备的品质就是这个程序的输入，而最后获得的属性就是这个程序的输出；符号执行的过程则是进行分析，从某组给定的属性逆向推导，最后得到某组质量的装备，佩戴这组装备可以获得一开始给定的属性。

#### 符号执行的基本原理

​		符号执行的关键思想是把输入变为符号值，那么程序的输出值就是一个符号输入值的函数。在程序执行期间，获得每个执行路径中的变量的值要满足的约束。在获得程序内所有路径的约束之后，通过将约束提供给诸如可满足性模理论（SMT）[7]求解器之类的求解器，可以获得执行每条路径的输入变量的具体值。

参考：https://zhuanlan.zhihu.com/p/26927127

#### 通过符号执行生成测试输入

​		在程序的符号执行期间，执行每个路径所需要满足的约束是以迭代的方式计算的。在访问程序内每条可能的路径之后，获得所有路径的约束。使用**SMT**等求解器可以获得满足约束条件的变量的具体值。得到的值是与约束条件相对应的，可以用来进行测试。



### SymJS

SymJS是一个用于自动测试JavaScript Web应用程序客户端的框架，该工具包括一个用于JavaScript的符号执行引擎和一个Web页面的自动的事件资源管理器。而其中符号引擎包括一个符号虚拟机、一个字符串+数字解算器和一个符号可执行DOM模型。

​		SymJS解释从目标程序源代码编译的字节码。现有的符号执行器（如**Klee**[2]和**Symbol Pathfinder**[3]）都采用这种方法。处理字节码而不是源代码允许实现符号执行器，而无需处理语言的复杂语法。SymJS被实现为Rhino字节码的解释器，它在执行每条字节码指令时更新程序状态(堆/栈内容和路径条件)。在命中分支指令时，它复制程序状态并继续执行两个分支。

|    Tool     | Target Lang. | Sym.VM | Dep./Cache Solving | String Solving |
| :---------: | :----------: | :----: | :----------------: | :------------: |
|    SymJS    |  JavaScript  |  Yes   |        Yes         |      Yes       |
|  KLEE [2]   |      C       |  Yes   |        Yes         |       No       |
|  SAGE [6]   |  x86 binary  |   No   |        Yes         |       No       |
| Sym JPF [3] |     Java     |  Yes   |         No         |       No       |
|  Kudzu [8]  |  JavaScript  |   No   |         No         |      Yes       |
| Jalangi [9] |  JavaScript  |   No   |         No         |    Limited     |

<center>
    <i>表I 符号执行器的比较</i>
</center>


​		为了实现目标程序的符号执行，本文修改了原始Rhino对表II中所示指令的解释。堆栈操作、异常处理和变量范围管理的指令处理保持不变。



|              操作              |                             指令                             |
| :----------------------------: | :----------------------------------------------------------: |
| Arithemetic/Logical Operations | ADD, SUB, MUL, DIV, MOD, NEG, POS, BITNOT, BITAND, BITOR, BITXOR, LSH, RSH,URSH *etc.* |
|          Comparisons           |        EQ, NE, GE, GT, LE, LT, NOT, SHEQ, SHNE *etc.*        |
|            Branches            |                 IFEQ, IFNE, IFEQ_POP *etc.*                  |
|         Function Calls         |                RETURN, CALL, TAIL_CALL *etc.*                |
|      Object Manipulations      | NEW, REF, IN, INSTANCEOF, TYPEOF, GETNAME, SETNAME, NAME *etc.* |
|        Object Accesses         | GETPROP, SETPROP, DELPROP, GETELEM, SETELEM, GETREF, SETREF *etc.* |

<center>
    <i>表II 修改后的指令解释</i>
</center>
例如，指令 `ADD op1 op2` 解释如下：

(1)从堆栈中弹出操作数 `op1` 和 `op2` 。操作数可以使用抽象的值，也可以使用具体的值。

(2)检查操作数的类型。如果两个操作数都是字符串，则计算结果是操作数的拼接。如果它们是数值，则结果是操作数的总和。否则，将**根据ECMAScript语言标准转换值**，结果是将获得的值拼接或相加。

#### 分支指令

​		比较指令之后是Rhino字节码中的分支指令。SymJS处理比较和分支指令对，如下所示：

​		首先，在进行必要的类型转换后，生成与比较结果相对应的布尔公式。假设所创建的公式由符号 $c$ 表示，我们检查 $c$ 及其否定 $\neg c$ 是否与路径条件 $pc$ 一起满足。换句话说，我们检查 $pc \land c$ 和 $pc \land \neg c$ 的可满足性。如果两者都是可满足的，我们建立对应于 $pc \land c$ 和 $pc \land \neg c$ 的状态 $s_1, s_2$，并从状态 $s_1$ 和 $s_2$ 继续执行。如果其中一个是可满足的，则选择与可满足的状态相对应的状态，并从该点继续执行。

​		SymJS支持两种方式来管理在命中分支等情况下创建的状态。第一种方法是**存储程序状态变量，包括堆/栈的内容**，如[2] [3]中所做的那样。第二种方法是**只记住在分支上走哪一侧。此方法需要在回溯时从初始状态重新执行目标程序。**但是，它得益于其简单的实现和较小的内存占用。这种方法被称为“模糊”，类似于[4] [6]中介绍的技术。但是，本文的技术是在符号执行器上实现的，不需要修改现有JavaScript工具[8] [9]所需的目标代码。

​		在通过模糊化执行程序的符号过程中，状态仅由分支上的哪一方表示和存储。该信息可用于从程序的初始状态重新执行该程序，并探索目标可能采取的状态空间。在模糊化期间，用与表I中的测试1-6相对应的路径条件符号执行图1中的目标程序之后的状态如表IV所示。符号L、R表示在分支指令上走左/右分支。

```javascript
function func0 (s, a) {
    if(””.equals(s)) { // block 0
		s = null;
	} else {
		if (s.length <= 5) { // block 1
			a = a + status;
		} else {
			if(””.equals(s)) { // block 2
				Lib.m0(); // Unreachable
			} else { // block 3
				Lib.m1();
			}
		}
	}
	if (a <= Lib.m2()) { // block A
	a = 0;
	} else { // block B
	a = a + s.length; // Error with null s
	}
}
```

<center>
    <i>图1 一个代码框架：
    s, a, Lib.m2()可以取任意值</i>
</center>



![](http://images.yingwai.top/picgo/SymJSf2.png)

<center>
    <i>图2 代码执行路径</i>
</center>



| Test No. | Blocks Executed | State Representation |
| :------: | :-------------- | -------------------- |
|    1     | 0, A            | L;L                  |
|    2     | 0, B            | L;R                  |
|    3     | 1, A            | R;L;L                |
|    4     | 1, B            | R;L;R                |
|    5     | 3, A            | R;R;R;L              |
|    6     | 3, B            | R;R;R;R              |

<center>
    <i>表IV 执行图1代码后的模糊表示</i>
</center>



#### PASS

​		对于表IV中的每个状态表示，可以获得相应的路径条件。如果有可能获得满足约束条件的解，则可以将其用作测试期间的输入。关于数值的约束可以通过SMT求解器来解决，但**SMT不能处理对于字符串的约束**。因此本文使用约束求解器PASS。

​		**PASS可以处理对整数、位向量、浮点数和字符串的约束。**虽然以前的支持字符串约束的约束解算器使用位向量或自动机，但通过参数化数组传递引入的建模可以实现更高效的求解。因此，它可以解决ECMAScript标准中与字符串操作相对应的大部分约束。



#### 符号存根及驱动程序

​		符号变量是通过符号执行生成测试输入的对象。**SymJS允许通过函数调用定义符号变量。**下面的代码片段展示了定义符号字符串变量的例子：

`var s = symjs_mk_symbol_string();`

​		以上的例子定义了字符串类型的符号变量，也可以通过`symjs_mk_symbolic_int()`、`symjs_mk_symbolic_bool()`和`symjs_mk_symbolic_real()`分别定义整型、布尔型以及浮点型的符号变量。虽然SymJS只允许字符串、整数、布尔值和浮点数是符号的，但它们的约束作为更复杂的对象的组成部分保留在赋值/引用上，从而允许生成对象组成部分的值不断变化的测试。

​		为了确定图1中函数`func0()`的测试输入，需要额外的代码段。首先需要一个如下所示的符号驱动程序：

```javascript
var s = symjs_mk_symbolic_string();
var a = symjs_mk_symbolic_float();
func0(s, a);
```

<center>
    <i>图3 用于执行图1中代码的符号驱动程序</i>
</center>

驱动程序声明符号变量并将它们作为参数传递给函数。注入依赖项的存根也是必需的。如下所示的符号存根包括符号变量声明。通过存根，包含对`Lib.m2()`的函数调用的返回值，以测试SymJS获得的输入。

```javascript
Lib.m2 = function() {
    return symjs_mk_symbolic_float();
};
```

<center>
    <i>图4 提供图1中代码使用的Lib.m2()的符号存根</i>
</center>

函数`symjs_mk_symbolic_*()`是用于在测试生成期间定义新符号变量的表达式。**SymJS允许使用生成的测试输入进行正常的具体执行。**在具体执行期间，函数返回测试输入中包含的具体值。SymJS可以将测试输入导出为JavaScript Object Notation (JSON) 格式的外部文件。文件可以由测试回放库读取，该库在`symjs_mk_symbolic_*()`函数调用中返回相应的测试输入数据。加载到典型Web浏览器中的测试库可以在没有自定义JavaScript解释器的情况下执行生成的测试。



## 符号存根及驱动程序的自动生成

​		如第2部分所述，符号存根和驱动程序需要对目标函数进行符号执行并获得测试输入。返回符号变量的符号存根用于生成从被测函数调用的函数的返回值。需要符号驱动程序来改变传递给测试函数的参数。

### 生成符号存根和驱动程序的策略

​		本文的符号存根生成技术为指定的函数和类生成存根。本文的驱动程序生成技术发出调用指定函数的代码。

​		对于存根生成，生成函数根据调用者期望的返回值类型，创建和返回对象。以下是预期类型与返回对象之间的映射关系：

* 字符串、整数、布尔值和浮点数这些可以被SymJS作为符号处理的类型（后面称为SymJS基础类型）：

  新定义的相应类型的符号变量。

* 其他类：

  预期类型的新实例化对象。如果该类的目标是生成存根，则返回**新实例化的存根对象**。

* Void：不返回任何内容。

​		为了创建类的存根，还需要生成构造函数的存根。在这里，本文生成空构造函数，这将导致所有无状态对象。我们的方法假设没有对存根类的字段的直接访问，并且不为字段生成存根。

​		注意：即使返回值类型不是SymJS的基本类型，也可能得到多个测试输入。如果在返回的对象中定义的函数返回符号变量，会出现这种情况。如果**非SymJS基本类型的对象包含返回SymJS基本类型的对象的函数，并且非SymJS基本类型生成了存根**，就会发生这种情况。因此可以通过调用返回非SymJS基本类型的函数来获得多组测试输入。

​		使用本文的技术生成的符号驱动程序具有以下功能：

* 如果待测函数不是静态的、需要一个实例去执行，则实例化对应类的对象并调用该函数
* 如果待测函数是静态的，就直接调用函数

对于传递给函数的参数，驱动程序根据预期类型提供以下对象：

* SymJS基本类型：

  新定义的相应类型的符号变量。

* 其他类：

  预期类型的新实例化对象。如果该类的目标是存根生成，则传递新实例化的存根对象。

​		选择参数的方式类似于解决在符号存根中返回什么的方式。

### 从注释生成符号存根和驱动程序

​		上一部分提出的符号存根/驱动程序生成策略需要知道来自目标代码的类型信息，生成存根需要调用方期望的返回值类型，需要传递给被测函数的参数类型才能生成驱动程序。

​		但是JavaScript是一种动态类型语言，在真正运行前很难确定返回值和参数的类型。很多JavaScript程序对返回值和参数的类型有一个期望，通常是在应用程序编程接口（API）等中给出的。此外，有一种方法可以**用机器可读的方式表示JavaScript代码的类型信息**，就是JSDoc风格的注释。本文从JSDoc3[13]约定的风格的注释中获取类型信息，生成符号存根和驱动程序。

​		JSDoc3主要允许通过`@return`注释声明返回值类型。为了为图1的代码片段中使用的函数`Lib.m2()`生成符号存根，需要如下所示的注释：

```javascript
/** @return {Number} m2 value */
Lib.m2 = function() { ... };
```

如果将此类注释附加到函数的原始源代码，则可以确定返回值的类型。根据获得的返回值类型，可以以全自动的方式生成图4中的符号存根。该示例演示了为返回SymJS原语的函数生成符号存根。下面展示了为返回非SymJS基本类型的函数生成符号存根的示例：

```javascript
/** @return {tx.Data} data */
tx.Ui.getValue = function(){ ... };
```

<center>
    ↓
</center>

```javascript
tx.Ui.getValue = function(){
    return new tx.Data();
};
```

​		符号驱动程序的生成也是一样的。

​		传递给函数的参数类型通常使用JSDoc3的`@param`注释给出。函数`func0()`的符号驱动程序可以从该函数中的注释生成：

```javascript
/** @param {String} s
  * @param {Number} a */
function func0(s, a) { ... }
```

注释给出了函数的参数类型，允许生成图3中的符号驱动程序。

​		提出的自动生成符号存根和驱动程序的技术是作为JSDoc3的插件实现的。也可以使用其他来源的类型信息，例如API规范文档。



## Evaluation

​		本文使用一个工业JavaScript程序进行了实验。该程序对应于在本文的自定义Web应用程序实现框架上实现的Web应用程序的客户端部分。该程序调用未在我们的框架中包装的ECMAScript标准中定义的API，并且它只包含对标准API或本文的框架的调用。表V显示了目标程序的统计数据：

| #Line | #Function | #File |
| :---: | :-------: | :---: |
|  431  |    23     |   1   |

<center>
    <i>表V 目标程序的统计数据</i>
</center>

### 符号存根和驱动程序的生成

​		使用本文的方法成功地为框架中定义的所有类和函数生成存根。表VI列出了生成的存根的统计数据：

| #Line(Orig.) | #Line(Stub) | #Function | #File |
| :----------: | :---------: | :-------: | :---: |
|     2843     |    1304     |    154    |  13   |

<center>
    <i>表VI 用于生成存根的框架源代码和生成的存根的统计数据</i>
</center>

### 生成测试输入和执行测试

​		所有函数的符号执行在1秒内完成，并生成测试输入。测试输入的数量（将具体值赋给符号变量）在目标函数之间是不同的。对于没有分支的函数，只生成1个测试输入，而对于更复杂的函数，获得的测试数量变化到27个。

​		利用获得的测试输入来具体执行目标函数。运行在网络浏览器上的测试回放库用于重放测试。使用JSCover[14]测量测试期间的代码覆盖率，获得92%的行覆盖率。结果表明，该技术能够自动生成单元测试输入，实现高代码覆盖率。

### 实验中未覆盖的代码

​		实验结果表明，该方法可以生成代码覆盖率较高的测试输入，但没有达到100%的覆盖率，这意味着目标程序的某些部分没有被执行。以下是使用我们的方法未执行到的代码类。

​		**意外类型的代码处理对象没有被覆盖到。**由于JavaScript是一种动态类型语言，函数可能会返回意外类型的对象。为了处理这种情况，目标程序包含类型检查和后续错误处理代码。但是，通过本文的技术生成的符号存根总是返回源代码批注中描述的类型的对象。这样的存根不能利用处理不同于注释类型的对象的代码部分。

​		**没有对象类型前提的代码也没有被覆盖到。**目标程序包含在运行时确定对象类型并相应地处理它们的代码片段。然而，本文的技术不能涵盖这样的程序。从返回值类型未知的函数中，我们生成返回默认JavaScript “Object” 的存根。因此，与自定义类的对象交互的代码没有被覆盖。



## Conclusions

​		本文提出了一种为JavaScript代码自动生成单元测试输入数据的技术。该技术使用符号执行引擎，以便在测试期间实现高代码覆盖率。该技术分为两个阶段，由以下全自动步骤组成：

* 基于从注释获得的类型信息生成符号存根/驱动程序
* 通过目标代码的符号执行生成测试输入

​		实验结果显示，该技术可以生成92%的行覆盖率的测试，表明本文的技术可以自动生成和执行JavaScript代码的单元测试。

### Future Work

​		未来的方向包括更多的验证试验和各种目标程序。虽然作者已经用相对较小的程序进行了实验，但也需要在较大的目标上进行实验。

​		根据实验结果，需要改进符号存根。可以使用引发异常的符号存根触发代码处理异常。除了更复杂的自动存根生成策略外，手动修改自动生成的存根被认为是增加覆盖率的有效方法。



## 参考文献

[1] C. Cadar, V . Ganesh, P . M. Pawlowski, D. L. Dill, and D. R. Engler, *“EXE: Automatically Generating Inputs of Death,”* in Proceedings of the 13th ACM Conference on Computer and Communications Security, 2006, pp. 322–335.

[2] C. Cadar, D. Dunbar, and D. Engler, *“KLEE: Unassisted and Automatic Generation of High-coverage Tests for Complex Systems Programs,”* in Proceedings of the 8th USENIX Conference on Operating Systems Design and Implementation, 2008, pp. 209–224.

[3] C. S. Pǎsǎreanu and N. Rungta, *“Symbolic PathFinder: Symbolic Execution of Java Bytecode,”* in Proceedings of the IEEE/ACM International Conference on Automated Software Engineering, 2010, pp. 179–180.

[4] K. Sen, D. Marinov, and G. Agha, *“CUTE: A Concolic Unit Testing Engine for C,”* in Proceedings of the 10th European Software Engineering Conference, 2005, pp. 263–272.

[5] N. Tillmann and J. De Halleux, *“Pex: White Box Test Generation for .NET,”* in Proceedings of the 2nd International Conference on Tests and Proofs, ser. TAP’08, 2008, pp. 134–153.

[6] P . Godefroid, M. Y . Levin, and D. Molnar, *“SAGE: Whitebox Fuzzing for Security Testing,”* Queue, 2012, pp. 20:20–20:27.

[7] L. De Moura and N. Bjørner, *“Satisfiability Modulo Theories: Introduction and Applications,”* Commun. ACM, vol. 54, no. 9, 2011, pp. 69–77.

[8] P. Saxena, D. Akhawe, S. Hanna, F. Mao, S. McCamant, and D. Song, *“A Symbolic Execution Framework for JavaScript,”* in Proceedings of the 2010 IEEE Symposium on Security and Privacy, 2010, pp. 513–528.

[9] K. Sen, S. Kalasapur, T. Brutch, and S. Gibbs, *“Jalangi: A selective record-replay and dynamic analysis framework for javascript,”* in Proceedings of the 2013 9th Joint Meeting on Foundations of Software Engineering, 2013, pp. 488–498.

[10] G. Li and I. Ghosh, *“PASS: String Solving with Parameterized Array and Interval Automaton,”* in Proceedings of Haifa Verification Conference, 2013, pp. 15–31.

[11] ECMA International, Standard ECMA-262 - ECMAScript Language Specification, 5th ed., June 2011. [Online]. Available: http://www.ecma-international.org/publications/standards/Ecma-262.htm

[12] *“Rhino,”* https://developer.mozilla.org/en-US/docs/Rhino, [Online; accessed 2014.08.15].

[13] *“Use JSDoc,”* http://usejsdoc.org/index.html, [Online; accessed 2014.08.15].

[14] *“JSCover - JavaScript code coverage,”* http://tntim96.github.io/JSCover/ http://usejsdoc.org/index.html, [Online; accessed 2014.08.15].